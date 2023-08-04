---
title: "「良いコード/悪いコードで学ぶ設計入門」の理解をPHP(Laravel)で深めよか"
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Laravel", "設計", "品質", "ミノ駆動本"]
published: false
---

# TL;DR

- 「良いコード/悪いコードで学ぶ設計入門」をテーマに、PHP(Laravel)を使用してプログラミング設計原則を詳解
- 各トピックごとに「問題のあるコード」例と「改良されたコード」例を PHP(Laravel)で提供し、良い設計原則に従う方法を具体的に提示
- これを読めば、設計原則の理解が深まり自身の開発に活かせるはず
- だいぶはしょったので、気になった人はぜひ購入を

https://gihyo.jp/book/2022/978-4-297-12783-1

# 第 1 章 悪しき構造の弊害を知覚する

## 理解を困難にする条件分岐のネスト

### 問題のあるコード

条件分岐が深くネストされてしまうと、コードの可読性は大きく低下し、結果的に理解が困難になる傾向があります。さらに、条件の複雑さが増すと、それに伴ってバグの発生率も高まる可能性があります。

以下に、このネストした条件分岐の問題を具体的に示すコード例を示します。

```php
class UserController extends Controller {
    public function show($id) {
        $user = User::find($id);

        if ($user) {
            if ($user->isActive()) {
                return view('user.show', ['user' => $user]);
            } else {
                return redirect('home')->with('error', 'User is not active');
            }
        } else {
            return redirect('home')->with('error', 'User not found');
        }
    }
}
```

### 改良されたコード

問題のあるコードを見ると、二つの条件、ユーザーが存在するかどうか、そしてユーザーがアクティブかどうかという条件がネストしています。

このようなネストした条件分岐は、「ガード節」を使用して平坦化することができます。ガード節とは、関数の先頭でエラーケースをチェックして早期にリターンするというテクニックです。

以下に、ガード節を使用して改善したコードを示します。

```php
class UserController extends Controller {
    public function show($id) {
        $user = User::find($id);

        if (!$user) {
            return redirect('home')->with('error', 'User not found');
        }

        if (!$user->isActive()) {
            return redirect('home')->with('error', 'User is not active');
        }

        return view('user.show', ['user' => $user]);
    }
}
```

## 未初期化状態（生焼けオブジェクト）

### 問題のあるコード

**未初期化状態**、または**生焼けオブジェクト**とは、コンストラクタが完全にオブジェクトを初期化せず、未完全な状態でオブジェクトが存在することを指します。これはしばしばバグや不明な挙動を引き起こす原因となります。

以下に、未初期化状態の問題を示す Laravel のサンプルコードを示します。

```php
class User {
    public $name;
    public $email;

    public function __construct($name) {
        $this->name = $name;
    }
}

$user = new User('John Doe');
echo $user->email; // null
```

### 改良されたコード

上記のコードでは、`User`クラスのコンストラクタは`$name`だけを初期化し、`$email`は初期化していません。したがって、新しい`User`オブジェクトを作成した直後に`$email`にアクセスすると、未初期化の状態（`null`）が返されます。

このような問題を解決するためには、コンストラクタで全ての必要なフィールドを初期化すべきです。

```php
class User {
    private $name;
    private $email;

    public function __construct($name, $email) {
        $this->name = $name;
        $this->email = $email;
    }

    public function getName() {
        return $this->name;
    }

    public function getEmail() {
        return $this->email;
    }
}

$user = new User('John Doe', 'john@example.com');
echo $user->getEmail(); // john@example.com
```

上記の修正後のコードでは、コンストラクタは全てのフィールド（`$name`と`$email`）を初期化します。したがって、新しい`User`オブジェクトを作成した直後に`$email`にアクセスしても、未初期化の状態を返すことはありません。これによりオブジェクトの生存期間中における状態の予測可能性が向上します。

# 第 2 章　設計の初歩

## ベタ書きせず、意味のあるまとまりでメソッド化

### 問題のあるコード

コードの中に長いベタ書きのロジックがあると、その部分の意図が理解しにくく、また再利用やテストも難しくなります。

以下にそのようなコードの例を示します。

```php
class UserController extends Controller
{
    public function update(Request $request, User $user)
    {
        $request->validate([
            'name' => 'required|max:255',
            'email' => 'required|email|unique:users,email,' . $user->id,
        ]);

        $user->name = $request->name;
        $user->email = $request->email;
        $user->save();

        return redirect()->route('users.show', $user);
    }
}
```

ここで問題なのは、バリデーションロジックがコントローラに混在しているため、コードの見通しが悪くなり、再利用性やテスト性も損なわれていることです。

### 改良されたコード

これを改善するために、`FormRequest`クラスを使ってバリデーションロジックを切り離します。

```php
class UserController extends Controller
{
    public function update(UpdateUserRequest $request, User $user)
    {
        $user->name = $request->name;
        $user->email = $request->email;
        $user->save();

        return redirect()->route('users.show', $user);
    }
}
```

```php
class UpdateUserRequest extends FormRequest
{
    public function authorize()
    {
        return true; // Or implement your authorization logic
    }

    public function rules()
    {
        $userId = $this->route('user')->id;

        return [
            'name' => 'required|max:255',
            'email' => 'required|email|unique:users,email,' . $userId,
        ];
    }
}
```

これにより、コントローラのコードがスッキリとし、再利用性やテスト性が向上します。また、バリデーションロジックが一箇所にまとまっているため、修正や保守が容易になります。

## 関係し合うデータとロジックをクラスにまとめる

### 問題のあるコード

以下のようなコードでは、`User`と`Company`の関係を適切に表現できていないため、ロジックが冗長になり、メンテナンスも困難になります。

```php
class UserController extends Controller
{
    public function assignCompany(Request $request, User $user)
    {
        $company = Company::find($request->companyId);
        if (!$company) {
            return redirect()->back()->with('error', 'Company not found');
        }

        $user->companyId = $company->id;
        $user->save();

        return redirect()->route('users.show', $user);
    }
}
```

このコードでは、ユーザーを企業に割り当てるロジックが`UserController`に存在しますが、ユーザーと企業の関係に関するロジックは、本来は`User`クラスまたは`Company`クラスで管理すべきです。

### 改良されたコード

```php
class User extends Model
{
    public function assignCompany($company)
    {
        $this->companyId = $company->id;
        $this->save();
    }
}
```

```php
class UserController extends Controller
{
    public function assignCompany(Request $request, User $user)
    {
        $company = Company::find($request->companyId);
        if (!$company) {
            return redirect()->back()->with('error', 'Company not found');
        }

        $user->assignCompany($company);

        return redirect()->route('users.show', $user);
    }
}
```

この改善版では、ユーザーに企業を割り当てるロジックを`User`クラスに移動させました。これにより、`User`と`Company`の関係に関するロジックが一箇所にまとまり、コードの保守性と再利用性が向上します。また、`User`クラスと`Company`クラスが直接的に関係するロジックは、それぞれのクラスに含められることで、より直感的に理解できるようになります。

# 第 3 章　設計の初歩

## 成熟したクラスへ成長させる設計術

### 問題のあるコード

以下のコードは、ユーザーの生年月日から年齢を計算するシンプルな設計です。ユーザーの生年月日と現在日を格納し、それらを元に年齢を計算するクラスがそれぞれ定義されています。

```php
class User
{
    public $birthdate;
    public $today;

    public function setBirthdate($birthdate)
    {
        $this->birthdate = $birthdate;
    }

    public function setToday($today)
    {
        $this->today = $today;
    }

    public function getBirthdate()
    {
        return $this->birthdate;
    }

    public function getToday()
    {
        return $this->today;
    }
}

class AgeCalculator
{
    public function calculateAge($birthdate, $today)
    {
        return Carbon::createFromFormat('Y-m-d', $birthdate)->diff(Carbon::createFromFormat('Y-m-d', $today))->format('%y');
    }
}
```

しかしながら、このコードでは、以下の問題が存在します。

- コンストラクタが実装されていないため、初期化されていないプロパティが利用される可能性がある
- 計算ロジックがデータ保持側に存在しない
- インスタンス変数とメソッド引数が不変でないため、値の変更が発生しやすい
- 生年月日と現在の日付のように、特定の形式や規則を必要とする値について、間違った形式の値が渡される可能性があります

### 改良されたコード

そこで、以下のように修正します。

```php
class Birthdate
{
    private $value;

    public function __construct($value)
    {
        if (!preg_match('/\d{4}-\d{2}-\d{2}/', $value)) {
            throw new InvalidArgumentException('Invalid birthdate format. It should be YYYY-MM-DD');
        }
        $this->value = $value;
    }

    public function getValue()
    {
        return $this->value;
    }
}

class Today
{
    private $value;

    public function __construct($value)
    {
        if (!preg_match('/\d{4}-\d{2}-\d{2}/', $value)) {
            throw new InvalidArgumentException('Invalid date format for today. It should be YYYY-MM-DD');
        }
        $this->value = $value;
    }

    public function getValue()
    {
        return $this->value;
    }
}

class User
{
    private $birthdate;
    private $today;

    public function __construct(Birthdate $birthdate, Today $today)
    {
        $this->birthdate = $birthdate;
        $this->today = $today;
    }

    public function getBirthdate()
    {
        return $this->birthdate->getValue();
    }

    public function getToday()
    {
        return $this->today->getValue();
    }

    public function calculateAge()
    {
        return Carbon::createFromFormat('Y-m-d', $this->birthdate->getValue())->diff(Carbon::createFromFormat('Y-m-d', $this->today->getValue()))->format('%y');
    }
}
```

上記の改善後のコードでは、以下のように問題が解決されています。

- コンストラクタを使用して、プロパティを適切に初期化しています
- User クラスにデータとその計算ロジックをまとめ、一貫性を保てるようにしました
- プロパティを不変に保つために、`private`で宣言し、外部からの直接アクセスを防ぐために Getter メソッドを使用しています
- 生年月日と現在の日付のように、特定の形式や規則を持つ値については、値オブジェクト（Value Object）を作成することで値の誤りを防ぐことが可能です

以上のように、各クラスが単体で正常に動作し、それぞれが持つべきデータとロジックを適切に配置することで、コードの可読性や保守性を向上させることができます。

# 第 4 章 不変の活用

## 可変がもたらす意図せぬ影響

### 問題のあるコード

**可変**というのは、データが変更できるという特性を指します。プログラミングにおいては変数の値を変更できるという意味を持ちます。しかしながら、この可変性は意図せぬ影響を引き起こす可能性があります。特に大規模なアプリケーションや複数人での開発では、誤って変数の値を変えてしまうとバグの原因になることがあります。

以下に示すコードはその一例です。

```php
class User
{
    public $name;

    public function __construct(string $name)
    {
        $this->name = $name;
    }
}

$user = new User('John');
echo $user->name; // "John"

$user->name = 'Tom';
echo $user->name; // "Tom"
```

このコードでは、`User`クラスの`name`プロパティが`public`で宣言されているため、外部から自由に値を変更することができます。こうした自由度は、意図せぬ値の変更を引き起こす可能性があります。

### 改良されたコード

不変性を活用することで、これらの問題を軽減することが可能です。

以下に示すコードはその一例です。

```php
class User
{
    public readonly string $name;

    public function __construct(string $name)
    {
        $this->name = $name;
    }
}

$user = new User('John');
echo $user->name; // "John"

$user->name = 'Tom'; // Error: Cannot modify readonly property User::$name
```

このコードでは、`User`クラスの`name`プロパティが`readonly`として宣言されています。これにより、値はコンストラクタで設定された後は変更することができません。そのため、意図せぬ値の変更を防ぐことができます。このように、`readonly`プロパティはクラスの状態を安全に保つための強力なツールとなります。また、クラスの不変性を強制することでコードの読みやすさも向上します。

# 第 5 章 低凝集 -バラバラになったモノたち-

## 初期化ロジックの分散

### 問題のあるコード

コードの初期化ロジックが分散している場合、そのコードは理解しにくくなり、エラーが発生しやすくなります。また、新たな初期化ロジックを追加する際の困難さも増します。

```php
class Point {
    public $value;

    public function __construct($value) {
        $this->value = $value;
}
    }

$standardPoint = new Point(3000);
$premiumPoint = new Point(10000);
```

このコードでは初期化ロジックが分散しており、理解しにくくなっています。具体的には、`Point`クラスのインスタンス作成時に初期値が直接渡されています。これにより、初期値が何を表しているのか、またなぜそのような値が設定されているのかがコードから読み取ることができません。さらに、新たな初期化ロジックを追加する際、各インスタンス作成の部分すべてを修正しなければならず、手間が増えます。

### 改良されたコード

```php
class Point {
    private $value;

    private function __construct($value) {
        $this->value = $value;
    }

    public static function createStandardPoint() {
        return new self(3000);
    }

    public static function createPremiumPoint() {
        return new self(10000);
    }

    public function getValue() {
        return $this->value;
    }
}

$standardPoint = Point::createStandardPoint();
$premiumPoint = Point::createPremiumPoint();
```

改善策としては、`Point`クラスに`private`コンストラクタを設け、ファクトリメソッド`createStandardPoint`と `createPremiumPoint` を用意します。このファクトリメソッドを使うことで、会員種別によってポイントの初期値が適切に設定されることが保証されます。つまり、初期化ロジックが一箇所にまとまり、新たなロジックを追加する際もそのファクトリメソッド内で行えばよいため、メンテナンス性が向上します。また、`Point`クラス自体の値は外部から直接変更できないようにし、`getValue`メソッドを通じてのみ取得できるようにします。これにより、`Point`クラスの値の整合性と保守性が向上します。

## 共通処理クラス

### 問題のあるコード

共通処理クラスとは、異なる箇所で再利用可能なコードを抽出したクラスのことを指します。ただし、共通処理クラスを作成する際には、そのクラスが担当する責任を明確にすることが重要です。

```php
class Common {
    public function authenticate($username, $password) {
        // authenticate user
    }

    public function sendMail($userEmail, $subject, $body) {
        // send mail
    }

    public function calculateTax($income) {
        // calculate tax
    }

    public function generateReport($data) {
        // generate report
    }

    public function logError($error) {
        // log error
    }
}
```

このコードの`Common`クラスは、ユーザーの認証、メールの送信、税金の計算、レポートの生成、エラーのログ記録といった、全く関連性のない 5 つの責任を持っています。これにより、各メソッドがどのように互いに影響を与えるか理解するのが難しくなり、テストや保守も一層困難になります。
これを改善するには、各機能を独自のクラスに分割することが有効です。これにより、各クラスが一つの責任だけを持つことになり、コードの理解、テスト、保守が容易になります。また、一つのクラスが変更される理由が明確になり、予期しない副作用を防ぐことが可能になります。

### 改良されたコード

```php
class Authenticator {
    public function authenticate($username, $password) {
        // authenticate user
    }
}

class Mailer {
    public function sendMail($userEmail, $subject, $body) {
        // send mail
    }
}

class TaxCalculator {
    public function calculateTax($income) {
        // calculate tax
    }
}

class ReportGenerator {
    public function generateReport($data) {
        // generate report
    }
}

class ErrorLogger {
    public function logError($error) {
        // log error
    }
}
```

このようにすることで、各クラスは単一の責任を持つことになり、コードの理解性や保守性が向上します。さらに、一つのクラスが変更される理由が明確になり、予期しない副作用も防ぐことができます。

# 第 6 章 条件分岐 -迷宮化した分岐処理を解きほぐす技法-

## switch 文の重複

### 問題のあるコード

一つのプログラム内で同じコードが何度も登場することはよくあります。しかし、これはプログラムの読みやすさ、保守性、拡張性を損なう可能性があるため、可能な限り避けるべきです。特に、条件分岐の中でのコードの重複は複雑性を増加させ、エラーの元になりやすいです。

例えば以下のような PHP コードを考えてみましょう。

```php
class OrderController extends Controller
{
    public function shipOrder(Request $request, $id)
    {
        $order = Order::find($id);

        switch ($request->input('status')) {
            case 'processing':
                // 処理中の注文の処理
                break;
            case 'shipped':
                // 出荷された注文の処理
                break;
            case 'delivered':
                // 配送済みの注文の処理
                break;
            default:
                return response()->json(['status' => 'failed', 'message' => 'Invalid status'], 400);
        }

        // コード続き…
    }

    public function cancelOrder(Request $request, $id)
    {
        $order = Order::find($id);

        switch ($request->input('status')) {
            case 'processing':
                // 処理中の注文のキャンセル処理
                break;
            case 'shipped':
                // 出荷された注文のキャンセル処理
                break;
            case 'delivered':
                // 配送済みの注文のキャンセル処理
                break;
            default:
                return response()->json(['status' => 'failed', 'message' => 'Invalid status'], 400);
        }

        // コード続き…
    }
}
```

これらのメソッドでは、同じスイッチ文が複数の場所に記述されています。ここでは、それぞれのメソッドで注文の状態に基づいて処理を行っています。

### 改良されたコード 1

解決策の一つとして、注文のステータスを処理する別のクラスを以下のように作成することが考えられます。

```php
class OrderStatusHandler
{
    public function handle($status, Order $order)
    {
        switch ($status) {
            case 'processing':
                // 処理中の注文の処理
                break;
            case 'shipped':
                // 出荷された注文の処理
                break;
            case 'delivered':
                // 配送済みの注文の処理
                break;
            default:
                return false;
        }

        return true;
    }
}

class OrderController extends Controller
{
    protected $statusHandler;

    public function __construct(OrderStatusHandler $statusHandler)
    {
        $this->statusHandler = $statusHandler;
    }

    public function shipOrder(Request $request, $id)
    {
        $order = Order::find($id);

        if(!$this->statusHandler->handle($request->input('status'), $order)){
            return response()->json(['status' => 'failed', 'message' => 'Invalid status'], 400);
        }

        // コード続き…
    }

    public function cancelOrder(Request $request, $id)
    {
        $order = Order::find($id);

        if(!$this->statusHandler->handle($request->input('status'), $order)){
            return response()->json(['status' => 'failed', 'message' => 'Invalid status'], 400);
        }

        // コード続き…
    }
}
```

これにより、注文のステータスを処理するコードが一箇所にまとまり、コードの重複が避けられます。

さらに、各ステータスごとに異なる振る舞いを持つクラス（ステートパターン）を作成することで、スマートにスイッチ文の重複を解消することもできます。ただし、この解決策は問題の規模や状況によりますので、ケースバイケースで適用を検討してください。

### 改良されたコード 2

インターフェースを使用することで、より効果的にスイッチ文の重複を解消できます。これは、特にオブジェクト指向プログラミングのポリモーフィズム（多態性）の原則を活用する方法です。

各ステータスを個別のクラスとし、それらが共通のインターフェースを実装する形にすることができます。各クラスはステータスごとの具体的な振る舞いを定義し、コントローラーではステータスに依存せずに振る舞いを呼び出すだけで良くなります。

まず、注文状態を扱うインターフェースとこのインターフェースを実装した具体的な状態クラスを作成します。

```php
interface OrderStatusInterface
{
    public function handle(Order $order);
}

class ProcessingOrderStatus implements OrderStatusInterface
{
    public function handle(Order $order)
    {
        // 処理中の注文の処理
    }
}

class ShippedOrderStatus implements OrderStatusInterface
{
    public function handle(Order $order)
    {
        // 出荷された注文の処理
    }
}

class DeliveredOrderStatus implements OrderStatusInterface
{
    public function handle(Order $order)
    {
        // 配送済みの注文の処理
    }
}
```

このとき、状態に対応するクラスのインスタンスをマップとして定義し、それを使用して注文の処理を実行する`OrderStatusService`クラスを作成します。

```php
class OrderStatusService
{
    protected $statusHandlers = [
        'processing' => ProcessingOrderStatus::class,
        'shipped' => ShippedOrderStatus::class,
        'delivered' => DeliveredOrderStatus::class,
    ];

    public function handle($status, Order $order)
    {
        if (!isset($this->statusHandlers[$status])) {
            throw new Exception('Invalid status');
        }

        $handlerClass = $this->statusHandlers[$status];
        $handler = new $handlerClass();
        $handler->handle($order);
    }
}
```

そして、コントローラーで`OrderStatusService`クラスを使って注文を処理します。

```php
class OrderController extends Controller
{
    protected $statusService;

    public function __construct(OrderStatusService $statusService)
    {
        $this->statusService = $statusService;
    }

    public function shipOrder(Request $request, $id)
    {
        $order = Order::find($id);

        try {
            $this->statusService->handle($request->input('status'), $order);
        } catch (Exception $e) {
            return response()->json(['status' => 'failed', 'message' => 'Invalid status'], 400);
        }

        // コード続き…
    }

    public function cancelOrder(Request $request, $id)
    {
        $order = Order::find($id);

        try {
            $this->statusService->handle($request->input('status'), $order);
        } catch (Exception $e) {
            return response()->json(['status' => 'failed', 'message' => 'Invalid status'], 400);
        }

        // コード続き…
    }
}
```

この方法では、新たな`OrderStatusService`クラスが登場します。ここではマッピング（連想配列）を使い、注文状態とそれに対応するクラスのインスタンスを管理しています。これにより、注文状態を追加または変更するときは、このマッピングに対応する項目を追加または変更するだけで、コントローラーを変更する必要はありません。つまり、この構成は、アプリケーションの将来の拡張性を向上させ、コードのメンテナンスを容易にします。

このように、スイッチ文の重複問題は、オブジェクト指向プログラミングのポリモーフィズム原則を適用することで効果的に解消できます。それぞれのソリューションには、それぞれの適用シーンとメリットがありますので、プロジェクトの要件と目標により選択してください。
