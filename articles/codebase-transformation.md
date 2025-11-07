---
title: "600ファイル5000箇所の多言語対応を半日で終わらせた話"
emoji: "🚿"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["TypeScript", "AST"]
published: false
publication_name: "dress_code"
---

# TL;DR

- 約600ファイル、5,000箇所への中国語追加を「構造分割＋AST 自動変換」により実装4時間＋実行2時間で完了
- AI に一からやらせると無駄な再計算が生じ非効率だったので、工程を「抽出」「翻訳」「適用」の3つに分割し AST 解析を利用することで効率化を実現
- この手法は多言語対応だけでなく、大規模リファクタリング、セキュリティ対応、コーディング規約統一にも応用可能

# はじめに

こんにちは、DressCode でプロダクトエンジニアをしている[ないとー](https://x.com/_bwkw_)です。

この記事では、AI だけでは解決できなかった大規模コード変更を、AST(抽象構文木)解析によって効率化した実践例を紹介します。同じような課題に直面しているエンジニアの方々の参考になれば幸いです👋

# 背景

私たちは DRESS CODE という、Workforce Management 領域の業務効率化を支援する SaaS を開発しています。現在は東南アジア圏を中心に、日本語・英語・インドネシア語・ベトナム語・タイ語でサービスを提供中です。

![挑戦する事業ドメイン/マーケット](/images/codebase-transformation/domain-market.png)

そんな折、CEO から「中国語簡体字も対応してほしいんだよね〜」と言われ、中国語対応に取り組むことになりました。

# 最初のアプローチ

どれくらい手間がかかるのか見積もるため、まずはコードベースを一通り調査したところ、以下のようなコード群が約600ファイル、5,000箇所に存在していました。

```typescript
export class InvalidWorker extends ErrorWithDisplayMessages {
  constructor(workerId: string) {
    super(`invalid worker. workerId is ${workerId}`, {
      ja: `不正な従業員です。 workerId: ${workerId}`,
      en: `Invalid worker. workerId: ${workerId}`,
      id: `Pekerja tidak valid. workerId: ${workerId}`,
      vi: `Người lao động không hợp lệ. workerId: ${workerId}`,
      th: `คนงานไม่ถูกต้อง workerId: ${workerId}`,
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
    labelVi: "Nam",
    labelTh: "ชาย",
  },
};
```

当初は「まぁ AI に任せればすぐ終わるだろうな〜」と安直に考え、ファイル単位で AI に翻訳と書き換えを行わせるアプローチを試しました。

しかし進めていくうちに以下のような問題が発生し、このアプローチは行き詰まりました。

- 1ファイル処理に5分以上かかる → 600ファイルで計算すると3,000分 ≈ 50時間
- ファイルを処理するたびに処理が重くなる
- 翻訳対象のファイルを適切に探索できない

AI は毎回ファイル全体の構造を解析し、翻訳対象やキーの形状の違いを逐次判断する必要があるため、同じ形式の構造でも無駄な再計算が発生し、効率が著しく低下していました。

# 構造的アプローチへの転換

そこで、AI が無駄な再計算を行わないために、処理を「**抽出**」「**翻訳**」「**適用**」の3工程に分割する方針に切り替えました。

| 工程     | 役割                                     | ツール               |
| -------- | ---------------------------------------- | -------------------- |
| **抽出** | 翻訳すべきテキストをすべて一括で取り出す | ts-morph（AST 解析） |
| **翻訳** | 抽出結果を一括翻訳                       | 翻訳できればなんでも |
| **適用** | 翻訳結果をソースコードに反映             | ts-morph（AST 解析） |

このアプローチにより、以下のメリットが得られます。

- AI が同じ構造を何度も解析しない → **処理時間が大幅に短縮**
- 文脈理解が最小限ですむ → **翻訳精度が向上**
- 各工程を独立して最適化できる → **工程ごとに最適なツールとプロンプト設計を採用可能**

それでは、各工程の詳細を見ていきましょう。

## Step1. 抽出

コードベースを分析すると、多言語対応には大きく2つのパターンがありました。

### パターン1: DictSetObject（オブジェクト型の辞書）

**DictSetObject**は、各言語をキー、テキスト内容を値にした、シンプルなオブジェクトリテラルです。典型的な例は以下のような形です。

```typescript
export class InvalidWorker extends ErrorWithDisplayMessages {
  constructor(workerId: string) {
    super(`invalid worker. workerId is ${workerId}`, {
      ja: `不正な従業員です。 workerId: ${workerId}`,
      en: `Invalid worker. workerId: ${workerId}`,
      id: `Pekerja tidak valid. workerId: ${workerId}`,
      vi: `Người lao động không hợp lệ. workerId: ${workerId}`,
      th: `คนงานไม่ถูกต้อง workerId: ${workerId}`,
    });
  }
}
```

### パターン2: Selection（フォーム選択肢の構造化オブジェクト）

**Selection**は、フォームの選択肢定義などで使われる、構造化された大きめのオブジェクトです。各言語に対応する`label`プロパティが割り当てられています。

```typescript
export const Selections = {
  GENDER_MALE: {
    key: "MALE",
    labelKey: "GENDER_MALE",
    labelDictId: "form-selection.label.GENDER_MALE",
    labelJa: "男性",
    labelEn: "Male",
    labelId: "Pria",
    labelVi: "Nam",
    labelTh: "ชาย",
  },
};
```

まず、これらのパターンを [ts-morph](https://github.com/dsherret/ts-morph) を使って AST で検出し、新言語のフィールドに `__TRANSLATE__` という翻訳用マーカーを挿入します。

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

これにより、翻訳が必要な箇所に以下のような翻訳用マーカーが挿入されます。

```typescript
export class InvalidWorker extends ErrorWithDisplayMessages {
  constructor(workerId: string) {
    super(`invalid worker. workerId is ${workerId}`, {
      ja: `不正な従業員です。 workerId: ${workerId}`,
      en: `Invalid worker. workerId: ${workerId}`,
      id: `Pekerja tidak valid. workerId: ${workerId}`,
      vn: `Người lao động không hợp lệ. workerId: ${workerId}`,
      th: `คนงานไม่ถูกต้อง workerId: ${workerId}`,
      zhCN: __TRANSLATE__, ← ⭐️ 翻訳用マーカーを挿入
    });
  }
}
```

続いて、コードベース全体をもう一度走査し、挿入した翻訳用マーカーを基に必要な情報を Markdown 形式で出力します。

| File        | Line | ja                   | en                | zhCN            |
| ----------- | ---- | -------------------- | ----------------- | --------------- |
| user.dto.ts | 45   | ユーザー名           | Username          | `__TRANSLATE__` |
| error.ts    | 89   | エラーが発生しました | An error occurred | `__TRANSLATE__` |
| ...         | ...  | ...                  | ...               | ...             |

ここでのポイントは以下の2点です。

- 翻訳のブレを防ぐため、日本語と英語の両方を抽出
- AI への並列処理を可能にし、かつ1回あたりのコンテキスト量を最適化するため、100エントリごとに分割して複数の Markdown ファイル（`batch-1.md`, `batch-2.md`, ...）として出力

これにより、5,000箇所の翻訳対象を構造化されたデータとして抽出できました。

## Step2. 翻訳

次に、Step1 で生成した Markdown ファイル群を AI に投げて一括翻訳します。

構造化されたデータを入力として使うことで、高速かつ統一的な翻訳を得られます。また、ファイル単位で背景文脈を制御できるため、同義語の揺れや誤訳も減りやすく、精度の高い翻訳結果を得ることができました。

以下は実際に使用したプロンプトです。

```
project/operation-tools/locale-addition/output/{languageCode}/ 配下の
全てのバッチファイル（batch-1.md から batch-N.md まで）の {languageCode} カラムを
{languageName}で翻訳してください。

翻訳の指針：
- ja（日本語）と en（英語）を参考にする
- ビジネス用語・技術用語は正確に翻訳
- 簡潔で自然な表現にする

各ファイルは100エントリ（最後のファイルは端数）の Markdown テーブル形式です。
batch-1 から順番に翻訳してください。
```

## Step3. 適用

最後に、Step2 で翻訳された Markdown ファイルを読み取り、ソースコード内の `__TRANSLATE__` マーカーを実際の翻訳テキストに置換します。

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

このように工程を「**抽出**」「**翻訳**」「**適用**」に分割することで、当初50時間を想定していた作業を6時間（実装4時間、実行2時間）で完了できました。今後さらに多言語対応が求められる場面が増えてくることを考えると、この手法の効果は非常に大きいと感じています。

# おわりに

この記事では、ts-morph を使った AST 変換による多言語対応の効率化手法を紹介しました。

大規模で複雑に見える作業も、構造的に分割し、タスクごとに自動化・省力化することで着実な成果が得られることを実感しました。このアプローチは多言語対応だけでなく、以下のようなさまざまな課題にも応用できます。

- リファクタリング（関数名やロジックの差し替え）
- セキュリティ対応（パターンの一括置換や検出）
- コーディング規約の統一やログ出力の型統一

AI 時代において、「どの部分を AI に任せ、どこを人が設計・判断すべきか」という線引きが、再現性の高い開発と効率向上の鍵となります。もし似たような課題に悩んでいる方がいれば、タスクの構造化や AI 活用と自動化の線引きについて、ぜひ一度見直してみてください！
