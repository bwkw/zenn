---
title: "君たちの知らないAPIの話をしよう"
emoji: "👺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["API", "APIデザインパターン"]
published: false
publication_name: "levtech"
---

:::message
この記事では、**Web API** にフォーカスして話を進めていきます。
一部「**API**」と表現している部分がありますが、それらは全て「**Web API**」として解釈いただけますと幸いです。
:::

# TL;DR
- API設計の学習に、「APIデザインパターン」を読んだ
- 良いAPIは、実行可能であり、表現力があり、シンプルで予測可能なもの
- API設計は、RESTの原則に沿って直感的に出来ることが多いが、デザインパターンを学ぶことで複雑な要件に対応することが可能
- 今回は、数あるデザインパターンの中で、カスタムメソッド、ロングランオペレーション、再実行可能ジョブ、シングルトンサブリソースの4つを紹介

# はじめに
こんにちは、エンジニア歴2年目に入り、後輩の入社に怯えている内藤です。

先日、API設計について体系的に学べる本がないかな〜って探してて、先輩エンジニアに聞いたら、「APIデザインパターン」を紹介いただきました。

https://techplay.jp/book/373

この本は、名前の通りAPIのデザインパターンを紹介している本なのですが、この本を読んだ正直な感想としては「めちゃくちゃ使うってほどでもないな...w」でした。ただ、特定の複雑な要件や特殊な状況に対応する際には、改めて読み返す価値のある一冊だなとも思いました。

そこで、この記事では、APIの基本からいくつかのデザインパターンに触れ、その有用性について整理できたらな〜と思います！

# そもそも良いAPIとは？
良いAPIを設計するためには、以下の4つの特性を網羅している必要があります。

### 実行可能であること
良いAPIは、システムがユーザーの要求を迅速かつ正確に満たす能力を持つべきです。これは、ユーザーが求める操作を効率的に、かつ適切に実行できることを意味します。

### 表現力があること
表現力豊かなAPIは、ユーザーが望む機能をシンプルかつ直感的に表現できることを意味します。つまり、ユーザーは少ない努力で目的を達成できるべきです。

### シンプルであること
シンプルさは、APIの要素数を単純に減らすことではありません。むしろ、ユーザーが必要とする機能を最も直接的かつ簡単な方法で利用できるようにすることが求められます。

### 予測可能であること
APIは、その動作が一貫しており、ユーザーが直感的に理解できるようなものでなければなりません。これにより、使用時の不意打ちがなく、信頼性の高いインターフェイスを提供します。

# よく見るAPI
Webサービスで一番馴染みのあるAPIの設計思想はRESTかと思います。

RESTの設計は以下の四つの基本原則に基づいています。
- アドレス可能性 : 各リソースは一意のURIで識別される
- ステートレス性 : サーバーとクライアント間の通信はステートレスである
- 接続性 : リソース間の関係をリンクを通じて表現できる
- 統一インターフェース : HTTPメソッドの使い分け(GET、POST、PUT、DELETEなど)によって、一貫したインターフェースでさまざまな操作を実現できる

このRESTの原則に沿った典型的なAPIのエンドポイント設計の例を以下に示します。

| 目的          | エンドポイント                            | メソッド      |
|-------------|------------------------------------|-----------|
| ユーザの一覧取得    | http://api.example.com/users       | GET       |
| ユーザの新起登録    | http://api.example.com/users       | POST      |
| 特定のユーザの情報取得 | http://api.example.com/users/:id	  | GET       |
| ユーザの情報更新    | http://api.example.com/users/:id   | PUT/PATCH |
| ユーザの情報の削除   | 	http://api.example.com/users/:id	 | DELETE    |

このように、RESTの設計原則に従ってAPIを構築することで、ほとんどのAPI設計は直感的に、かつ問題なく行うことができます。

# デザインパターンの紹介
ここからが本題です。大抵の場合、上の例で示したようなAPI設計で十分です。ただ、いくつかの複雑な要件では、デザインパターンが必要になることがあります。

## カスタムメソッド
### 概要
カスタムメソッドは、標準的なCRUD操作（作成、読み取り、更新、削除）では対応できない特定の操作が必要になる場合に便利です。例えば、メールの送信や即時の文書翻訳など、通常の `create` や `update` メソッドでは処理が難しい操作がこれに該当します。

以下にGoogleが出しているカスタムメソッドの記事を示します。
https://cloud.google.com/apis/design/custom_methods?hl=ja

### 実装例
以下は、カスタムメソッドを扱うAPIの実装例です。`EmailApi` 抽象クラスを介して、特定のユーザーにメールを送信する方法を示しています。
この例の注目点は、エンドポイントの定義における:sendの使用です。ここでの `:send` は、カスタムメソッドの命名規則を示しており、標準的なHTTPメソッド（GET、POSTなど）に対する補足的な操作を表します。

※ デザインパターンのコードを引用しています。
> ```typescript
> abstract class EmailApi {
>   @post("/{id=users/*emails/*}:send")
>   abstract SendEmail(req: SendEmailRequest): Email;
> }
> 
> interface Email {
>   id: string;
>   subject: string;
>   content: string;
>   state: string;
>   deleted: boolean;
>   // ...
> }
> ```

## ロングランオペレーション（Long Running Operations, LRO）
### 概要
LROは、完了までに時間がかかる処理を指し、現代のプログラミング言語では非同期動作を通じてこれをサポートしています。これにより、時間を要する処理を行いつつ、他の作業を同時に進めることができます。非同期処理では、作業の開始時にプレースホルダーを返し、作業完了時には結果と共にプレースホルダーを「解決」するか、エラー発生時に「拒否」します。これにより、結果を非同期に処理したり、結果が返されるまで待機したりすることが可能です。

https://cloud.google.com/translate/docs/advanced/long-running-operation?hl=ja

### 対象とする問題
APIが複雑な作業や大量のデータを扱う場合、簡単なAPI設計では迅速な操作と時間を要する操作の間で一貫性を保つことが困難です。そのため、時間がかかる作業は特別な取り扱いが必要になります。

### 問題点
ロングランオペレーション（LRO）の管理には、操作の進行状況を監視する過程が複雑化する可能性があります。APIがデータを処理するにあたり、ユーザーは進捗に関する定期的な更新を求めます。しかしながら、プロセスがクラッシュしたり、ネットワークが切断されたりするなど、何らかの理由で接続が失われた場合、既存の進捗情報へのアクセスが困難になり、進行状況の監視を再開することが難しくなります。

### 実装例
以下は、LROを扱うAPIの実装例です。この例では、 `ChatRoomApi` 抽象クラスを利用して、進行中の操作の状態を取得する方法と、必要に応じて操作をキャンセルする方法を紹介します。LROを管理するためには、`Operation` リソースの定義が不可欠です。このプロセスには、`Operation` オブジェクトの生成、進行状態の追跡、そして操作の一時停止、再開、キャンセルを可能にするメソッドが含まれます。示されているコードスニペットは、これらの機能の一部を実装する方法を示しています。

```typescript
abstract class ChatRoomApi {
  @get("/{id=operations/*}")
  abstract GetOperation<ResultT, MetadataT>(req: GetOperationRequest):
    Operation<ResultT, MetadataT>;

  @get("/{id=operations/*}:wait")
  abstract WaitOperation<ResultT, MetadataT>(req: WaitOperationRequest):
    Operation<ResultT, MetadataT>;

  @post("/{id=operations/*}:cancel")
  abstract CancelOperation<ResultT, MetadataT>(req: CancelOperationRequest):
    Operation<ResultT, MetadataT>;
}

interface Operation<ResultT, MetadataT> {
  id: string;
  done: boolean;
  expireTime: Date;
  result?: ResultT | OperationError;
  metadata?: MetadataT;
}

interface OperationError {
  code: string;
  message: string;
  details?: any;
}

interface GetOperationRequest {
  id: string;
}

interface WaitOperationRequest {
  id: string;
}

interface CancelOperationRequest {
  id: string;
}
```

## 再実行可能ジョブ
### 概要
再実行可能ジョブは、APIのオンデマンド実行機能を初期設定と実行の二つの主要なステップに分ける特別なリソースです。従来のAPIメソッドでは、メソッド呼び出し時に必要な設定を提供し、即座に処理が行われます。一方、再実行可能ジョブでは、処理を実行するための設定を使ってジョブをまず作成し、次に `run` というカスタムメソッドで実際の処理を行います。

https://cloud.google.com/dataproc/docs/concepts/jobs/restartable-jobs?hl=ja

### 対象とする問題
クライアントがAPIメソッドを非同期に呼び出す必要があるという前提のもと、前述のLROでは解決しない場合があります。以下のような事例はその最たる例です。

#### オンデマンドメソッドの実行
設定と実行の権限を分ける必要性が生じ、特に開発者と運用チームの間で権限を区別する場合に有用です。

#### 定期的な自動実行の要望
APIが自身でスケジュールを設定し、外部システムなしでメソッドを実行できるようにすることで、シンプルかつ安全な操作を可能にします。

### 問題点
ジョブのライフサイクル管理（作成、実行、監視、再実行など）をAPIに組み込むことで、システムの複雑さが増します。

### 実装例
以下は、 `ChatRoomApi` 抽象クラスを利用した再実行可能ジョブのAPI実装例です。ジョブの定義と、実行用のカスタム `run` メソッドを備えています。`CreateAnalyzeChatRoomJob` で分析ジョブを設定し、`RunAnalyzeChatRoomJob` で非同期にメッセージ分析を実行します。

```typescript
abstract class ChatRoomApi {
  @post("/analyzeChatRoomJobs")
  abstract CreateAnalyzeChatRoomJob(req: CreateAnalyzeChatRoomJobRequest):
    AnalyzeChatRoomJob;

  @post("/{id=analyzeChatRoomJobs/*}:run")
  abstract RunAnalyzeChatRoomJob(req: RunAnalyzeChatRoomJobRequest):
    Operation<AnalyzeChatRoomJobExecution, RunAnalyzeChatRoomJobMetadata>;
}

interface CreateAnalyzeChatRoomJobRequest {
  resource: AnalyzeChatRoomJob;
}

interface AnalyzeChatRoomJob {
  id: string;
  chatRoom: string;
  destination: string;
  compressionFormat: string;
}

interface RunAnalyzeChatRoomJobRequest {
  id: string;
}

interface AnalyzeChatRoomJobExecution {
  id: string;
  job: AnalyzeChatRoomJob;
  sentenceComplexity: number;
  sentiment: number;
  abuseScore: number;
}

interface RunAnalyzeChatRoomJobMetadata {
  messagesProcessed: number;
  messagesCounted: number;
}
```

## シングルトンサブリソース
### 概要
シングルトンサブリソースは、あるリソースの一部を独立したエンティティとして設計する手法です。この方法では、リソースの特定の構成要素を単純なプロパティではなく、リソースとプロパティの中間に位置づけ、その要素に独立性を持たせます。

https://cloud.google.com/apis/design/design_patterns?hl=ja#singleton_resources

### 対象とする問題
API設計時、リソースの構成要素がプロパティに適しているにもかかわらず、通常のプロパティとして扱えない場合があります。 以下のような事例はその最たる例です。

#### サイズや複雑さ
単一の構成要素がリソース全体よりも大きく複雑になる場合、リソースから分離するべきです。例えば、大きなバイナリオブジェクトの保存では、バイナリデータをメタデータと共に保持することは少ないです。

#### セキュリティ
リソースの一部が異なるアクセス制限を必要とする場合、それをリソースから完全に分離することが重要です。例えば、従業員データのAPIでは、報酬情報が一般的な情報よりも厳格なアクセス制限を受ける可能性があります。

#### ボラティリティ
リソースの特定構成要素が通常と異なるアクセスパターンを持つ場合、頻繁に更新される要素は他と分離し、独立して更新できるようにするべきです。例えば、ライドシェアのAPIでドライバーの位置情報は頻繁に更新されますが、ナンバープレートなどの情報はそうではありません。

### 問題点
シングルトンサブリソースを導入すると、リソースとサブリソースをアトミックに操作することができなくなる場合があります。これは、サブリソースを独立させることにより、親リソースと同時に操作することが難しくなるためです。

### 実装例
以下は、ライドシェアリングサービスのAPIにおけるシングルトンサブリソースの実装例です。`RideSharingApi` 抽象クラスを用いて、ドライバーの詳細情報や位置情報の取得・更新を行います。具体的には、ドライバーの情報とその位置情報を別々に管理するエンドポイントを提供しています。

```typescript
abstract class RideSharingApi {
  @get("/{id=drivers/*}")
  abstract GetDriver(req: GetDriverRequest): Driver;

  @patch("/{resource.id=drivers/*}")
  abstract UpdateDriver(req: UpdateDriverRequest): Driver;

  @get("/{id=drivers/*/location}")
  abstract GetDriverLocation(req: GetDriverLocationRequest): DriverLocation;

  @patch("/{resource.id=drivers/*/location}")
  abstract UpdateDriverLocation(req: UpdateDriverLocationRequest): DriverLocation;
}

interface Driver {
  id: string;
  name: string;
  licensePlate: string;
}

interface DriverLocation {
  id: string;
  lat: number;
  long: number;
  updateTime: Date;
}

interface GetDriverLocationRequest {
  id: string;
}

interface UpdateDriverLocationRequest {
  resource: DriverLocation;
  fieldMask: FieldMask;
}
```

# おわりに
これまで、良いAPIの本質とその実現に役立つデザインパターンについて紹介してきました。

APIは、利用者にとって実行可能であり、表現力があり、かつシンプルで予測可能なインターフェースを提供することが求められます。 単純な標準メソッドだけでは限界がある場面も多々ありますが、そのような場面では紹介した原則とパターンを駆使し、より快適なAPI生活を送りましょう👋
