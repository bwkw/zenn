---
title: "拡張性は「誰が何を変えるか」で設計する"
emoji: "🧭"
type: "tech"
topics: ["設計", "アーキテクチャ", "拡張性", "TypeScript", "デザインパターン"]
published: false
---

# TL;DR

- 拡張性は、パターン名より先に「誰が何を変えるか」で決めると判断がぶれにくくなります。
- 表の横軸に沿うと、実装差し替えは Strategy / Factory、手順の組み替えは Decorator や Chain of Responsibility、参加者の追加は Registry / Observer / Plugin が候補になります。

# はじめに

こんにちは、Dress Code でプロダクトエンジニアをしている [ないとー](https://x.com/_bwkw_) です！

私たちは [DRESS CODE](https://www.dress-code.com/ja) という、グローバル向けの Workforce Management プロダクトを開発しています。人事・情報システム・総務・採用・コーポレートなど複数部門を横断する業務 OS として動くコンパウンドプロダクトで、複数の機能が同居し相互に参照し合います。

![事業ドメイン](/images/design-extensibility-who-what/business-domain.png)

このようなプロダクトを開発していると、「どこを差し替え可能にするか」「どこまで境界を開くか」という問いに何度も向き合うことになります。機能を増やすたびに、外部連携を広げるたびに、拡張点の判断は積み重なります。グローバル向けであることも重なって、国・地域ごとに労務法制や運用が異なるため、同じ機能でも振る舞いの差や種別の追加が継続的に求められます。

「拡張性」の話になると、まず「どんなやり方があるか」の整理に時間がかかります。実装を差し替えるのか、手順を組み替えるのか、後から参加者を足すのか、切り口からして違います。文献ごとに呼び方も揺れるため、パターン名から入ると判断の軸が定まりにくくなります。

そこでこの記事では、「**誰が拡張するか**」と「**何を変えるか**」の 2 軸で見取り図を置き、その上に代表パターンを載せます。以降、前者を **主体**、後者を **やること** と呼びます。

同じように拡張点の切り方で迷っている方の参考になれば幸いです👋

# 拡張点の見取り図

縦軸が **主体**（誰が拡張するか）、横軸が **やること**（何を変えるか）です。

**主体**は、その拡張の責任を誰が持つかで 3 段階あります。

- **自分**: 拡張点も実装もコアが決める。変更はリポジトリ内で完結する。
- **チーム内**: 拡張点はコアが決め、実装は各チームが後から加える。
- **第三者**: 拡張点を社外に公開する。互換を守る責任が生まれる。

**やること**は、拡張が何を変えるかを 3 つに分けています。

- **実装を差し替える**: interface はそのまま、中身の実装を入れ替える。
- **手順を組み替える**: 処理の流れや、前後に挟む責務を変える。
- **参加者を足す**: 同じ呼び出し口に、後から実装を足す。

| 主体 \\ やること | 実装を差し替える                                                                     | 手順を組み替える                                                              | 参加者を足す                                  |
| ---------------- | ------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------- | --------------------------------------------- |
| 自分             | [Strategy](#strategy) / [Factory Method](#factory-method-%E3%81%A8-abstract-factory) | [Template Method](#template-method)                                           | -                                             |
| チーム内         | -                                                                                    | [Decorator](#decorator) / [Chain of Responsibility](#chain-of-responsibility) | [Registry](#registry) / [Observer](#observer) |
| 第三者           | -                                                                                    | -                                                                             | [Plugin](#plugin-%E3%81%A8-spi)               |

表の `-` は、この記事では代表例を省いた組み合わせです。設計として不可能なわけではなく、第三者が手順や型の境界まで触れる形もありえます。ただしそこまで開くと公開 API と互換維持の負担が一気に重くなるため、実務ではまず **参加者を足す** 列の Plugin で境界を引くほうが現実的です。

迷ったときは、「主体は誰か」と「何を変えるか」の 2 点を先に決めてみてください。その交点のセルが、この記事で参照すべき節と対応しています。

ここからは見取り図の 1 つ 1 つを具体的にみていきます。

# 実装を差し替える

## Strategy

[Strategy](https://en.wikipedia.org/wiki/Strategy_pattern) は、役割の interface を固定し、DI や設定でその中身の実装を差し替えるやり方です。呼び出し元は interface の形だけを知っていれば足りて、どの実装が渡ってくるかは関知しません。

```typescript
// 役割（interface）を先に固定し、実装の詳細は外から差し込む
interface PaymentGateway {
  charge(amount: number, currency: string): Promise<void>;
}

class StripeGateway implements PaymentGateway {
  async charge(amount: number, currency: string): Promise<void> {
    // Stripe API に委譲
  }
}
class PayPalGateway implements PaymentGateway {
  async charge(amount: number, currency: string): Promise<void> {
    // PayPal API に委譲
  }
}

// DI で受け取った実装だけを使う（中身は関知しない）
class CheckoutService {
  constructor(private gateway: PaymentGateway) {}

  async checkout(amount: number, currency: string): Promise<void> {
    await this.gateway.charge(amount, currency);
  }
}
```

実装の種類がそう増えず、interface が長く安定している場面に向きます。差し替えはコンパイル時や DI コンテナ・環境変数などで決めることが多く、「どの実装を使うかの判断を呼び出し元から切り離したい」という動機がはっきりしているときに、パターンの価値が出ます。

ただ、interface は「最大公約数」になりがちです。ある実装だけが持つ固有機能を前面に出したくなると無理が出てきて、ダミー実装や意味の薄い分岐が少しずつ積み重なっていきます。そうなったら一度立ち止まってください。1 つの実装を選んで使うのではなく、種別ごとに処理を増やし続けたいなら、別の形が向きます。

## Factory Method と Abstract Factory

[Factory Method](https://en.wikipedia.org/wiki/Factory_method_pattern) / [Abstract Factory](https://en.wikipedia.org/wiki/Abstract_factory_pattern) は、生成の判断を呼び出し側から分離するパターンです。固定するのは「生成後に使う interface」で、変えるのは「どの具象を返すか」です。Factory Method は単一オブジェクトの生成切り替え、Abstract Factory は関連するオブジェクト群を一貫した組で返すときに使います。

```typescript
// Factory Method: 生成の判断を一か所に閉じる
interface Logger {
  info(msg: string): void;
}
class ConsoleLogger implements Logger {
  info(msg: string): void {
    console.log(`[INFO] ${msg}`);
  }
}
class NoopLogger implements Logger {
  info(): void {} // テスト環境では何もしない
}

// 環境ごとの生成判断はここだけ。呼び出し側は Logger の形だけ知ればよい
function createLogger(env: string): Logger {
  return env === "production" ? new ConsoleLogger() : new NoopLogger();
}

const logger = createLogger(process.env.NODE_ENV ?? "development");
logger.info("起動しました"); // 呼び出し側は実装の種類を知らない

// Abstract Factory: 関連するオブジェクト群を一貫した組で返す
// 例: DB の種類によって Connection と QueryBuilder を一貫した組で差し替える
//   （Postgres と MySQL ではプレースホルダ記法が違うため、組を間違えると壊れる）
interface Connection {
  execute(sql: string, params: unknown[]): Promise<unknown[]>;
}
interface QueryBuilder {
  selectById(table: string, id: number): { sql: string; params: unknown[] };
}

interface DbDriver {
  createConnection(): Connection;
  createQueryBuilder(): QueryBuilder;
}

// Postgres 向け：プレースホルダは $1, $2, ...
class PostgresDriver implements DbDriver {
  createConnection(): Connection {
    return new PostgresConnection();
  }
  createQueryBuilder(): QueryBuilder {
    return new PostgresQueryBuilder();
  }
}

// MySQL 向け：プレースホルダは ?, ?, ...
class MysqlDriver implements DbDriver {
  createConnection(): Connection {
    return new MysqlConnection();
  }
  createQueryBuilder(): QueryBuilder {
    return new MysqlQueryBuilder();
  }
}
```

生成の判断を一か所に閉じることで、呼び出し側は「何が返ってくるか」だけに集中できます。テストでフェイクへ差し替えたい場面や、環境・設定によって生成物を変えたい場面が典型で、「生成の詳細を意識させたくない境界」に挟むほど効きます。

ただ、Factory が機能ごとに散在すると、どこで何が生成されているか全体が見えにくくなります。生成条件が肥大化してきたら、アプリ起動時に組み立てを一か所へ集約するか、DI コンテナに任せる判断が必要です。

# 手順を組み替える

## Template Method

[Template Method](https://en.wikipedia.org/wiki/Template_method_pattern) は、処理の骨格（手順）を固定し、特定のステップだけサブクラスに委ねる形です。Strategy が「どのアルゴリズムを使うか」を外から差し込むのに対し、こちらは「この順序で実行する」という流れ自体をコードに固め、穴になる場所だけを開けておきます。

```typescript
abstract class ReportJob {
  // 骨格（処理順）は親クラスで固定する
  async run(): Promise<void> {
    const raw = await this.fetch();
    const out = this.transform(raw);
    await this.save(out);
  }

  // ここだけサブクラスで差し替える
  protected abstract fetch(): Promise<unknown>;

  // 必要なければそのまま使える既定ステップ
  protected transform(data: unknown): unknown {
    return data;
  }

  // ここもサブクラスで差し替える
  protected abstract save(out: unknown): Promise<void>;
}

class DailySalesReport extends ReportJob {
  protected async fetch(): Promise<unknown> {
    // DB から日次売上を取得
    return [];
  }
  protected async save(out: unknown): Promise<void> {
    // 集計結果を S3 に保存
  }
}
```

処理の流れを共通のまま保ちながら、一部のステップだけ種別ごとに変えたいときに向きます。レポートやバッチ処理が典型で、「fetch → transform → save という手順は固定でも、データの取得先や保存先は種別によって違う」という状況です。骨格がコードに固まっていることに価値があるので、「この後に必ずこの処理が走る」という保証がロジックの正しさに関わるほど、パターンが力を発揮します。

Strategy と悩む場面では、「アルゴリズム全体を外から差し込みたいのか、処理の順序を固定して穴だけ埋めたいのか」が分岐点になります。実行時に実装を切り替えたいなら Strategy が向き、Template Method は「この順序で実行する」骨格自体に意味があるので、その順序を変えたくなった時点でパターンの意味が薄れます。

差し替えが一ステップだけなら、サブクラスを立てるほど重くはなく、コールバックや Strategy のほうがシンプルに書けることが多いです。バリアントが増えて継承階層が膨らんできたら、合成への切り替えを検討するとよいです。

## Decorator

[Decorator](https://en.wikipedia.org/wiki/Decorator_pattern) は、本体の処理や interface を固定したまま、外側から責務を足すやり方です。ラッパーを順に重ねることで、本体に触れずに横断的な処理を差し込めます。

```typescript
import { readFile } from "node:fs/promises";

interface DataSource {
  read(key: string): Promise<string>;
}

// 本体：データを取り出すことだけに集中する
class FileDataSource implements DataSource {
  async read(key: string): Promise<string> {
    return readFile(key, "utf-8");
  }
}

// Decorator 1：結果をキャッシュする（同じ interface を保ったまま外側から包む）
class CachingDataSource implements DataSource {
  private cache = new Map<string, string>();

  constructor(private inner: DataSource) {}

  async read(key: string): Promise<string> {
    const hit = this.cache.get(key);
    if (hit !== undefined) return hit;
    const value = await this.inner.read(key); // 必ず inner を呼ぶ
    this.cache.set(key, value);
    return value;
  }
}

// Decorator 2：呼び出しをログに残す（同じ interface を保ったまま外側から包む）
class LoggingDataSource implements DataSource {
  constructor(private inner: DataSource) {}

  async read(key: string): Promise<string> {
    console.log(`read: ${key}`);
    return this.inner.read(key); // 必ず inner を呼ぶ
  }
}

// 組み合わせ：呼び出し元は DataSource としてしか扱わない
const ds: DataSource = new LoggingDataSource(
  new CachingDataSource(new FileDataSource())
);
```

ログ・キャッシュ・計測のように、本体のコードに触れずに横断的な処理を足したいときに向きます。同じ interface を保ったまま外側から包めるので、呼び出し元は何層に包まれているかを意識せずに済みます。

Chain of Responsibility と混同されやすいですが、違いは「止めるかどうか」です。Decorator は外側のラッパーが必ず内側を呼びます。一方、Chain of Responsibility は途中のハンドラが処理を打ち切れます。「条件によってここで止める」という分岐が生まれた時点で、それはもう Decorator ではなく Chain of Responsibility の仕事になっています。

ラップが深くなるほどスタックトレースは読みにくくなります。クラスの粒度・ログの粒度・ラップ順序のルールはチームで最初に決めておくと、あとから増える局面でも一貫して運用できます。

## Chain of Responsibility

[Chain of Responsibility](https://en.wikipedia.org/wiki/Chain-of-responsibility_pattern) は、複数のハンドラを鎖状につなぎ、順に渡して「誰かが引き取る or 全員が処理する」形です。途中で止める必要がなければ `reduce` で順に適用する関数合成で十分なので、「条件によって処理を打ち切る」という制御が必要になったときに初めて出番になります。

```typescript
type Next = () => Promise<Response>;
type Middleware = (req: Request, next: Next) => Promise<Response>;

// 認証：トークンがなければここで止める（これが Chain of Responsibility の核心）
const auth: Middleware = async (req, next) => {
  if (!req.headers.get("authorization"))
    return new Response("Unauthorized", { status: 401 }); // 止まる
  return next(); // 通過したら次へ渡す
};

// ロギング：前後に処理を挟みつつ、必ず next を呼ぶ
const logging: Middleware = async (req, next) => {
  console.log(`--> ${req.method} ${req.url}`);
  const res = await next();
  console.log(`<-- ${res.status}`);
  return res;
};

// 本体
const handler = async (_req: Request) => new Response("OK");

// チェーンを組み立てる：順番が振る舞いを決める
function buildChain(
  middlewares: Middleware[],
  final: (req: Request) => Promise<Response>
): (req: Request) => Promise<Response> {
  return middlewares.reduceRight(
    (next, mw) => (req) => mw(req, () => next(req)),
    final
  );
}

// logging → auth → handler の順で処理される
const app = buildChain([logging, auth], handler);
```

認証 → 認可 → レート制限 → 本体のように、順序付きで処理の段を増やしたい場面に向きます。ハンドラをリストで持ちさえすれば段を追加でき、各ハンドラが「自分の判断だけ」に集中できるため、処理が増えても全体の見通しが保たれます。

ただ、ハンドラが 1 つだけなら直接呼ぶほうがシンプルで、このパターンを持ち出す必要はありません。ハンドラ同士が状態を共有していたり、チェーンが極端に長くなっているようなら、責務の分け方から見直すほうがよいことが多いです。

# 参加者を足す

この 3 パターンは「同じ呼び出し口に後から参加者を増やす」という点では共通です。ただし、1 種別に対して何人が反応するか、そして参加できるのは誰かで性格が変わります。Registry は 1:1 の対応、Observer は 1:N のファンアウト、Plugin は組織の外まで開く、という段階的な拡張です。

## Registry

[Registry](https://martinfowler.com/eaaCatalog/registry.html) は、キーと実装の対応を登録し、あとからキーで引いて呼び出す仕組みです。Strategy が「同じ interface の差し替え」だとすれば、Registry は「種別を増やしても dispatch 側のコードを触らない」ための形です。

```typescript
// 拡張子をユニオン型で宣言（typo やリネーム漏れをコンパイルで検出できる）
type Extension = "json" | "csv" | "yaml";

interface Parser {
  parse(content: string): unknown;
}

const parsers = new Map<Extension, Parser>();

// 新しい種別を追加するときはここだけ
export function register(extension: Extension, parser: Parser): void {
  parsers.set(extension, parser);
}

// parse 側は種別が増えても変わらない
export function parse(extension: Extension, content: string): unknown {
  const parser = parsers.get(extension);
  if (!parser) throw new Error(`parser not found: ${extension}`);
  return parser.parse(content);
}

// 種別を増やしても parse のコードには触れない
register("json", { parse: (c) => JSON.parse(c) });
register("csv", { parse: (c) => c.split("\n").map((row) => row.split(",")) });
register("yaml", { parse: (c) => parseYaml(c) });

const data = parse("json", '{"name":"Alice"}');
```

種別が後から増えても、`parse` 側の `if` や `switch` を触らずに済みます。新しい処理を追加するときは `register` を呼ぶだけで、呼び分けのロジックには手を入れなくてよいため、種別の追加コストが局所化されます。

ただ、キーに素の文字列を使うとリネーム漏れが実行時まで気づけず、障害になったときに原因を追いにくいです。種別をユニオン型で宣言し、`register` と `parse` の引数にその型を載せておくと、コンパイル時に誤りを検出できます。同じ種別に複数の反応が要るようになったら Observer、社外の開発者にも参加を開くなら Plugin が次の検討先です。

## Observer

[Observer](https://en.wikipedia.org/wiki/Observer_pattern) は、ひとつのイベント（状態変化）に対して複数の独立した購読者を後から足す仕組みです。Registry が「1 キー＝1 ハンドラ」なのに対し、Observer は「1 イベント＝ N 個のリスナ」です。

```typescript
type Listener<T> = (payload: T) => void;

class EventBus<Events extends Record<string, unknown>> {
  private listeners = new Map<keyof Events & string, Listener<any>[]>();

  // 購読者はいつでも後から追加できる
  on<K extends keyof Events & string>(event: K, fn: Listener<Events[K]>): void {
    const list = this.listeners.get(event) ?? [];
    this.listeners.set(event, [...list, fn]);
  }

  // 発火すると登録された全購読者が呼ばれる（止められない）
  emit<K extends keyof Events & string>(event: K, payload: Events[K]): void {
    for (const fn of this.listeners.get(event) ?? []) fn(payload);
  }
}

type AppEvents = {
  "employee.created": { employeeId: string; name: string };
};

const bus = new EventBus<AppEvents>();

// 購読者を増やしても emit 側のコードは変わらない（1 イベントに N 人が反応する）
bus.on("employee.created", ({ employeeId }) => {
  /* ウェルカムメール送信 */
});
bus.on("employee.created", ({ employeeId }) => {
  /* 初期権限を付与 */
});
bus.on("employee.created", ({ name }) => {
  /* 監査ログに記録 */
});

// 発行側：何人が聞いているかを知らずに発火する
bus.emit("employee.created", { employeeId: "emp-001", name: "山田 太郎" });
```

ひとつの変化に対して複数の独立した反応（メール通知・監査ログ・キャッシュ破棄など）を後から足したいときに向きます。従業員が作成されたとき、どのチームが何をするかを emit 側は知らなくてよく、購読者が増えても発火側のコードは変わりません。Registry が「種別 → 実装」を 1:1 で引くのに対し、Observer は 1 イベントに対して N 個の独立した反応が成立する、というのが両者の違いです。

ただ、通知順に依存する実装が紛れ込むと、購読者が増えた段階で予期しない挙動が出ることがあります。どの順で呼ばれても結果が同じになる設計を前提にしておくのが安全で、順序の制御が必要な処理は Chain of Responsibility で明示的に扱うほうが向きます。社外の開発者にも参加を開きたいなら、Plugin が次の検討先です。

## Plugin と SPI

[Plugin](<https://en.wikipedia.org/wiki/Plug-in_(computing)>) は、第三者（社外の開発者など）が拡張を独立して配布し、ホストが動的に取り込む仕組みです。Registry に動的読み込みと公開 API を足した形で実現することが多いです。Java の SPI（Service Provider Interface）の [`ServiceLoader`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ServiceLoader.html) で説明される読み込み方も、同じ発想に収まります。

```typescript
// ホストが提供する API（Plugin から見える世界）
interface HostApi {
  registerCommand(name: string, run: () => void): void;
  onEvent(event: string, fn: () => void): void;
}

// ホストが読み込むモジュールの形
interface PluginModule {
  name: string;
  setup(host: HostApi): void;
}

// ホスト側：動的に読み込んで初期化する
async function loadPlugins(pluginNames: string[], host: HostApi): Promise<void> {
  for (const name of pluginNames) {
    const mod = await import(`./plugins/${name}.js`);
    const plugin: PluginModule = mod.default;
    plugin.setup(host); // モジュールが自分で必要な登録をする
  }
}
```

ESLint・Prettier・VS Code のように、第三者が拡張を持ち寄るプロダクトに向きます。社内では気軽に壊せる API も、外部に公開した途端に互換を守り続ける義務が生まれ、ドキュメントの維持や破壊的変更への対処が継続的なコストになります。そのコストに見合わない社内ツールでは、Observer や Registry で十分なことがほとんどです。

Registry や Observer が組織内での参加者追加だとすれば、Plugin は境界の外へ拡張を開く、性質の違う選択です。社外向けの公開 API になると、ドキュメントに書いた契約だけでなく、実際に観測できる挙動まで依存されやすくなります（[Hyrum's Law](https://www.hyrumslaw.com/)）。設計パターンというより、公開プラットフォームを運営する判断に近いです。

# おわりに

拡張点の設計は、パターン名より先に **主体** と **やること** を決めたほうが、あとで見直すときの判断が安定しやすいと感じています。文献の呼び方は揺れますが、軸さえ共有できればレビューで同じ地図を見られます。

一度足した境界を畳むのは簡単ではありません。想定外の要件が来たときは、コードを書く前に **主体** と **やること** をメモに書き出すだけでも、判断は変わることがあります。

機能が絡み合うコンパウンドに、国・地域ごとの差異まで重なると、拡張要求は複数方向から長く続きます。そういったプロダクトほど、判断の軸をチームで共有できたときの効きが大きいと感じています。

銀の弾丸ではありませんが、名前に振り回されたときに「誰が何を変えるのか」へ戻れる足場になればうれしいです👋
