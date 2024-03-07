---
title: "君たちの知らないAPIの話をしよう"
emoji: "👺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["API", "APIデザインパターン"]
published: false
publication_name: "levtech"
---

# TL;DR


# はじめに
現代のソフトウェア開発において、API設計は非常に重要な要素です。API設計は、アプリケーションの機能性、保守性に深く関わっています。

「**APIデザインパターン**」は、この重要かつ複雑な分野において、非常に価値のある一冊です。

https://techplay.jp/book/373

APIの設計は、表面上は単純なように思えるかもしれませんが、その背後には実行可能でありながら表現力があり、シンプルで予測可能なインターフェースを提供することが求められます。しかし、実際にこれらの原則を適用しようとした場合、単純な標準メソッドだけでは限界があります。

そこで「APIデザインパターン」の価値が際立ちます。この本は、API設計の課題を克服するための具体的な手法とデザインパターンを詳述しています。カスタムメソッド、ロングランオペレーション、再実行可能ジョブなど、多様なデザインパターンが紹介されており、これらはAPIの基本原則に忠実でありながら、より複雑な要件や特有の問題に対処するために考案されています。

# そもそも良いAPIとは？
![良いAPIとは？](/images/talk_about_apis_you_dont_know_about/good_api.png)

## Executable -実行可能であること-
APIの設計において最も基本的かつ重要な要素は「**実行可能性**」です。これは、システムがユーザーが行いたいことを実際に遂行できる能力を持つことを意味します。

実行可能なシステムは、ユーザーの要求を満たすだけでなく、迅速かつ正確な結果を提供することで、信頼性とユーザーエクスペリエンスを高めます。

## Expressive -表現力があること-
良いAPIは、実行可能性に加えて高い「**表現力**」を持つ必要があります。これは、ユーザーが望む機能をシンプルかつ直感的に実現できることを意味します。APIが提供する機能は、ユーザーが容易に理解し、使用できるように設計されていなければなりません。

API設計者は、ユーザーがどのようにAPIを使用するかを深く理解し、ユーザーの要求を先読みして適切な機能を提供する必要があります。表現力の高いAPIは、ユーザーが直感的に理解しやすく、より満足度の高い体験を提供します。

## Simple -シンプルであること-
良いAPI設計において重要なのは、その「**シンプルさ**」です。シンプルさとは、APIに含まれる要素（RPCやリソースなど）の数を単純に減らすことではなく、ユーザーが必要とする機能を最も直接的かつ簡単な方法で利用できるようにすることです。

シンプルさを追求する際には、一般的な機能は簡単に使えるようにしつつ、高度な機能も提供できるように設計することが大切です。つまり、APIが高度な機能を提供する場合でも、ユーザーにはその複雑さが隠蔽されているべきです。

## Predictable -予測可能であること-
良いAPIは、ユーザーに予期せぬ驚きを与えることなく、「**予測可能**」でなければなりません。驚かせないAPIとは、定義や基本動作が一貫しており、ユーザーが直感的に理解できるものです。この原則は、APIの設計全体において、特に名前付けや動作パターンにおいて重要です。

一貫したフィールド名や標準メソッドなど、繰り返し出てくるパターンを持つAPIは、利用者にとって習得が容易で、効率的です。 予測可能なAPIは、利用者が自信を持って使用できるようにし、生産性を高めるために不可欠です。利用者が何を期待するかを理解し、それに応えることで、APIはより使いやすく、効果的なものになります。

# 馴染みのあるAPI設計
以下の記事がとてもよくまとまっているのでこちらを参照してください。（丸投げ）
https://qiita.com/KNR109/items/d3b6aa8803c62238d990
https://qiita.com/baby-degu/items/6f516189445d98ddbb7d

# デザインパターンの紹介
ここからが本題です。本セクションでは、API設計パターンについて、TypeScriptの実際のソースコードを用いて説明します。APIメソッドの入力と出力についての考察を行い、リクエストとレスポンスのインターフェースには、規約として `Request` や `Response` という接尾辞を付けます。さらに、TypeScriptのデコレータを使ってRESTfulなAPIメソッドの実装例を紹介します。

## カスタムメソッド
### 概要
カスタムメソッドは、APIで標準メソッドの範囲を超える特定の操作を行うために使用されます。これらは、標準メソッドの厳格なルールに従う必要がなく、柔軟に対応できるため、特定のケースに適したアプローチを取ることが可能です。ただし、API利用者がカスタムメソッドに対して標準メソッドと同じような予測をしにくいため、API全体の一貫性を保つことが重要です。

https://cloud.google.com/apis/design/custom_methods?hl=ja

### 対象とする問題
多くのAPIで、標準メソッドでは対応できない特定の操作が必要になる場合があります。メールの送信や即時の文書翻訳など、通常のcreateやupdateメソッドでは処理しづらい操作がこれに該当します。カスタムメソッドは、これら非標準的な操作を実現し、APIの便利性を向上させます。

### 問題点
カスタムメソッドは適切なリソース構成があれば標準メソッドでも実現できる操作を提供しますが、リソース設計を不適切にするか、過度に使用することでAPIの設計が劣化するリスクがあります。

### 実装例
以下は、メールサービスAPIでカスタムメソッドを実装する例です。`EmailApi` 抽象クラスを介して、特定のユーザーにメールを送信する方法を示しています。

```typescript
abstract class EmailApi {
  @post("/{id=users/*emails/*}:send")
  abstract SendEmail(req: SendEmailRequest): Email;
}

interface Email {
  id: string;
  subject: string;
  content: string;
  state: string;
  deleted: boolean;
  // ...
}
```

## ロングランオペレーション（Long Running Operations, LRO）
### 概要
LROは、完了までに時間がかかる処理を指し、現代のプログラミング言語では非同期動作を通じてこれをサポートしています。これにより、時間を要する処理を行いつつ、他の作業を同時に進めることができます。非同期処理では、作業の開始時にプレースホルダーを返し、作業完了時には結果と共にプレースホルダーを「解決」するか、エラー発生時に「拒否」します。これにより、結果を非同期に処理したり、結果が返されるまで待機したりすることが可能です。

https://cloud.google.com/translate/docs/advanced/long-running-operation?hl=ja

### 対象とする問題
APIが複雑な作業や大量のデータを扱う場合、簡単なAPI設計では迅速な操作と時間を要する操作の間で一貫性を保つことが困難です。そのため、時間がかかる作業は特別な取り扱いが必要になります。

### 問題点
LROは、初学者にとっては理解が難しい場合があります。これは、リクエストによってのみ生成され、従来のリソース作成方法とは異なるアプローチを採用しているためです。LROは、明示的な作成プロセスを伴わず、その存在は他の作業を要求することに依存します。

### 実装例
以下は、LROを扱うAPIの実装例です。この例では、 `ChatRoomApi` 抽象クラスを利用して、進行中の操作の状態を取得する方法と、必要に応じて操作をキャンセルする方法を紹介します。LROを管理するためには、`Operation` リソースの定義が不可欠です。このプロセスには、`Operation` オブジェクトの生成、進行状態の追跡、そして操作の一時停止、再開、キャンセルを可能にするメソッドが含まれます。示されているコードスニペットは、これらの機能の一部を実装する方法を示しています。

```typescript
abstract class ChatRoomApi {
  @get("/{id=operations/*}")
  abstract GetOperation<ResultT, MetadataT>(req: GetOperationRequest):
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

interface CancelOperationRequest {
  id: string;
}
```

## 再実行可能ジョブ
### 概要
再実行可能ジョブは、APIのオンデマンド実行機能を初期設定と実行の二つの主要なステップに分ける特別なリソースです。従来のAPIメソッドでは、メソッド呼び出し時に必要な設定を提供し、即座に処理が行われます。一方、再実行可能ジョブでは、処理を実行するための設定を使ってジョブをまず作成し、次に `run` というカスタムメソッドで実際の処理を行います。

https://cloud.google.com/dataproc/docs/concepts/jobs/restartable-jobs?hl=ja

### 対象とする問題
クライアントがAPIメソッドを非同期に呼び出す必要があるという前提のもと、前述のLROでは解決しない場合があります。
以下のような事例はその最たる例です。

#### 非同期実行を行うカスタムメソッドの提供
設定管理が複雑になるにつれ、API自体に設定パラメータを保存する機能が有効になります。

#### オンデマンドメソッドの実行
設定と実行の権限を分ける必要性が生じ、特に開発者と運用チームの間で権限を区別する場合に有用です。

#### 定期的な自動実行の要望
APIが自身でスケジュールを設定し、外部システムなしでメソッドを実行できるようにすることで、シンプルかつ安全な操作を可能にします。

### 問題点
適切なフィルタリングがないと、 `Operation` リソースのリストから目的の `Job` リソースを特定するのが難しくなります。

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
API設計時、リソースの構成要素がプロパティに適しているにもかかわらず、通常のプロパティとして扱えない場合があります。
以下のような事例はその最たる例です。

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
