---
title: "拡張性は「誰が何を変えるか」で設計する"
emoji: "🧭"
type: "tech"
topics: ["設計", "アーキテクチャ", "拡張性", "TypeScript", "デザインパターン"]
published: false
---

# TL;DR

- 拡張性は、パターン名より先に「誰が何を変えるか」で決めると判断がぶれにくくなります。
- 実装差し替えなら Strategy / Factory、参加者を足すなら Registry / Observer / Plugin、流れを組み替えるなら Decorator や Chain of Responsibility が候補になります。
- 実行時に種類が増えない「閉じた世界」では、まず discriminated union で型に閉じるのが安全です。

# はじめに

こんにちは、Dress Code でプロダクトエンジニアをしている [ないとー](https://x.com/_bwkw_) です！

私たちは [DRESS CODE](https://www.dress-code.com/ja) という、グローバル向けの Workforce Management プロダクトを開発しています。

プロダクトを育てていると、機能追加や外部連携、運用の変化に合わせて、どこを差し替え可能にするか、どこまで境界を開くか、という判断を何度も迫られます。拡張点を増やすほど保守や後方互換の負担も積み上がり、一度入れた境界を後から畳むのは簡単ではありません。

設計の文脈で「拡張性」と聞くと、まず Strategy を思い浮かべる人も多いと思いますが、選択肢はそれだけではありません。Registry / パイプライン / イベント / Plugin / 型で閉じる方法まで、幅は広いです。名前が文献ごとに揺れるので、パターン名から入ると迷子になりやすい。

そこでこの記事では、**「誰が拡張するか」と「何を変えるか」** の 2 軸で見取り図を置き、代表パターンをその上に載せていきます。以下、この 2 つを **主体** と **やること** と呼びます。タイトルどおり、**拡張性は「誰が何を変えるか」で設計する**、という整理です。

同じように拡張点の切り方で迷っている方の参考になれば幸いです👋

# 拡張点の見取り図

縦軸が **主体**（誰が拡張するか）、横軸が **やること**（どう変えるか）です。セルに載っているのは代表例で、本文の見出しへ飛べます。

| 主体 \\ やること   | 実装を差し替える                                                                     | 手順を組み替える                                                              | 参加者を足す                                  | 型やルールで宣言する                                                                 |
| ------------------ | ------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------- | --------------------------------------------- | ------------------------------------------------------------------------------------ |
| 自分（コア開発者） | [Strategy](#strategy) / [Factory Method](#factory-method-%E3%81%A8-abstract-factory) | [Template Method](#template-method)                                           | -                                             | [discriminated union](#discriminated-union-%E3%81%A8-exhaustiveness-checking)        |
| チーム内           | -                                                                                    | [Decorator](#decorator) / [Chain of Responsibility](#chain-of-responsibility) | [Registry](#registry) / [Observer](#observer) | [DSL](#dsl-%E3%81%A8%E3%83%AB%E3%83%BC%E3%83%AB%E3%82%A8%E3%83%B3%E3%82%B8%E3%83%B3) |
| 第三者             | -                                                                                    | -                                                                             | [Plugin](#plugin-%E3%81%A8-spi)               | -                                                                                    |

空白セルは「技術的には可能でも、実務ではコストが見合いにくい」組み合わせです。たとえば第三者に手順や宣言を開く設計は可能でも、互換性維持のコストが大きく、実務では Plugin で参加者を足す形に寄せられます。

セル例:

- Strategy / Factory：決済プロバイダや Logger の切り替え。
- Template Method：レポート生成や ETL ジョブの骨格を固定し、ステップだけ差し替える。
- Decorator / Chain of Responsibility：認証・ログ・レート制限などのミドルウェア。
- Registry / Observer：イベント種別ごとの handler 登録、購読者通知。
- Plugin：ESLint・Prettier・VS Code のような第三者拡張。
- discriminated union / DSL：状態遷移、決済イベント種別、業務ルール表。

迷ったら、次の 3 つを順に言語化すると、表のどのセルに乗るかがほぼ決まります。

1. 主体（誰が拡張するか）：自分／チーム内／第三者。
2. やること（どう変えるか）：実装を差し替える／手順を組み替える／参加者を足す／型やルールで宣言する。
3. 一度開いた境界を、あとから畳めるか：互換性・移行コストの見積り。

この見方は新しいものではなく、ホスト側が拡張点を宣言し、拡張側がそこへ差し込むという語彙は Eclipse の extension point / extension に近いです（[Eclipse plugin architecture](https://www.eclipse.org/articles/Article-Plug-in-architecture/plugin_architecture.html)）。

# 実装を差し替える：Strategy / Factory

interface（呼び出し側との取り決め）を固定して、その実装を差し替える章です。主に自分（コア開発者）が静的に手を打つ場面を想定します。

## Strategy

[Strategy](https://en.wikipedia.org/wiki/Strategy_pattern) は、役割の interface を固定し、その中身の実装を差し替えるやり方です。DI（Dependency Injection、依存の差し込み）や設定で実装を切り替えます。

```typescript
// 役割（決済）だけを先に固定し、実装は後から差し替える
interface PaymentGateway {
  charge(amount: number, currency: string): Promise<ChargeResult>;
}

class StripeGateway implements PaymentGateway {
  async charge(amount: number, currency: string) {
    // Stripe API に委譲
  }
}
class PaypalGateway implements PaymentGateway {
  async charge(amount: number, currency: string) {
    // PayPal API に委譲
  }
}

class CheckoutService {
  // DI で渡された実装だけを使う（中身は知らない）
  constructor(private gateway: PaymentGateway) {}

  async checkout(amount: number, currency: string) {
    await this.gateway.charge(amount, currency);
  }
}
```

実装の種類がそう増えず、interface が長く安定している場面で向きます。差し替えはコンパイル時や DI コンテナ、環境変数などで決めることが多いです。

一方、interface は「最大公約数」になりがちで、ある実装だけの固有機能を前面に出したくなると無理が出ます。ダミー実装や意味の薄い分岐が増えてきたら、一度立ち止まってください。Registry で種別を増やすのか、discriminated union やルールで宣言側に逃がすのか、別のセルに移ったほうがすっきりすることがあります。

Strategy と Registry の境目で迷ったら、「実装を 1 つだけ選んで使う」のか「種別ごとにエントリを増やす」のかで分けると素直です。決済プロバイダのように 1 ユースケースに 1 実装なら Strategy、イベント種別ごとに違う handler を増やし続けるなら Registry が向きます。種別が増え続けるなら Registry、実行時に種類が増えないなら discriminated union が次の検討先です。

## Factory Method と Abstract Factory

[Factory Method](https://en.wikipedia.org/wiki/Factory_method_pattern) / [Abstract Factory](https://en.wikipedia.org/wiki/Abstract_factory_pattern) は、生成の判断を呼び出し側から分離するパターンです。固定点は「生成後に使う interface」で、変えるのは「どの具象を返すか」です。Factory Method は単一オブジェクトの生成切り替え、Abstract Factory は関連するオブジェクト群を一貫した組で返すときに使います。

```typescript
interface Logger {
  info(msg: string): void;
}

class ConsoleLogger implements Logger {
  info(msg: string) {
    console.log(msg);
  }
}
class NoopLogger implements Logger {
  info() {}
}

// 生成の判断はここだけ。呼び出し側は Logger だけ知っていればよい
function createLogger(env: string): Logger {
  return env === "production" ? new ConsoleLogger() : new NoopLogger();
}

// Abstract Factory: まとまりで整合した実装を返す
interface UiFactory {
  createButton(): { render(): string };
  createDialog(): { render(): string };
}
class LightUiFactory implements UiFactory {
  createButton() {
    return { render: () => "light-button" };
  }
  createDialog() {
    return { render: () => "light-dialog" };
  }
}
class DarkUiFactory implements UiFactory {
  createButton() {
    return { render: () => "dark-button" };
  }
  createDialog() {
    return { render: () => "dark-dialog" };
  }
}
```

DI コンテナを置いていないコードベースで、`new` の分岐を一か所に閉じたいときに向きます。テストで fake に切り替えたいときや、環境・プラン別に生成物を変えたいときにも使います。

一方で Factory が機能ごとに散在すると、どこで何が生成されるか追いにくくなります。生成条件が肥大化してきたら、アプリ起動時に組み立てを行う場所へ集約するか、DI コンテナや Registry に寄せる判断が必要です。

# 手順を組み替える：Template Method / Decorator / Chain of Responsibility

処理の手順や、前後に挟む責務を組み替える章です。チーム内でモジュールを組み立てる場面で出番が多くなります。

## Template Method

[Template Method](https://en.wikipedia.org/wiki/Template_method_pattern) は、処理の骨格（手順）を固定し、特定のステップだけ差し替える形です。Strategy が実装全体を入れ替えるのに対し、こちらは「流れは固定で、埋める場所だけ開ける」イメージです。

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
  protected async fetch() {
    // DB から日次売上を取得
  }
  protected async save(out: unknown) {
    // 集計結果を S3 に保存
  }
}
```

処理の流れは共通のまま、一部ステップだけ種別ごとに変えたいときに向きます。共通フローを基盤側で固定し、差分だけ実装側に任せたい場面でよく使われます。

差し替えが一ステップだけなら、Strategy やコールバックのほうが軽いこともあります。親クラスが肥大化して継承が深くなると変更が連鎖しやすいので、早い段階で合成（composition）へ寄せるかは意識しておくとよいです。

## Decorator

[Decorator](https://en.wikipedia.org/wiki/Decorator_pattern) は、本体の処理や interface は固定したまま、外側から責務を足すやり方です。Express や Koa のミドルウェアのように、関数を包んで連鎖させる形もこの一種です。

```typescript
type Handler = (req: Request) => Response;

const handleOrder: Handler = (req) => {
  // 本体の処理はここだけに閉じる
  return { ok: true } as Response;
};

function withAuth(inner: Handler): Handler {
  // 前処理を追加（認証）
  return (req) => {
    if (!req.headers.authorization) throw new Error("unauthorized");
    return inner(req);
  };
}

function withLogging(inner: Handler): Handler {
  // 前後にログを追加（計測・監査にも置き換えやすい）
  return (req) => {
    console.log(req.url);
    return inner(req);
  };
}

const decorated = withLogging(withAuth(handleOrder));
```

認証・ログ・キャッシュのように、interface を変えずに横から責務を足したいときに向きます。既存ロジックを残したまま、前処理・後処理だけ差し込みたい場面で扱いやすいです。

ラッパーが深くなるとデバッグやスタックトレースは読みにくくなります。公開する関数の形、ログの粒度、ラップ順序のルールは、チームで最初に決めておくと運用が楽です。

## Chain of Responsibility

[Chain of Responsibility](https://en.wikipedia.org/wiki/Chain-of-responsibility_pattern) は、複数ハンドラを鎖状につなぎ、順に渡して「誰かが引き取る or 全員が処理する」形です。途中で止める必要がなければ、単なる関数合成（`reduce` で順に適用するだけ）で足ります。Chain of Responsibility は「順に渡し、途中で止めるかどうか」が焦点です。

```typescript
type Next = () => Promise<Response>;
type Middleware = (req: Request, next: Next) => Promise<Response>;

// 認証ミドルウェア：条件を満たさなければここで止まる
const auth: Middleware = async (req, next) => {
  if (!req.headers.get("authorization"))
    return new Response("Unauthorized", { status: 401 });
  return next();
};

// ログミドルウェア：前後に処理を挟んで次へ渡す
const logging: Middleware = async (req, next) => {
  console.log(`--> ${req.method} ${req.url}`);
  const res = await next();
  console.log(`<-- ${res.status}`);
  return res;
};

// 本体
const handler = async (req: Request) => new Response("OK");

// 組み立て：順番を変えるだけで振る舞いが変わる
// logging -> auth -> handler
```

認証 → 認可 → レート制限 → 本体のように、順序付きで段を増やしたいときに向きます。常に同じ 1 ハンドラしか使わないなら直接呼ぶほうが素直です。ハンドラ同士が密に状態共有する構成や、チェーンが極端に長い場合は別の形を検討したほうがよいです。

# 参加者を足す：Registry / Observer / Plugin

同じ呼び出し口のまま、後から参加者を足していく章です。イベントの入口や dispatch（呼び分け）の仕組みは固定し、後から増えていく実装の集合を用意します。ただし Plugin は、足す主体が第三者（社外の開発者など）になります。

## Registry

[Registry](https://martinfowler.com/eaaCatalog/registry.html) は、キーと実装の対応を登録し、あとからキーで引いて呼び出す仕組みです。Strategy が「同じ interface の差し替え」だとすれば、Registry は「種別を増やしても dispatch（呼び分け）側のコードを触らない」ための形です。

```typescript
// キーと処理の対応を保持する
const handlers = new Map<string, (payload: unknown) => Promise<void>>();

// 新しい種別を足すときはここだけ
export function register(
  event: string,
  handler: (payload: unknown) => Promise<void>
) {
  handlers.set(event, handler);
}

// dispatch 側は種別が増えても変わらない
export async function dispatch(event: string, payload: unknown) {
  const handler = handlers.get(event);
  if (!handler) throw new Error(`No handler for ${event}`);
  await handler(payload);
}

// 使い方
register("order.created", async (p) => {
  /* 注文処理 */
});
register("order.cancelled", async (p) => {
  /* キャンセル処理 */
});
await dispatch("order.created", { orderId: "123" });
```

種別が後から増え続けるが、dispatch 側に `if` や `switch` を足したくないときに向きます。キーが素の文字列だとリネーム漏れが実行時まで気付けない不具合になりやすいので、`as const` や discriminated union で型を効かせると安全です。同じ種別に複数の反応が要るようになったら Observer、社外の開発者が登録するなら Plugin が次の検討先です。

## Observer

[Observer](https://en.wikipedia.org/wiki/Observer_pattern) は、ひとつのイベント（状態変化）に対して複数の独立した購読者を後から足す仕組みです。Registry が「1 キー＝1 ハンドラ」なのに対し、Observer は「1 イベント＝ N 個のリスナ」です。

```typescript
type Listener<T> = (event: T) => void;

class EventBus<Events extends Record<string, unknown>> {
  private listeners = new Map<string, Listener<any>[]>();

  // 購読者を後から足せる
  on<K extends keyof Events & string>(event: K, fn: Listener<Events[K]>) {
    const list = this.listeners.get(event) ?? [];
    list.push(fn);
    this.listeners.set(event, list);
  }

  // 発火すると全購読者が呼ばれる
  emit<K extends keyof Events & string>(event: K, payload: Events[K]) {
    for (const fn of this.listeners.get(event) ?? []) fn(payload);
  }
}

// 使い方：購読者同士は互いを知らない
const bus = new EventBus<{ "user.created": { id: string } }>();
bus.on("user.created", (e) => {
  /* メール送信 */
});
bus.on("user.created", (e) => {
  /* 監査ログ */
});
bus.emit("user.created", { id: "u-1" });
```

ひとつの変化に対して複数の独立した反応（メール・ログ・キャッシュ破棄など）を後から足したいときに向きます。通知順に依存すると壊れやすいので、順序不定で動く設計にしておくのが安全です。

## Plugin と SPI

[Plugin](<https://en.wikipedia.org/wiki/Plug-in_(computing)>) は、第三者（社外の開発者など）が拡張を独立して配布し、ホストが動的に取り込む仕組みです。Registry に動的読み込みと公開 API を足した形で実現することが多いです。Java の SPI（Service Provider Interface）の [`ServiceLoader`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ServiceLoader.html)（[SPI チュートリアル](https://docs.oracle.com/javase/tutorial/ext/basics/spi.html)）で説明される読み込み方も、同じ棚に並びます。

```typescript
// ホストが提供する API（Plugin から見える世界）
interface HostApi {
  registerCommand(name: string, run: () => void): void;
  onEvent(event: string, fn: () => void): void;
}

// Plugin の interface
interface Plugin {
  name: string;
  setup(host: HostApi): void;
}

// ホスト側：動的に読み込んで初期化する
async function loadPlugins(pluginNames: string[], host: HostApi) {
  for (const name of pluginNames) {
    const mod = await import(`./plugins/${name}.js`);
    const plugin: Plugin = mod.default;
    plugin.setup(host); // Plugin が自分で必要な登録をする
  }
}
```

ESLint・Prettier・VS Code のように、第三者が拡張を持ち寄るプロダクトで向きます。

公開 API を長く守るコストに見合わない社内ツールでは、Plugin まで行かずに済むことが多いです。

Observer や Registry が「内輪の参加者追加」だとすれば、Plugin は **境界の外へ参加者を開く別格の選択** です。社外の開発者が使う公開 API になると、ドキュメントに書いた契約だけでなく、実際に観測できる挙動まで依存されやすくなります（[Hyrum's Law](https://www.hyrumslaw.com/)）。設計パターンというより、公開プラットフォームを運営する判断に近いです。

### Plugin を公開するときの注意

一度公開した API は、[Hyrum's Law](https://www.hyrumslaw.com/) が言うように、ドキュメントに書かれていない振る舞いまで依存として固定されやすくなります。バージョン方針は [Semantic Versioning](https://semver.org/lang/ja/) とセットで決め、SDK・サンプル・移行ガイドまでコストに含めて判断する必要があります。「将来 Plugin 化」と言い始めたら、まず Strategy や Registry で足りないかを確認するのが無難です。

# 型やルールで宣言する：discriminated union / DSL

実行時に差し込む代わりに、型やルールという宣言の形で「変えてよい場所」を決めておく章です。コアとチーム内の両方に顔を出すパターンがあるので、見取り図では行が分かれています。

## discriminated union と exhaustiveness checking

[discriminated union（判別可能ユニオン）](https://www.typescriptlang.org/docs/handbook/2/narrowing.html) と exhaustiveness checking（網羅性チェック）を使うと、種類を足したときにコンパイルが漏れを教えてくれます。種類が増えたときの分岐漏れを、コンパイラに見つけてもらえます（Open-Closed Principle、[Bertrand Meyer, OOSC2](https://bertrandmeyer.com/wp-content/upLoads/OOSC2.pdf) の精神に近い扱いです）。

```typescript
// 種類を増やすとき → ここに種類（バリアント）を足すだけ
type PaymentEvent =
  | { kind: "charged"; amount: number }
  | { kind: "refunded"; amount: number; reason: string }
  | { kind: "disputed"; caseId: string };

function handle(event: PaymentEvent): string {
  switch (event.kind) {
    case "charged":
      return `charged ${event.amount}`;
    case "refunded":
      return `refunded ${event.amount}: ${event.reason}`;
    case "disputed":
      return `disputed: ${event.caseId}`;
    default:
      // バリアントを足して case を書き忘れるとここでコンパイルエラー
      const _: never = event;
      return _;
  }
}
```

種類がアプリと一緒にデプロイされ、実行時に第三者が新しい型を持ち込まない「閉じた世界」なら第一候補です。「まず設定 DB に」と考える前に、型で表現できないかを検討する価値があります。データ型を固定したまま操作だけ増やす形（古典的には Visitor）も、TypeScript ではこの discriminated union と関数分割でそのまま書けます。

## DSL とルールエンジン

分岐がコード中に増えてきたとき、ルールを宣言として並べ直すアプローチです（[Internal Reprogrammability](https://martinfowler.com/bliki/InternalReprogrammability.html)、[Rules Engine](https://martinfowler.com/bliki/RulesEngine.html)、[DSL ガイド](https://martinfowler.com/dsl.html)）。

```typescript
type Ctx = { tenureMonths: number; region: string };
type Rule = { test: (c: Ctx) => boolean; grantDays: number };

// ルールを宣言的に並べる（レビュアが読みやすい）
const vacationRules: Rule[] = [
  { test: (c) => c.tenureMonths >= 12, grantDays: 15 },
  { test: (c) => c.region === "EU", grantDays: 5 },
];

// 評価は単純にマッチしたルールの合計
function calcVacation(ctx: Ctx): number {
  return vacationRules
    .filter((r) => r.test(ctx))
    .reduce((sum, r) => sum + r.grantDays, 0);
}
```

ルールの見え方をレビュアや業務担当に合わせたいとき、分岐がコードの中に散らばるよりも一覧で見たいときに向きます。Fowler が書いているように、非エンジニアが直接書けることを目標にすると失敗しやすく、「読みやすさ・レビューしやすさ」を目的にするのが現実的です。

# 設計レビューでの使い方

レビューでは、次の 3 つを言語化すると議論がぶれにくくなります（見取り図の直後にも同じ問いを置いています）。

1. **主体（誰が拡張するか）**（コア開発者 / チーム内 / 第三者）
2. **やること（どう変えるか）**（実装を差し替える / 手順を組み替える / 参加者を足す / 型やルールで宣言する）
3. **一度開いた境界を、あとから畳めるか**（互換性・移行コストの見積り）

ここまで決まれば、Plugin か Strategy か、Registry か discriminated union か、といった選択は「表のどのセルか」にほぼ収まります。迷ったら表のセルから本文の該当節に戻るのが速いです。

# おわりに

拡張点の設計は、パターン名より先に**主体とやること**を決めたほうが、あとからの負債と向き合いやすいと感じています。文献の呼び方は揺れますが、軸さえ共有できればレビューで同じ地図を見られます。

一度足した境界を畳むのは簡単ではありません。想定外の要件が来たときは、コードを書く前に主体とやることをメモに書き出すだけでも、判断の質は変わることがあります。段階的な置き換えは [Strangler Fig Application](https://martinfowler.com/bliki/StranglerFigApplication.html) の話も参照してください。

銀の弾丸ではありませんが、名前に振り回されたときに「誰が何を変えるのか」へ戻れる足場になればうれしいです👋
