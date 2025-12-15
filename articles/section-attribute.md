---
title: "画面が増え続けるプロダクトの Server-Driven UI という選択肢"
emoji: "🧩"
type: "tech"
topics:
  [
    "ServerDrivenUI",
    "設計パターン",
    "フロントエンド",
    "バックエンド",
    "TypeScript",
  ]
published: false
publication_name: "dress_code"
---

# TL;DR

# はじめに

この記事は [Dress Code Advent Calendar 2025](https://adventar.org/calendars/12017) の 19 日目の記事です。

こんにちは、Dress Code でプロダクトエンジニアをしている[ないとー](https://x.com/_bwkw_)です！私は [DRESS CODE](https://www.dress-code.com/ja) という、人事、情報システム、総務、採用、コーポレートなど、複数の部門を横断する業務 OS として機能するコンパウンドプロダクトを開発しています。

![挑戦する事業ドメイン/マーケット](/images/codebase-transformation/domain-market.png)

このようなプロダクトを開発していると、「詳細画面」を作る機会がやたらと多いことに気づきます。従業員のプロフィール、勤務地となる拠点の情報、導入している機器の管理情報など、扱うドメインが広いからこそ詳細画面の数も自然と増えていきます。これは DRESS CODE に限った話ではなく、EC サイトなら商品詳細・注文詳細・顧客詳細、CRM なら取引先・案件・担当者、プロジェクト管理ツールならプロジェクト・タスク・メンバーといったように、どんなプロダクトでも「〇〇の詳細画面」を大量に作ることになるのではないでしょうか。

これらの画面は見た目やデザインは違えど、よく見ると構造がほとんど同じです。例えば DRESS CODE の従業員プロフィールの詳細画面はこのような画面ですが、セクションごとにカードがあり、その中にラベルと値のペアが並ぶという構造は他の詳細画面と変わりません。

![詳細画面のイメージ](/images/section-attribute/detail-screen-image.png)

この共通構造を **Server-Driven UI** としてバックエンドから制御すれば、画面が増えても破綻しない設計ができるのではないか。この記事では、その設計判断と実装アプローチを紹介します。同じような課題を感じている方の参考になれば幸いです👋

# よくある実装とその課題

まず、典型的な詳細画面の実装を見てみましょう。先ほどの従業員プロフィールの詳細画面を例にします。

## バックエンドとフロントエンドの実装

バックエンドでは、従業員 `Entity` を取得して `DTO` に変換し、API レスポンスとして返します。

```typescript
// Controller
@Get(':id')
async getEmployeeProfile(@Param('id') id: string) {
  const employee = await this.employeeRepository.findById(id);
  return {
    // 個人識別情報
    lastName: employee.lastName,
    firstName: employee.firstName,
    lastNameKana: employee.lastNameKana,
    firstNameKana: employee.firstNameKana,
    birthDate: employee.birthDate,
    gender: employee.gender,
    bloodType: employee.bloodType,
    nationality: employee.nationality,
    // 連絡先
    email: employee.email,
    phoneNumber: employee.phoneNumber,
    // ... さらにフィールドが続く
  };
}
```

フロントエンドでは、このレスポンスを受け取って各フィールドを描画します。

```tsx
function EmployeeProfileDetail({ data }: Props) {
  return (
    <div>
      <Card title="個人識別情報">
        <FormItem label="姓" required>
          <TextInput value={data.lastName} onChange={...} />
        </FormItem>
        <FormItem label="名" required>
          <TextInput value={data.firstName} onChange={...} />
        </FormItem>
        <FormItem label="生年月日">
          <DatePicker value={data.birthDate} onChange={...} />
        </FormItem>
        <FormItem label="性別">
          <Select value={data.gender} options={genderOptions} onChange={...} />
        </FormItem>
        <FormItem label="血液型">
          <Select value={data.bloodType} options={bloodTypeOptions} onChange={...} />
        </FormItem>
        <FormItem label="国籍">
          <Select value={data.nationality} options={nationalityOptions} onChange={...} />
        </FormItem>
      </Card>

      <Card title="連絡先">
        <FormItem label="メールアドレス">
          <TextInput value={data.email} onChange={...} />
        </FormItem>
        <FormItem label="電話番号">
          <TextInput value={data.phoneNumber} onChange={...} />
        </FormItem>
      </Card>

      {/* さらにカードが続く... */}
    </div>
  );
}
```

これ自体は普通のコードです。しかしプロダクトが成長するにつれて、いくつかの課題が見えてきます。

## 課題1：詳細画面が増えるたびに同じ作業を繰り返す

従業員プロフィールの詳細画面ができたら、次は拠点情報の詳細画面を作ることになります。すると、また同じことの繰り返しです。

拠点エンティティを取得して `DTO` に変換し、レスポンス型を定義し、フロントエンドで各フィールドを描画する。`Card` で囲んで、`FormItem` でラベルを付けて、`TextInput` や `DatePicker` や `Select` を配置して...拠点情報も、導入機器も同様です。構造は同じなのに、実装はゼロから書いていては工数がかかります。

## 課題2：実装者によって構造が揺れる

詳細画面をゼロから実装するということは、実装者の裁量が大きいということでもあります。

フロントエンドでは、ある人は「個人識別情報」と「連絡先」を別々のカードに分け、別の人は「基本情報」として1つにまとめる。生年月日を `DatePicker` で実装する人もいれば、年月日を3つの `Select` で実装する人もいる。

バックエンドでも同じです。ある人はフラットな構造でレスポンスを返し、別の人はセクションごとにネストする。多言語対応も、バックエンドでローカライズ済みの文字列を返す人もいれば、キーだけ返してフロントエンドで翻訳する人もいる。画面が10個、20個と増えていくと、こうした揺れがコードベース全体の一貫性を損なっていきます。

## 課題3：フィールドの追加・変更が面倒

「従業員プロフィールにマイナンバーのフィールドを追加してほしい」という要望が来たとします。

やることは、バックエンドで `DTO` にフィールドを追加し、`Entity` からの変換ロジックを追加し、API レスポンスの型定義を更新し、フロントエンドの型定義を更新し、`FormItem` と適切なコンポーネントを追加し、保存時の送信ロジックを更新する...たった1つのフィールドを追加するのに、複数箇所の変更が必要です。そして、これが「拠点情報にも住所フィールドを追加してほしい」となると、また同じ変更を別の画面で行うことになります。

## 課題4：権限制御の実装がエンドポイントごとにバラつく

「マイナンバーは人事担当者だけが閲覧できるようにしてほしい」という要望が来たとします。

当然、権限制御はバックエンドで行います。しかし、詳細画面ごとにレスポンスの型が異なると、権限制御の実装も画面ごとに異なるものになりがちです。ミドルウェアで制御する人もいれば、サービス層で制御する人もいる、フィールド単位で出し分ける人もいれば、エンドポイント自体へのアクセスを制限する人もいる。

詳細画面が増えるたびに権限制御のロジックがあちこちに散らばり、「このフィールドは誰が閲覧・編集できるんだっけ？」という問いに答えるには、各エンドポイントの実装を一つ一つ確認するしかありません。

# Server-Driven UI という選択

## なぜ Server-Driven UI なのか

ここで、改めて詳細画面を観察してみます。従業員プロフィールも、拠点情報も、導入機器も、結局は「セクション（カード）の中にフィールド（属性）が並んでいる」という同じ構造です。

Server-Driven UI とは、UI の構造やコンテンツをサーバー側で定義し、クライアントはその定義に従って描画するアーキテクチャです。Airbnb や Shopify などでも採用されています。詳細画面の構造が共通しているなら、この共通構造をバックエンドから返すようにすれば、フロントエンドは「受け取った構造を描画するだけ」になります。

私はこの共通構造を **Section / Attribute パターン** と名付け、Server-Driven UI として実装しました。

## バックエンドの API データ構造

`Section` は画面上の「カード」に相当し、`Attribute` は各セクション内の「フィールド」に相当します。従業員プロフィールをこの構造で表現するとこうなります。

```json
{
  "sections": [
    {
      "sectionKey": "PersonalIdentification",
      "name": "個人識別情報",
      "displayOrder": 1,
      "attributes": [
        {
          "attributeKey": "LastName",
          "name": "姓",
          "displayOrder": 1,
          "value": "テスト",
          "valueType": "TEXT",
          "required": true,
          "readonly": false
        },
        {
          "attributeKey": "FirstName",
          "name": "名",
          "displayOrder": 2,
          "value": "太郎",
          "valueType": "TEXT",
          "required": true,
          "readonly": false
        },
        {
          "attributeKey": "BirthDate",
          "name": "生年月日",
          "displayOrder": 3,
          "value": "1990-04-01T00:00:00.000Z",
          "valueType": "DATE",
          "required": false,
          "readonly": false
        },
        {
          "attributeKey": "Gender",
          "name": "性別",
          "displayOrder": 4,
          "value": "MALE",
          "valueType": "SELECT",
          "required": false,
          "readonly": false
        }
      ]
    },
    {
      "sectionKey": "Contact",
      "name": "連絡先",
      "displayOrder": 2,
      "attributes": [
        {
          "attributeKey": "Email",
          "name": "メールアドレス",
          "displayOrder": 1,
          "value": "test001@company.com",
          "valueType": "TEXT",
          "required": false,
          "readonly": false
        },
        {
          "attributeKey": "PhoneNumber",
          "name": "電話番号",
          "displayOrder": 2,
          "value": "09011112222",
          "valueType": "TEXT",
          "required": false,
          "readonly": false
        }
      ]
    }
  ]
}
```

ポイントは `valueType` です。フロントエンドはこの値を見て適切なコンポーネントを選択します。`TEXT` ならテキストフィールド、`DATE` なら日付ピッカー、`SELECT` ならセレクトボックス。`name` にはローカライズ済みのラベルが入るため、多言語対応もバックエンド側で完結します。

## フロントエンドでの描画

フロントエンドでは、`Section[]` を受け取って動的に描画する `AttributeRenderer` を用意します。

```tsx
function AttributeRenderer({ attributes, sectionKey }: Props) {
  return (
    <>
      {attributes.map((attr) => (
        <FormItem key={attr.attributeKey} label={attr.name} required={attr.required}>
          {renderComponent(attr)}
        </FormItem>
      ))}
    </>
  );
}

function renderComponent(attr: Attribute) {
  switch (attr.valueType) {
    case 'TEXT':
      return <TextInput value={attr.value} onChange={...} />;
    case 'DATE':
      return <DatePicker value={attr.value} onChange={...} />;
    case 'SELECT':
      return <Select value={attr.value} onChange={...} />;
    // ...
  }
}
```

これにより、詳細画面のコンポーネントはシンプルになります。

```tsx
function EmployeeProfileDetail() {
  const { data } = useEmployeeProfile();

  return (
    <div>
      {data.sections.map((section) => (
        <Card key={section.sectionKey} title={section.name}>
          <AttributeRenderer
            attributes={section.attributes}
            sectionKey={section.sectionKey}
          />
        </Card>
      ))}
    </div>
  );
}
```

拠点情報も、導入機器も、このコンポーネントをそのまま使えます。違うのは、どのセクションにどの属性があるかという「定義」だけです。

# Server-Driven UI がもたらす変化

## 課題1〜3の解決：工数削減と構造の統一

API のレスポンスが常に `Section[]` という構造になることで、課題1〜3はまとめて解決します。

新しい詳細画面を作るとき、フロントエンドのコンポーネントはそのまま使い回せます。バックエンドでどのセクションにどの属性があるかを定義し、変換・保存処理を書けば画面が動きます。フィールドを追加する場合も、属性の定義を追加するだけです。また、フラットにするかネストにするか、ラベルをどこで持つか、多言語対応をどこでやるか、といった判断の余地がなくなり、全ての詳細画面が同じ構造で統一されます。

これが Server-Driven UI の強みです。UI の構造がバックエンドで一元管理されるため、画面が10個、20個と増えても、フロントエンドのコードは増えません。

## 課題4の解決：権限管理の一元化

課題4の「権限制御がエンドポイントごとにバラつく」問題も、Server-Driven UI によって解決に向かいます。

私たちのプロダクトでは、ビジネスデータを「Graph」という概念で管理しています。Graph とは、全てのビジネスアプリケーションから共通して利用されるコアデータベースを中心とした統合データ構造です。いわゆる SSOT（Single Source of Truth = 信頼できる唯一の情報源）として機能し、組織内のすべてのデータが1つの場所で管理・更新されるように設計されています。

Section / Attribute パターンの `sectionKey` や `attributeKey` は、この Graph の構造を元に定義しています。つまり、UI に表示されるセクションや属性は、Graph というデータ構造と一致しています。

この設計により、権限管理を Graph 中心のものへと昇華させることができます。エンドポイントごとに権限制御を個別実装するのではなく、Graph の `sectionKey` や `attributeKey` に対して「誰が閲覧できるか」「誰が編集できるか」を定義する。レスポンスの型が `Section[]` で統一されているため、権限制御の実装も統一できます。

# Server-Driven UI のトレードオフ

とはいえ、Server-Driven UI は万能ではありません。向いているケースと向いていないケースがあります。

**向いているケース**

- 定型的な CRUD 画面（従業員プロフィール、拠点情報、導入機器など）
- 「ラベル + 入力欄」が並ぶシンプルなフォーム
- 項目数が多く、画面ごとに個別実装するのが大変な場合

**向いていないケース**

- 複雑なレイアウト（左右配置、タブ切り替え、条件分岐による表示切り替えなど）
- フィールド間の相関バリデーション（「開始日 < 終了日」など）が多い画面

Server-Driven UI は、フロントエンドの柔軟性を一部犠牲にする代わりに、定型的な画面の開発効率と一貫性を得るトレードオフです。全ての画面に適用するのではなく、「この部分は定型的だから Server-Driven UI を使う」「この部分はリッチな体験が必要だから個別実装する」と使い分けるのが現実的です。

# おわりに

詳細画面は、ドメインが違っても「セクションの中に属性が並ぶ」という共通構造を持っています。この共通性に気づき、Server-Driven UI として API レスポンスを `Section[]` で統一する設計を採用しました。

画面が増えても破綻しない。これが Server-Driven UI を選んだ理由です。フロントエンドのコードを増やさずに画面を増やせる設計は、プロダクトがスケールしていく上で大きな武器になると考えています。

全ての画面に適用できるわけではありませんが、定型的な詳細画面が多いプロダクトでは効果を発揮する設計判断です。まだ道半ばではありますが、同じような課題を感じている方の参考になれば幸いです👋
