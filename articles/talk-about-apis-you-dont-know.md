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

先日、API設計について体系的に学べる本がないかな〜って探していたところ、先輩エンジニアに「APIデザインパターン」を紹介いただきました。

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
ここからが本題です。大抵の場合、上の例で示したようなAPI設計で十分です。ただ、いくつかの複雑な要件では、上の例で示したようなAPI設計では[良いAPIを設計するための4つの特性](https://zenn.dev/levtech/articles/talk-about-apis-you-dont-know#%E3%81%9D%E3%82%82%E3%81%9D%E3%82%82%E8%89%AF%E3%81%84api%E3%81%A8%E3%81%AF%EF%BC%9F)を満たすことが出来ず、デザインパターンが必要になることがあります。

## カスタムメソッド
### 概要
カスタムメソッドは、標準的なCRUD操作（作成、読み取り、更新、削除）では対応できない特定の操作が必要になる場合に便利です。例えば、メールの送信や即時の文書翻訳など、通常の `create` や `update` メソッドでは処理が難しい操作がこれに該当します。

参考までに、以下にGoogleが出しているカスタムメソッドの記事を示します。
https://cloud.google.com/apis/design/custom_methods?hl=ja

### 実装例
以下は、カスタムメソッドを扱うAPIの実装例です。`EmailApi` 抽象クラスを介して、特定のユーザーにメールを送信する方法を示しています。 この例の注目点は、エンドポイントの定義における `:send` の使用です。ここでの `:send` は、カスタムメソッドの命名規則を示しており、標準的なHTTPメソッド（GET、POSTなど）に対する補足的な操作を表します。

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

## ロングランオペレーション (Long Running Operations, LRO)
### 概要
LROは、完了までに時間がかかる処理を指し、これを非同期動作を通じてサポートしています。特に、APIが複雑な作業や大量のデータを扱う場合、通常のAPI設計では迅速な操作と時間を要する操作の間で一貫性を保つことが難しくなります。このような時間がかかる処理を行う場合、非同期処理を利用して、作業の開始時にプレースホルダーを返し、作業完了時には結果と共にプレースホルダーを「解決」するか、エラー発生時に「拒否」します。

参考までに、以下にGoogleが出しているLROの記事を示します。
https://cloud.google.com/translate/docs/advanced/long-running-operation?hl=ja

### 実装例
以下は、LROを扱うAPIの実装例です。この例では、 `ChatRoomApi` 抽象クラスを利用して、進行中の操作の状態を取得する方法と、必要に応じて操作をキャンセルする方法を紹介します。この実装では、 `Operation` リソースを用いて操作の管理と状態の追跡を効率的に行います。

※ デザインパターンのコードを引用しています。
> ```typescript
> abstract class ChatRoomApi {
>   @get("/{id=operations/*}")
>   abstract GetOperation<ResultT, MetadataT>(req: GetOperationRequest):
>     Operation<ResultT, MetadataT>;
> 
>   @get("/{id=operations/*}:wait")
>   abstract WaitOperation<ResultT, MetadataT>(req: WaitOperationRequest):
>     Operation<ResultT, MetadataT>;
> 
>   @post("/{id=operations/*}:cancel")
>   abstract CancelOperation<ResultT, MetadataT>(req: CancelOperationRequest):
>     Operation<ResultT, MetadataT>;
> }
> 
> interface Operation<ResultT, MetadataT> {
>   id: string;
>   done: boolean;
>   expireTime: Date;
>   result?: ResultT | OperationError;
>   metadata?: MetadataT;
> }
> 
> interface OperationError {
>   code: string;
>   message: string;
>   details?: any;
> }
> 
> interface GetOperationRequest {
>   id: string;
> }
> 
> interface WaitOperationRequest {
>   id: string;
> }
> 
> interface CancelOperationRequest {
>   id: string;
> }
> ```

## 再実行可能ジョブ
### 概要
再実行可能ジョブは、APIの使用においてオンデマンドメソッドの実行や定期的な自動実行のニーズに対応します。設定と実行の権限を分けることで、開発者と運用チーム間の権限分離が可能になり、外部システムを必要とせずにAPI自身でスケジュール設定しメソッドを実行できるようになります。

参考までに、以下にGoogleが出している再実行可能ジョブの記事を示します。
https://cloud.google.com/dataproc/docs/concepts/jobs/restartable-jobs?hl=ja

### 実装例
以下は、再実行可能ジョブのAPI実装例です。この例では、メッセージ分析を目的としたジョブの設定(`CreateAnalyzeChatRoomJob`)と、非同期にそのジョブを実行するためのカスタムrunメソッド(`RunAnalyzeChatRoomJob`)を利用しています。

※ デザインパターンのコードを引用しています。
> ```typescript
> abstract class ChatRoomApi {
>   @post("/analyzeChatRoomJobs")
>   abstract CreateAnalyzeChatRoomJob(req: CreateAnalyzeChatRoomJobRequest):
>     AnalyzeChatRoomJob;
> 
>   @post("/{id=analyzeChatRoomJobs/*}:run")
>   abstract RunAnalyzeChatRoomJob(req: RunAnalyzeChatRoomJobRequest):
>     Operation<AnalyzeChatRoomJobExecution, RunAnalyzeChatRoomJobMetadata>;
> }
> 
> interface CreateAnalyzeChatRoomJobRequest {
>   resource: AnalyzeChatRoomJob;
> }
> 
> interface AnalyzeChatRoomJob {
>   id: string;
>   chatRoom: string;
>   destination: string;
>   compressionFormat: string;
> }
> 
> interface RunAnalyzeChatRoomJobRequest {
>   id: string;
> }
> 
> interface AnalyzeChatRoomJobExecution {
>   id: string;
>   job: AnalyzeChatRoomJob;
>   sentenceComplexity: number;
>   sentiment: number;
>   abuseScore: number;
> }
> 
> interface RunAnalyzeChatRoomJobMetadata {
>   messagesProcessed: number;
>   messagesCounted: number;
> }
> ```

## シングルトンサブリソース
### 概要
API設計において、リソースの特定の構成要素がそのサイズや複雑さ、セキュリティ要件、または頻繁な更新の必要性により、通常のプロパティとして適切に扱うことが難しい場合があります。シングルトンサブリソースを利用することで、これらの構成要素を独立したエンティティとして設計し、それぞれに適した管理方法を適用することが可能になります。

参考までに、以下にGoogleが出しているシングルトンサブリソースの記事を示します。
https://cloud.google.com/apis/design/design_patterns?hl=ja#singleton_resources

### 実装例
以下は、シングルトンサブリソースを扱うAPIの実装例です。`RideSharingApi` 抽象クラスは、ドライバーの基本情報の取得・更新メソッドと、位置情報の取得・更新メソッドを別々に提供します。これにより、ドライバーの位置情報などの頻繁に変更されるデータを効率的に管理し、更新することが可能になります。

※ デザインパターンのコードを引用しています。
> ```typescript
> abstract class RideSharingApi {
>   @get("/{id=drivers/*}")
>   abstract GetDriver(req: GetDriverRequest): Driver;
> 
>   @patch("/{resource.id=drivers/*}")
>   abstract UpdateDriver(req: UpdateDriverRequest): Driver;
> 
>   @get("/{id=drivers/*/location}")
>   abstract GetDriverLocation(req: GetDriverLocationRequest): DriverLocation;
> 
>   @patch("/{resource.id=drivers/*/location}")
>   abstract UpdateDriverLocation(req: UpdateDriverLocationRequest): DriverLocation;
> }
> 
> interface Driver {
>   id: string;
>   name: string;
>   licensePlate: string;
> }
> 
> interface DriverLocation {
>   id: string;
>   lat: number;
>   long: number;
>   updateTime: Date;
> }
> 
> interface GetDriverLocationRequest {
>   id: string;
> }
> 
> interface UpdateDriverLocationRequest {
>   resource: DriverLocation;
>   fieldMask: FieldMask;
> }
> ```

# おわりに
これまで、良いAPIの本質とその実現に役立つデザインパターンについて紹介してきました。

私自身、デザインパターンを知らないあまり、強引にAPIを設計していたことも多々あったな...と考えさせられましたし、API設計を考える上で一つのひっかかりが出来たという意味で、読んで価値ある本だったなと思いました。

まだまだ紹介出来てないデザインパターンが多くあるので、興味が湧いた方は是非手に取ってみてください👋
