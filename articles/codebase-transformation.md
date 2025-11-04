---
title: "600ファイル、5000箇所の多言語対応を半日で終わらせた話"
emoji: "🚿"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [""]
published: false
publication_name: "dress_code"
---

# TL;DR

- 約600ファイル、5,000箇所への中国語追加を「構造分割＋AST自動変換」により実装4時間＋実行2時間で完了
- AI では無駄な再計算が発生し効率が著しく低下していたので、AST 解析で「抽出→翻訳→適用」に分割することで桁違いの効率化を実現
- この手法は多言語対応だけでなく、大規模リファクタリング、セキュリティ対応、コーディング規約統一にも応用可能

# はじめに

こんにちは、DressCode でプロダクトエンジニアをしているないとーです。

この記事では、AI だけでは解決できなかった大規模コード変更を、AST(抽象構文木)解析によって効率化した実践例を紹介します。同じような課題に直面しているエンジニアの方々の参考になれば幸いです👋

# 背景

私たちは DRESS CODE という、Workforce Management 領域の業務効率化を支援する SaaS を開発しています。現在は東南アジア圏を中心に、日本語・英語・インドネシア語・ベトナム語・タイ語でサービスを提供中です。

![挑戦する事業ドメイン/マーケット](/images/codebase-transformation/domain-market.png)

そんな折、CEO から「中国語簡体字も対応してほしいんだよね〜」と言われ、中国語対応に取り組むことになりました。

# 最初のアプローチ

どれぐらい手間がかかるのか見積もるため、まずはコードベースを一通り調査してみたところ、下記のようなコード群が約600ファイル、5,000箇所存在していました。

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

export const Selections = {
  GENDER_MALE: {
    key: "MALE",
    labelKey: "GENDER_MALE",
    labelDictId: `${DictSchema.FormSelection.resource}.label.GENDER_MALE`,
    labelJa: "男性",
    labelEn: "Male",
    labelId: "Pria",
    labelTh: "ชาย",
    labelVn: "Nam",
    labelZhCN: "男性",
  },
};
```

当初は「まぁAIに任せればすぐ終わるだろうな〜」と安直に考え、ファイル単位でAIに翻訳と書き換えを行わせるアプローチを試しました。

しかし進めていくうちに以下のような問題が発生し、現実的ではないなと言う判断になりました...

- 1ファイル処理に5分以上かかる → 600ファイルで計算すると3,000分 ≈ 50時間
- ファイルを処理するたびに処理が重くなる → APIのコンテキスト管理が複雑化
- 翻訳対象のファイルを適切に探索できない → 漏れと重複が発生

AI は毎回ファイル全体の構造を解析し、翻訳対象やキーの形状の違いを逐次判断する必要があるため、同じ形式の構造でも無駄な再計算が発生し、効率が著しく低下していました。

# 構造的アプローチへの転換

そこで、AIが無駄な再計算を行わないために、処理を「**抽出**」「**翻訳**」「**適用**」の3工程に分割する方針に切り替えました。

| 工程 | 役割                                     | ツール               |
| ---- | ---------------------------------------- | -------------------- |
| 抽出 | 翻訳すべきテキストをすべて一括で取り出す | ts-morph（AST解析）  |
| 翻訳 | 抽出結果をAIで一括翻訳                   | 翻訳できればなんでも |
| 適用 | 翻訳結果をソースコードに反映             | ts-morph（AST解析）  |

このアプローチにより、以下のメリットが得られます。

- AI が同じ構造を何度も解析しない → 処理時間が大幅に短縮
- 文脈理解が最小限ですむ → 翻訳精度が向上
- 各工程を独立して最適化できる → 工程ごとに最適なツールとプロンプト設計を採用可能

結果として、作業効率も品質も大幅に改善しました。

## Step1. 抽出

コードベースを分析すると、多言語対応には大きく2つのパターンがありました。

抽出フェーズでは、これらのパターンを認識し、

1. 多言語対応必要箇所に `__TRANSLATE__` という翻訳マーカーを挿入
2. 挿入された `__TRANSLATE__` を基に翻訳しやすい Markdown テーブル形式に変換(抽出)

を行います。

### パターン1: DictSetObject（オブジェクト型の辞書）

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

### パターン2: Selection（フォーム選択肢の構造化オブジェクト）

**Selection**は、フォームの選択肢定義などで使われる、構造化された大きめのオブジェクトです。各言語ごとに`label`プロパティが割り当てられています。

```typescript
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

まずは、これらのパターンを [ts-morph](https://github.com/dsherret/ts-morph) を使って AST で検出し、新言語のフィールドに `__TRANSLATE__` という文字列を挿入します。`ts-morph` は TypeScriptのAST(抽象構文木)を JavaScript/TypeScript で自在に操作できるライブラリです。以下はコードサンプルです。

```typescript
import { Project, SyntaxKind } from "ts-morph";

const project = new Project({
  tsConfigFilePath: "./tsconfig.json",
});

// 対象ファイルを再帰的に探索
const sourceFiles = project.getSourceFiles("src/**/*.ts");

sourceFiles.forEach((sourceFile) => {
  // DictSetObjectパターンの検出と処理
  const objectLiterals = sourceFile.getDescendantsOfKind(
    SyntaxKind.ObjectLiteralExpression
  );

  objectLiterals.forEach((obj) => {
    const properties = obj.getProperties();
    const hasLangKeys = ["ja", "en", "id", "vn", "th"].every((lang) =>
      properties.some((p) => p.getName() === lang)
    );

    if (hasLangKeys && !properties.some((p) => p.getName() === "zhCN")) {
      // 翻訳マーカーを挿入
      obj.addPropertyAssignment({
        name: "zhCN",
        initializer: "'__TRANSLATE__'",
      });
    }
  });

  sourceFile.saveSync();
});
```

続いて、翻訳用マーカーを見つけてその位置を基に Markdown 形式でまとめて出力します。

| File        | Line | ja                   | en                | zhCN            |
| ----------- | ---- | -------------------- | ----------------- | --------------- |
| user.dto.ts | 45   | ユーザー名           | Username          | `__TRANSLATE__` |
| error.ts    | 89   | エラーが発生しました | An error occurred | `__TRANSLATE__` |

ここでのポイントは以下の通りです。

- 翻訳のブレを防ぐために日本語と英語両方を抽出（後続での翻訳に活かす）
- 並列での翻訳を可能にする＆1ファイルあたりのコンテキスト量調整のため100エントリ単位で分解

## Step2. 翻訳

翻訳処理そのものの詳細は割愛しますが、抽出済み Markdown を AI にバッチで投げることで、高速かつ統一的な翻訳を得られます。

背景文脈を制御できるため、同義語の揺れや誤訳も減りやすく、精度の高い翻訳結果を得ることができました。

## Step3. 適用

最後に、Markdown から翻訳を読み取り、ソースコードの翻訳マーカーを置換します。これも `ts-morph` を使って実装しています。

```typescript
import fs from "fs";
import { Project } from "ts-morph";

// Markdown テーブルをパース
const markdownContent = fs.readFileSync("translations.md", "utf-8");
const translations = parseMarkdownTable(markdownContent);

// ts-morph でプロジェクトを開く
const project = new Project({
  tsConfigFilePath: "./tsconfig.json",
});

const sourceFiles = project.getSourceFiles("src/**/*.ts");

sourceFiles.forEach((sourceFile) => {
  const objectLiterals = sourceFile.getDescendantsOfKind(
    SyntaxKind.ObjectLiteralExpression
  );

  objectLiterals.forEach((obj) => {
    const zhCnProp = obj.getProperty("zhCN");

    if (zhCnProp?.getText().includes("__TRANSLATE__")) {
      // マーク位置から対応する翻訳を検索
      const key = obj.getProperty("labelKey")?.getText();
      const translation = translations.find((t) => t.key === key);

      if (translation) {
        // マーカーを翻訳結果に置換
        const initializer = zhCnProp.getFirstChildByKind(
          SyntaxKind.StringLiteral
        );
        initializer?.replaceWithText(`'${translation.zhCN}'`);
      }
    }
  });

  sourceFile.saveSync();
});
```

このように工程を、「抽出」「翻訳」「適用」に分割して進めることで、実装には4時間、実行時間は2時間という投資で大幅な時間削減を実現できました。（後続での言語追加も考えると尚更ですね！）

| アプローチ       | 実装時間 | 実行時間        | 総時間  |
| ---------------- | -------- | --------------- | ------- |
| AI のみ          | -        | ~50時間（推定） | ~50時間 |
| 構造的アプローチ | 4時間    | 2時間           | 6時間   |

# おわりに

この記事では、`ts-morph` を使った AST 変換による多言語対応の効率化手法を紹介しました。大掛かりで面倒に思える作業も、構造的に分割して自動化することで現実的な解決策が見つかります。

「AI に任せておけばなんとかなる」と考えていた頃よりも、アプローチを見直したことで、作業効率は大きく向上しました。また、このアプローチは、多言語対応だけでなく以下のような領域にも応用可能です。

- リファクタ（関数名/ロジック差し替え）
- セキュリティ対応（特定パターン一括置換）
- コーディング規約、ログ統一 ...etc

もし似たような課題に悩んでいる方がいれば、「この作業、もっと構造的に分けられないか？」と一度立ち止まって考えてみてください！👋
