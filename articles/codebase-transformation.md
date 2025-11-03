---
title: "数千ファイルの多言語対応を2時間で完了させた話"
emoji: "🤡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [""]
published: true
published_at: 2023-12-05 08:00
---

# TL;DR

- 5,000箇所の言語追加タスクを、構造分割&コード自動変換で実装4時間＋バッチ2時間で片付けた
- AIだけに頼ると「作業量」で詰まるが、AST解析（ts-morph）で圧倒的に効率化できた
- この考え方は言語追加だけでなく、構造化リファクタ・セキュリティ対応などにも活かせる

# 背景

私たちは、Workforce Management 領域のあらゆる業務の摩擦を減らすべく「Dress Code」というプロダクトを開発しています。東南アジア圏を中心に、日本語、英語、インドネシア語、ベトナム語、タイ語でサービスを提供してきました。そんな折、CEO から「中国語簡体字も対応してほしいんだよね〜」と言われ、中国語対応に取り組むことになりました。

# 調査

どれぐらい手間がかかるのか見積もるため、まずはコードベースを一通り調査してみたところ、下記のようなコード群が約500ファイル、5,000箇所存在していました。

```typescript
export class InvalidWorker extends ErrorWithDisplayMessages {
  constructor(workerId: string) {
    super(`invalid worker. workerId is ${workerId}`, {
      ja: `不正な従業員です。 workerId: ${workerId}`,
      en: `Invalid worker. workerId: ${workerId}`,
      id: `Pekerja tidak valid. workerId: ${workerId}`,
      vn: `Người lao động không hợp lệ. workerId: ${workerId}`,
      th: `คนงานไม่ถูกต้อง workerId: ${workerId}`,
      zhCN: `无效的员工。workerId：${workerId}`,
    });
  }
}

COUNTRY_KR: {
  key: 'KR',
  labelKey: 'COUNTRY_KR',
  labelDictId: `${DictSchema.FormSelection.resource}.label.COUNTRY_KR`,
  labelJa: '韓国',
  labelEn: 'South Korea',
  labelId: 'Korea Selatan',
  labelTh: 'เกาหลีใต้',
  labelVn: 'Hàn Quốc',
  labelZhCN: '韩国',
},
```

当初は「AIに任せればすぐ終わるだろう」と考え、ファイル単位でAIに翻訳と書き換えを行わせるアプローチを試しました。しかし1ファイル処理するだけで約5分、全体では数日〜数週間かかる計算になり、実用的ではありませんでした。

AIは毎回ファイル全体の構造を解析し、翻訳対象やキーの形状の違いを逐次判断する必要があるため、同じ形式の構造でも無駄な再計算が発生し、効率が著しく低下していました。

# 構造的アプローチへの転換

そこで、処理を「抽出」「翻訳」「適用」の3工程に分割する方針に切り替えました。

- 抽出: 翻訳すべきテキストをすべて一括で取り出す
- 翻訳: 抽出結果をAIで一括翻訳
- 適用: 翻訳結果をソースコードに反映

これにより、AIが同じ構造を何度も解析する必要がなくなり、文脈理解も最小限ですむようになりました。各工程ごとに最適なツールとプロンプト設計を採用でき、結果的に作業効率も品質も大幅に改善しました。

## 実際の作業内容

ではここからは実際の作業内容をより具体的に解説します。

### 抽出

コードベースを分析すると、多言語対応には大きく2つのパターンがありました。

抽出フェーズでは、これらのパターンを認識し、

1. 多言語対応箇所に `__TRANSLATE__` という文字列を挿入
2. 挿入された `__TRANSLATE__` を基に翻訳しやすい Markdown テーブル形式に変換(抽出)

を行います。

**パターン1: DictSetObject（オブジェクト型の辞書）**

**DictSetObject**は、各言語をキー、テキスト内容を値にした、シンプルなオブジェクトリテラルです。典型的な例は以下のような形です。

```typescript
export class InvalidWorker extends ErrorWithDisplayMessages {
  constructor(workerId: string) {
    super(`invalid worker. workerId is ${workerId}`, {
      ja: `不正な従業員です。 workerId: ${workerId}`,
      en: `Invalid worker. workerId: ${workerId}`,
      id: `Pekerja tidak valid. workerId: ${workerId}`,
      vn: `Người lao động không hợp lệ. workerId: ${workerId}`,
      th: `คนงานไม่ถูกต้อง workerId: ${workerId}`,
    });
  }
}
```

**パターン2: Selection（フォーム選択肢の構造化オブジェクト）**

**Selection**は、フォームの選択肢定義などで使われる、構造化された大きめのオブジェクトです。各言語ごとに`label`プロパティが割り当てられています。

```typescript
// Before
export const Selections = {
  GENDER_MALE: {
    key: "MALE",
    labelKey: "GENDER_MALE",
    labelDictId: "form-selection.label.GENDER_MALE",
    labelJa: "男性",
    labelEn: "Male",
    labelId: "Pria",
    labelTh: "ชาย",
    labelVn: "Nam",
  },
};
```

まずは、これらのパターンを**ts-morph**を使って検出し、新言語のフィールドに `__TRANSLATE__` という文字列を挿入します。

```typescript
import { Project, SyntaxKind } from "ts-morph";

// 既存の言語コード
const EXISTING_LOCALES = ["ja", "en", "id", "th", "vn"];

/**
 * パターン1: DictSetObject の変換
 */
function transformDictSetObject(sourceFile, languageCode) {
  const objects = sourceFile.getDescendantsOfKind(
    SyntaxKind.ObjectLiteralExpression
  );

  for (const obj of objects) {
    const keys = obj
      .getProperties()
      .map((p) => p.getName())
      .filter(Boolean);

    // 既存5言語が全て揃っているか確認
    const hasAllLocales = EXISTING_LOCALES.every((loc) => keys.includes(loc));
    if (!hasAllLocales || keys.includes(languageCode)) continue;

    // 新言語フィールドを追加
    obj.addPropertyAssignment({
      name: languageCode,
      initializer: "'__TRANSLATE__'",
    });
  }
}

/**
 * パターン2: Selection の変換
 */
function transformSelection(sourceFile, languageCode) {
  const REQUIRED_STRUCTURE = ["key", "labelKey", "labelDictId"];
  const REQUIRED_LABELS = [
    "labelJa",
    "labelEn",
    "labelId",
    "labelTh",
    "labelVn",
  ];

  const objects = sourceFile.getDescendantsOfKind(
    SyntaxKind.ObjectLiteralExpression
  );

  for (const obj of objects) {
    const keys = obj
      .getProperties()
      .map((p) => p.getName())
      .filter(Boolean);

    // Selectionパターンかどうかを判定
    const isSelection =
      REQUIRED_STRUCTURE.every((f) => keys.includes(f)) &&
      REQUIRED_LABELS.every((f) => keys.includes(f));

    if (!isSelection) continue;

    // labelZhCN などを追加
    const labelField = `label${languageCode
      .charAt(0)
      .toUpperCase()}${languageCode.slice(1)}`;
    if (keys.includes(labelField)) continue;

    obj.addPropertyAssignment({
      name: labelField,
      initializer: "'__TRANSLATE__'",
    });
  }
}

// 実行
const project = new Project({ tsConfigFilePath: "tsconfig.json" });
for (const file of project.getSourceFiles("src/**/*.ts")) {
  transformDictSetObject(file, "zhCN");
  transformSelection(file, "zhCN");
  file.saveSync();
}
```

続いて `__TRANSLATE__` を見つけてその位置を基にMarkdown形式でまとめて出力します。

| File        | Line | ja                   | en                | zhCN            |
| ----------- | ---- | -------------------- | ----------------- | --------------- |
| user.dto.ts | 45   | ユーザー名           | Username          | `__TRANSLATE__` |
| error.ts    | 89   | エラーが発生しました | An error occurred | `__TRANSLATE__` |

ここでのポイントは以下の通りです。

- 翻訳のブレを防ぐために日本語と英語両方を抽出（後続での翻訳に活かす）
- 並列での翻訳を可能にする＆1ファイルあたりのコンテキスト量調整のため100エントリ単位で分解

### 翻訳

翻訳処理そのものの詳細は割愛しますが、抽出済みMarkdownをAIにバッチで投げることで、高速かつ統一的な翻訳を得られます。

背景文脈を制御できるため、同義語の揺れや誤訳も減りやすく、精度の高い翻訳結果を得ることができました。

### 適用

最後に、Markdownから翻訳を読み取り、ソースコードの `__TRANSLATE__` を置換します。これもts-morphを使って実装しています。

```typescript
import { Project } from "ts-morph";
import * as fs from "fs";

/**
 * Markdownバッチファイルをパースして翻訳データを取得
 */
function parseMarkdownBatch(filePath) {
  const content = fs.readFileSync(filePath, "utf-8");
  const lines = content.split("\n");
  const entries = [];

  // ヘッダー行をスキップしてデータ行を処理
  for (let i = 2; i < lines.length; i++) {
    if (!lines[i].trim()) continue;

    const columns = lines[i].split("|").map((c) => c.trim());
    entries.push({
      file: columns[1],
      line: parseInt(columns[2]),
      translated: columns[5], // zhCN列
    });
  }

  return entries;
}

/**
 * __TRANSLATE__ を実際の翻訳テキストに置換
 */
function applyTranslations(languageCode) {
  const project = new Project({ tsConfigFilePath: "tsconfig.json" });

  // 全バッチファイルを読み込み
  const batchFiles = fs.readdirSync(`output/${languageCode}`);

  for (const batchFile of batchFiles) {
    const entries = parseMarkdownBatch(`output/${languageCode}/${batchFile}`);

    for (const entry of entries) {
      const sourceFile = project.getSourceFile(entry.file);
      if (!sourceFile) continue;

      // 指定行のオブジェクトリテラルを取得
      const node = sourceFile
        .getDescendantsAtRange(entry.line, entry.line + 1)
        .find((n) => Node.isObjectLiteralExpression(n));

      if (!node) continue;

      // 対象言語のプロパティを取得
      const prop = node.getProperty(languageCode);
      if (!prop) continue;

      // __TRANSLATE__ を実際の翻訳に置換
      const initializer = prop.getInitializer();
      if (initializer?.getText().includes("__TRANSLATE__")) {
        initializer.replaceWithText(`'${entry.translated}'`);
      }
    }

    // ファイルを保存
    sourceFile?.saveSync();
  }
}

// 実行
applyTranslations("zhCN");
```

このように工程を、「抽出」「翻訳」「適用」に分割して進めることで、実装には4時間、実行時間は2時間という投資で相当の時間を削減できたように思います。後続での言語追加対応を考えると、投資対効果としては十分です。

## 最後に

大規模コードベースでの言語追加は、手作業の時代ではありません。AST変換を使えば、数千箇所の変更を数時間で完了できます。同時に、型安全性も保持できます。
この構造変換思考は、

- リファクタ（関数名/ロジック差し替え）
- セキュリティ対応（特定パターン一括置換）
- コーディング規約、ログ統一 ...etc

パターン化された大量作業で真価を発揮する。「人が全部やれ」と言われたら心が折れるサイズでも、構造で分割・自動化バッチ化するだけで“つらさ”が消える。使い方次第で「現場仕事の多く」に応用できるはずなので、皆さんもぜひ利用ください
