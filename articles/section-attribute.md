---
title: "toB SaaS の詳細画面、全部同じ構造じゃん"
emoji: "⚙️"
type: "tech"
topics: ["設計パターン", "フロントエンド", "バックエンド", "TypeScript", "SaaS"]
published: false
publication_name: "dress_code"
---

# TL;DR

- toB SaaS の詳細画面（従業員プロフィール、拠点情報、顧客企業など）は、見た目は違えど構造は同じ
- この共通構造を **Section / Attribute パターン** として抽出し、バックエンド・フロントエンドで統一することで、新しい詳細画面の追加が「定義を書くだけ」に
- さらに、Graph という概念と組み合わせることで、権限管理も一元的に扱える

# はじめに

こんにちは、Dress Code でプロダクトエンジニアをしている[ないとー](https://x.com/_bwkw_)です。

私たちは [DRESS CODE](https://www.dress-code.com/ja) という toB SaaS を開発しています。人事、情報システム、総務、採用、コーポレートなど、複数の部門を横断する業務 OS として機能するコンパウンドプロダクトです。

![挑戦する事業ドメイン/マーケット](/images/codebase-transformation/domain-market.png)

このようなプロダクトを開発していると、「詳細画面」を作る機会がやたらと多いことに気づきます。従業員のプロフィール、勤務地となる拠点の情報、顧客企業の契約内容、導入している機器の管理情報…。扱うドメインが広いからこそ、詳細画面の数も自然と増えていきます。

これは DRESS CODE に限った話ではありません。EC サイトなら商品詳細・注文詳細・顧客詳細、CRM なら取引先・案件・担当者、プロジェクト管理ツールならプロジェクト・タスク・メンバー。toB SaaS を開発していると、必ずと言っていいほど「〇〇の詳細画面」を大量に作ることになります。

これらの画面、見た目やデザインは違えど、よく見ると構造がほとんど同じなんですよね。例えばこんな画面を想像してみてください。

![詳細画面のイメージ](/images/section-attribute/detail-screen-image.png)

セクションごとにカードがあり、その中にラベルと値のペアが並ぶ。入力欄があって、日付ピッカーがあって、セレクトボックスがある。UIのテイストは違っても、ユーザーが情報を閲覧・編集するという体験の構造は同じです。

でも、多くのプロジェクトでは、これらの画面を毎回ゼロから実装しているのではないでしょうか。この記事では、詳細画面の「よくある実装」が抱える問題と、私たちが試しているアプローチを紹介します。同じような課題を感じている方の参考になれば幸いです👋

# 詳細画面の「よくある実装」

まず、典型的な詳細画面の実装を見てみましょう。先ほどの画像にあった「従業員プロフィール」の詳細画面を例にします。

## バックエンドの実装

バックエンドでは、従業員エンティティを取得して DTO に変換し、API レスポンスとして返します。

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

レスポンスの型も定義します。

```typescript
interface EmployeeProfileResponse {
  // 個人識別情報
  lastName: string;
  firstName: string;
  lastNameKana: string | null;
  firstNameKana: string | null;
  birthDate: Date | null;
  gender: string | null;
  bloodType: string | null;
  nationality: string | null;
  // 連絡先
  email: string | null;
  phoneNumber: string | null;
  // ... さらにフィールドが続く
}
```

## フロントエンドの実装

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

これ自体は普通のコードに見えます。しかし、プロダクトが成長するにつれて、いくつかの問題が顕在化してきます。

# 何が問題なのか？

## 問題1：詳細画面が増えるたびに同じ作業を繰り返す

従業員プロフィールの詳細画面ができました。次は「拠点情報」の詳細画面を作ってください、と言われます。

すると、また同じことをやることになります。拠点エンティティを取得して DTO に変換し、フィールドを持つレスポンス型を定義し、フロントエンドでは各フィールドを個別のコンポーネントで描画する。Card コンポーネントで囲んで、FormItem でラベルを付けて、TextInput や DatePicker や Select を配置して…。

顧客企業の詳細画面も同様です。導入機器の詳細画面も同様です。

詳細画面が増えるたびに、構造は同じなのに、実装はゼロから書いている。これは単純に工数がかかります。

## 問題2：実装者によって構造が揺れる

詳細画面をゼロから実装するということは、実装者の裁量が大きいということでもあります。

フロントエンドでは、ある人は「個人識別情報」と「連絡先」を別々のカードに分ける。別の人は「基本情報」として1つのカードにまとめる。ある人は生年月日を `DatePicker` で実装し、別の人は年月日を3つの `Select` で実装する。

バックエンドでも同様のことが起きます。ある人はフラットな構造でレスポンスを返し、別の人はセクションごとにネストした構造で返す。ある人はフィールドのラベル（「姓」「名」など）をバックエンドのレスポンスに含め、別の人はフロントエンドで定義する。多言語対応も同様で、ある人はバックエンドでローカライズ済みの文字列を返し、別の人はキーだけ返してフロントエンドで翻訳する。

画面が10個、20個と増えていくと、こうした揺れの蓄積がコードベース全体の一貫性を損なっていきます。

## 問題3：フィールドの追加・変更が面倒

「従業員プロフィールに『マイナンバー』というフィールドを追加してほしい」という要望が来たとします。やることは以下の通りです。

1. バックエンドで DTO にフィールドを追加
2. バックエンドで Entity からの変換ロジックを追加
3. API レスポンスの型定義を更新
4. フロントエンドの型定義を更新
5. フロントエンドで FormItem と適切なコンポーネントを追加
6. フロントエンドで保存時の送信ロジックを更新

たった1つのフィールドを追加するのに、複数箇所の変更が必要です。そして、これが「拠点情報にも住所フィールドを追加してほしい」となると、また同じ変更を別の画面で行うことになります。

## 問題4：権限制御の実装がエンドポイントごとにバラつく

「マイナンバーは人事担当者だけが閲覧できるようにしてほしい」という要望が来たとします。当然、権限制御はバックエンドで行います。

しかし、詳細画面ごとに個別のエンドポイントを実装していると、権限制御の仕方もエンドポイントごとにバラつきがちです。ある人はミドルウェアで制御し、別の人はサービス層で制御する。ある人はフィールド単位で出し分け、別の人はエンドポイント自体へのアクセスを制限する。

```typescript
// ある人の実装：Controller で直接チェック
if (!currentUser.isHRAdmin) {
  delete response.myNumber;
}

// 別の人の実装：Service 層でチェック
// また別の人の実装：DTO の変換時にチェック
// さらに別の人の実装：エンドポイント自体に認可をかける
```

詳細画面が増えるたびに、こうした権限制御のロジックがあちこちに散らばっていきます。「このフィールドは誰が閲覧・編集できるんだっけ？」という問いに答えるには、各エンドポイントの実装を一つ一つ確認するしかない。

# 共通構造の発見

ここで、改めていくつかの詳細画面を並べて観察してみます。

従業員プロフィールの詳細画面には「個人識別情報」「連絡先」といったセクションがあり、それぞれの中に「姓」「名」「生年月日」「メールアドレス」といったフィールドが並んでいます。

拠点情報の詳細画面にも「基本情報」「所在地」といったセクションがあり、それぞれの中に「拠点名」「住所」「電話番号」といったフィールドが並んでいます。

顧客企業の詳細画面にも「企業情報」「契約情報」といったセクションがあり…、そう、全部同じ構造なんです。

セクション（カード）があって、その中にフィールド（属性）が並んでいる。フィールドにはラベルがあり、値の型に応じたコンポーネントで描画される。この構造は、どの詳細画面でも変わりません。

であれば、この共通構造を「パターン」として抽出し、再利用可能な形にできるはずです。

# Section / Attribute パターン

私たちはこの共通構造を **Section / Attribute パターン** と名付け、バックエンドとフロントエンドの両方で統一的に扱えるデータ構造として定義しました。

## データ構造

Section は画面上の「カード」や「グループ」に相当し、Attribute は各セクション内の「フィールド」に相当します。先ほどの従業員プロフィールを、この構造で表現すると以下のようになります。

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
          "value": "テスト",
          "valueType": "TEXT",
          "required": true,
          "readonly": false
        },
        {
          "attributeKey": "FirstName",
          "name": "名",
          "value": "太郎",
          "valueType": "TEXT",
          "required": true,
          "readonly": false
        },
        {
          "attributeKey": "BirthDate",
          "name": "生年月日",
          "value": "1990-04-01T00:00:00.000Z",
          "valueType": "DATE",
          "required": false,
          "readonly": false
        },
        {
          "attributeKey": "Gender",
          "name": "性別",
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
          "value": "test001@company.com",
          "valueType": "TEXT",
          "required": false,
          "readonly": false
        },
        {
          "attributeKey": "PhoneNumber",
          "name": "電話番号",
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

ここでのポイントは `valueType` です。各属性には型情報が付与されており、フロントエンドはこの `valueType` を見て適切なコンポーネントを選択します。`TEXT` ならテキストフィールド、`DATE` なら日付ピッカー、`SELECT` ならセレクトボックス、といった具合です。

また、`name` フィールドにはローカライズ済みのラベルが入ります。多言語対応もバックエンド側で解決され、フロントエンドは受け取った `name` をそのまま表示するだけ。問題2で挙げた「ラベルをどこで持つか」「多言語対応をどこでやるか」という揺れも、このパターンによって統一されます。

## フロントエンドでの描画

フロントエンドでは、この `Section[]` を受け取って動的に描画する `AttributeRenderer` を用意します。

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

拠点情報の詳細画面も、顧客企業の詳細画面も、構造は全く同じです。異なるのは、どのセクションにどの属性があるかという「定義」だけ。

# 何が変わるのか？

## フィールド追加の工数が減る

「マイナンバー」フィールドを追加する場合、以前は複数箇所の変更が必要でした。Section / Attribute パターンでは、属性の定義を追加するだけです。バックエンドで「この Section にはこの Attribute がある」という定義を追加すれば、あとは共通の仕組みが自動的に処理してくれます。フロントエンドのコンポーネントを触る必要すらありません。

## バックエンドとフロントエンドの型が一致する

API のレスポンスは常に `Section[]` という構造になるため、画面ごとに異なる型定義を管理する必要がなくなります。「このフィールドの型は何だっけ」「nullable だっけ」といった混乱もなくなります。

## 新しい詳細画面の追加が「定義を書くだけ」になる

新しいエンティティの詳細画面を作るときも、もはやゼロから実装する必要はありません。どのセクションにどの属性があるかを定義すれば、バックエンドもフロントエンドも共通の仕組みが動いてくれます。

## 権限管理への応用

私たちのプロダクトでは、データ構造を「Graph」という概念で管理しています。これは全てのビジネスアプリケーションから共通して利用されるデータであり、「信頼できる唯一の情報源」として設計されたシステムの根幹です。

Section / Attribute パターンにおける `sectionKey` や `attributeKey` は、この Graph の構造を元に定義されます。つまり、UI に表示されるセクションや属性は、Graph というデータ構造と一致しているのです。

この設計により、権限管理を Graph を中心としたものへと昇華させることができます。エンドポイントごとに権限制御を個別に実装するのではなく、Graph の `sectionKey` や `attributeKey` に対して「この属性は誰が閲覧できるか」「誰が編集できるか」を定義する。すると、その定義が Section / Attribute パターンを通じて自動的に UI に反映される。権限のロジックがコードのあちこちに散らばることなく、Graph という一箇所で管理できるようになります。

# おわりに

toB SaaS の詳細画面は、ドメインが違っても構造は驚くほど共通しています。従業員プロフィールも、拠点情報も、顧客企業も、全部「セクションの中に属性が並んでいる」だけ。

この共通性に気づいたとき、Section / Attribute パターンとしてデータ構造を統一することにしました。バックエンドとフロントエンドが同じ構造でやり取りすることで、実装のばらつきがなくなり、新しい詳細画面の追加も定義を書くだけで済むようになります。さらに、Graph という概念と組み合わせることで、権限管理も一元的に扱える可能性が見えてきました。

まだ道半ばではありますが、同じような課題を感じている方の参考になれば幸いです。
