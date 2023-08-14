---
title: "悪いコードから知る保守性の真価"
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

# はじめに

コードは、単に機能を実現するための文字列の集まりではありません。それは、後のメンテナンス、拡張、および改修の基盤となり、製品の品質や開発チームの生産性に直接影響します。悪いコードは、後の段階での修正や追加が難しくなるだけでなく、バグの原因となることが多いです。さらに、新しいメンバーがそのコードを理解する際の学習コストが増加します。結果として、プロジェクトの総コストが上昇し、リリースの遅延や品質の低下を招くことがあります。

この記事は、「良いコード/悪いコードで学ぶ設計入門」という本を基に、悪いコードの特徴とその影響、そしてより良いコードへの道を探るものです。具体的には **修正性(=変更容易性)** に焦点を当て、その低いコードがもたらす問題と、それをどのように改善できるかを示しています。

悪いコードの問題点を正確に把握し、それに対する改善策を知ることで、読者は具体的なコード改善のノウハウと、設計の基本的な考え方を学ぶことができるでしょう。設計のスキルは、単に良いコードを書くためだけではなく、チームの生産性や製品の品質向上にも繋がります。

# システム・ソフトウェア製品における品質特性

| 品質特性     | 品質副特性                   | 説明                                                                                         |
| ------------ | ---------------------------- | -------------------------------------------------------------------------------------------- |
| 機能適合性   | 機能完全性                   | ソフトウェアが特定の要求を満たす機能を持っているかどうか                                     |
|              | 機能正確性                   | ソフトウェアが正確な結果や出力を提供するかどうか                                             |
|              | 機能適切性                   | ソフトウェアが特定の要求や期待に適切であるかどうか                                           |
| 性能効率性   | 時間効率性                   | ソフトウェアの処理時間、応答時間、通信時間などが効率的かどうか                               |
|              | 資源効率性                   | ソフトウェアがメモリ、ストレージ、ネットワークリソースなどの消費を最小化しているかどうか     |
|              | 容量満足性                   | ソフトウェアがその容量やスケーラビリティを考慮して効果的に動作するかどうか                   |
| 互換性       | 共存性                       | ソフトウェアが他の製品やシステムと共存できるかどうか                                         |
|              | 相互運用性                   | ソフトウェアが異なる環境やシステムと情報をやり取りや協力して動作できるかどうか               |
| 使用性       | 適切度認識性                 | ソフトウェアがユーザーにとって期待通りの機能や性能を提供するかどうか                         |
|              | 習得性                       | ユーザーがソフトウェアの機能をどれだけ簡単に学べるか                                         |
|              | 運用操作性                   | ユーザーがソフトウェアを効果的、効率的に操作できるかどうか                                   |
|              | ユーザエラー防止性           | ソフトウェアがユーザーのエラーを防ぐための機能や手段を持っているかどうか                     |
|              | ユーザインターフェイス快美性 | ソフトウェアのユーザインターフェイスが魅力的であるかどうか                                   |
|              | アクセシビリティ             | さまざまなユーザーや状況に適応してソフトウェアが利用できるかどうか                           |
| 信頼性       | 成熟性                       | ソフトウェアが予期しない終了やエラーを回避する能力                                           |
|              | 可用性                       | ソフトウェアが確実に、正確に動作し続ける能力                                                 |
|              | 障害許容性                   | ソフトウェアが障害が発生しても正常に動作を続ける能力                                         |
|              | 回復性                       | ソフトウェアが何らかの問題や障害から正常な状態に復旧する能力                                 |
| セキュリティ | 機密性                       | ソフトウェアが情報を保護して不正なアクセスを防ぐ能力                                         |
|              | インテグリティ               | ソフトウェアが情報の正確性と完全性を保護する能力                                             |
|              | 否認防止性                   | ソフトウェアが行為やイベントの証拠を提供し、その否認を防ぐ能力                               |
|              | 責任追跡性                   | ソフトウェアが行為やイベントの原因を特定する能力                                             |
|              | 真正性                       | ソフトウェアがエンティティの真正性を確認する能力                                             |
| 保守性       | モジュール性                 | ソフトウェアの各部分が独立して機能し、他の部分との相互作用が最小限であるかどうか             |
|              | 再利用性                     | ソフトウェアの一部や全体が他のアプリケーションやモジュールでの使用に適しているかどうか       |
|              | 解析性                       | ソフトウェアが問題を特定、修正するためにどれだけ分析しやすいか                               |
|              | 修正性                       | ソフトウェアが変更や修正を行いやすいかどうか                                                 |
|              | 試験性                       | ソフトウェアがテストを行いやすく、結果が予測しやすいかどうか                                 |
| 移植性       | 適応性                       | ソフトウェアが異なる環境や条件に適応する能力                                                 |
|              | 設置性                       | ソフトウェアが新しい環境にインストールや移動するのが容易であるかどうか                       |
|              | 置換性                       | ソフトウェアが同じ目的の他のソフトウェアとの代替が可能か、または共存することができるかどうか |

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

## 形チェックで分岐しないこと

### 問題のあるコード

ここでは、`Product`という基底クラスと、そのサブクラスである`DigitalProduct`と`PhysicalProduct`を使用しています。基底クラスには`getPrice`メソッドが存在しますが、サブクラスではそれぞれ異なる方法で価格を計算します。

```php
class Product {
    protected $price;

    public function getPrice() {
        return $this->price;
    }
}

class DigitalProduct extends Product {
    public function getPrice() {
        return $this->price * 0.9;  // デジタル製品は10%引き
    }
}

class PhysicalProduct extends Product {
    public function getPrice() {
        return $this->price + 10;  // 物理製品は送料として10追加
    }
}

class ProductPrinter {
    public function printPrice(Product $product) {
        if ($product instanceof DigitalProduct) {
            echo "Digital Product Price: " . $product->getPrice();
        } elseif ($product instanceof PhysicalProduct) {
            echo "Physical Product Price: " . $product->getPrice();
        }
    }
}
```

この`ProductPrinter`クラスの printPrice メソッドは型による条件分岐を行っています。これはリスコフの置換原則を違反しています。リスコフの置換原則とは、「サブクラスはそのスーパークラスと置換可能でなければならない」という原則です。すなわち、型による条件分岐はこの原則を破る行為となります。

### 改良されたコード

リスコフの置換原則を適用するためには、各サブクラスでメソッドの振る舞いをオーバーライドするだけでなく、共通のインターフェイスを使用してこれらのクラスを扱うことが必要です。具体的には、サブクラスが各自の`printPrice`メソッドを実装し、それを呼び出すようにします。

```php
interface PricedProduct {
    public function getPrice(): float;
    public function printPrice(): void;
}

class DigitalProduct implements PricedProduct {
    protected $price;

    public function getPrice() {
        return $this->price * 0.9;  // デジタル製品は10%引き
    }

    public function printPrice() {
        echo "Digital Product Price: " . $this->getPrice();
    }
}

class PhysicalProduct implements PricedProduct {
    protected $price;

    public function getPrice() {
        return $this->price + 10;  // 物理製品は送料として10追加
    }

    public function printPrice() {
        echo "Physical Product Price: " . $this->getPrice();
    }
}

class ProductPrinter {
    public function print(PricedProduct $product) {
        $product->printPrice();
    }
}
```

このように、各製品クラスが`PricedProduct`インターフェイスを実装することで、`ProductPrinter`は単純に`print`メソッドを使って価格を出力するだけで、具体的な製品の型を知る必要はありません。これにより、形チェックでの分岐を避けて、リスコフの置換原則に従うことができます。

##　低凝集なコレクション処理

### 問題のあるコード

低凝集とは、一つのクラスまたはモジュールが行うべき役割が不明確で、多岐にわたる機能を持ってしまっている状態を指します。その結果、コードの再利用性や可読性が低下し、メンテナンスが困難になることが多いです。

この問題が発生する一例を以下に示します。ここでは、`UserService`と`SalesReportService`クラスが共通のコレクション処理（18 歳以上のユーザーをフィルタリングする）をそれぞれ独自に実装しています。

```php
class UserService {
    public function getAdultUsers() {
        $users = App\Models\User::all();
        return $users->filter(function($user){
            return $user->age >= 18;
        });
    }
}

class SalesReportService {
    public function getAdultUsersWithPurchaseHistory() {
        $users = App\Models\User::all();
        return $users->filter(function($user){
            return $user->age >= 18 && $user->purchases->count() > 0;
        });
    }
}
```

このコードでは、2 つのサービスクラスが重複して同じフィルタリングロジックを持っています。この状態では、同じロジックが散らばっているため、コードの再利用性が低下し、また修正が必要となった際には複数箇所を変更する必要が出てきます。

### 改良されたコード

この問題の解決策として、共通のコレクション処理を一つのクラス（`UserFilter`）にまとめ、それを使用してフィルタリングを行うように改良します。

```php
class UserFilter {
    private $users;

    public function __construct($users) {
        $this->users = $users;
    }

    public function filterAdultUsers() {
        return $this->users->filter(function($user) {
            return $user->age >= 18;
        });
    }
}

class UserService {
    public function getAdultUsers() {
        $users = new UserFilter(App\Models\User::all());
        return $users->filterAdultUsers();
    }
}

class SalesReportService {
    public function getAdultUsersWithPurchaseHistory() {
        $users = new UserFilter(App\Models\User::all());
        return $users->filterAdultUsers()->filter(function($user) {
            return $user->purchases->count() > 0;
        });
    }
}
```

この改良により、フィルタリングロジックが一箇所に集まり、コードの可読性と再利用性が向上します。また、将来的にフィルタリングロジックが変更になった場合も、一箇所の修正だけで対応が可能となり、メンテナンス性が向上します。

## 単一責任の原則違反で生まれる悪魔

### 問題のあるコード

プログラミングにおける設計原則は、コードの品質を維持しつつ、将来的な変更に対応できるような設計をするための道しるべとなります。しかし、これらの原則を無視したコードは、保守性や拡張性が低下する可能性があります。

例えば以下のような PHP コードを考えてみましょう。

```php
namespace App\Discounts;

class Discount
{
    protected $discountAmount;

    public function __construct()
    {
        $this->discountAmount = 300;
    }

    public function applyDiscount($price)
    {
        $discountedPrice = $this->calculateDiscountedPrice($price);

        return max($discountedPrice, 0);
    }
}

class SummerDiscount
{
    private $discount;

    public function __construct(Discount $discount)
    {
        $this->discount = $discount;
    }

    public function calculateDiscountedPrice($price)
    {
        return $price - $this->discount->applyDiscount();
    }
}
```

この設計には複数の問題が存在します。

1 つ目は単一責任の原則（Single Responsibility Principle, SRP）の違反です。`Discount`クラスは割引額の管理と割引の適用、という 2 つの責任を持っています。これにより、このクラスの役割が不明瞭になり、コードの理解と保守が困難になる可能性があります。
2 つ目は依存性の逆転の原則（Dependency Inversion Principle, DIP）の違反です。`SummerDiscount`クラスが具体的な`Discount`クラスに依存しています。そのため、異なる割引額を適用したいときに、`SummerDiscount`クラス自体を修正する必要があります。これは、クラス間の結合度が高くなり、コードの再利用性や柔軟性を阻害します。

### 改良されたコード

この問題を解決するための一つの解決策は、値オブジェクトとしての`RegularPrice`クラスを導入し、各割引クラスがこの値オブジェクトに依存するように設計を改良することです。このアプローチを採用することで、各クラスの責任が明確になり、拡張性や再利用性も向上します。

```php
<?php

namespace App\Discounts;

class RegularPrice
{
    private $price;

    public function __construct($price)
    {
        $this->price = $price;
    }

    public function calculateFinalPrice()
    {
        return $this->price;
    }
}

class RegularDiscount
{
    private $discountAmount = 300;

    public function calculateDiscountedPrice(RegularPrice $priceObj)
    {
        $price = $priceObj->calculateFinalPrice();
        $discountedPrice = $price - $this->discountAmount;

        return max($discountedPrice, 0);
    }
}

class SummerDiscount
{
    private $discountAmount = 400;

    public function calculateDiscountedPrice(RegularPrice $priceObj)
    {
        $price = $priceObj->calculateFinalPrice();
        $discountedPrice = $price - $this->discountAmount;

        return max($discountedPrice, 0);
    }
}
```

改良版のコードでは、割引額の管理と割引の適用が分離され、それぞれ`RegularDiscount`クラスと`SummerDiscount`クラスに一つずつの責任が与えられました。これにより単一責任の原則（SRP）が守られ、各クラスの役割が明確になりました。
また、`RegularDiscount`クラスと`SummerDiscount`クラスは、割引を適用する対象となる価格を引数として受け取ることで、具体的なクラスに依存しない設計になっています。これにより、依存性の逆転の原則（DIP）も守られ、割引額を変更するだけでなく、割引を適用する対象となる価格の計算方法を容易に変更することも可能となりました。

## 継承に絡む密結合

### 問題のあるコード

プログラミングにおける継承は、使用する際には注意が必要です。その理由は、継承が推奨されない場合が多いからです。以下に示す設計では`DiscountedProduct`が`Product`クラスを継承しています。`DiscountedProduct`は`Product`とほぼ同じ方法で動作しますが、割引が適用されている点が異なります。この設計により、継承による密結合が生じてしまいます。つまり、`Product`クラスに何らかの変更があると、そのすべての派生クラスも影響を受けることになります。

例えば以下のような PHP コードを考えてみましょう。

```php
class Product {
    protected $price;

    public function __construct($price)
    {
        $this->price = $price;
    }

    public function getPrice()
    {
        return $this->price;
    }
}

class DiscountedProduct extends Product {
    private $discount;

    public function __construct($price, $discount)
    {
        parent::__construct($price);
        $this->discount = $discount;
    }

    public function getPrice()
    {
        return $this->price - $this->discount;
    }
}
```

### 改良されたコード

この問題を解決する一つの方法は、継承よりも委譲による設計を選ぶことです。これは、コンポジション構造を採用するということになります。この設計では、各クラスが他のクラスのメソッドを利用することで、一つのクラスの変更が他の全てのクラスに影響を及ぼすという問題を避けることができます。

```php
class Product {
    protected $price;

    public function __construct($price)
    {
        $this->price = $price;
    }

    public function getPrice()
    {
        return $this->price;
    }
}

class Discount {
    private $discount;

    public function __construct($discount)
    {
        $this->discount = $discount;
    }

    public function applyDiscount($price)
    {
        return $price - $this->discount;
    }
}

class DiscountedProduct {
    private $product;
    private $discount;

    public function __construct(Product $product, Discount $discount)
    {
        $this->product = $product;
        $this->discount = $discount;
    }

    public function getPrice()
    {
        return $this->discount->applyDiscount($this->product->getPrice());
    }
}
```

以上のように設計を改良することで、各クラスの役割が明確になり、また一つのクラスが変更されても他のクラスに影響が出にくくなります。これにより、ソフトウェアの可読性と保守性が向上します。

##　スマート UI

### 問題のあるコード

スマート UI とは、ビジネスロジックが UI 層に散らばってしまい、結果として重複したコードが生じやすい設計を指します。そのような設計は、コードの管理や保守性を低下させ、バグの発生率を高めるリスクがあります。

```php
<!-- resources/views/orders/create.blade.php -->

<form method="POST" action="/orders">
    @csrf
    <input type="hidden" name="product_id" value="{{ $product->id }}">
    <input type="hidden" name="user_id" value="{{ $user->id }}">
    <input type="number" name="quantity" value="1">

    <p>価格: {{ $product->price * old('quantity', 1) }}円</p>

    @if ($user->country == 'Japan')
        <p>送料: 500円</p>
    @else
        <p>送料: 1000円</p>
    @endif

    @if ($user->membership == 'Gold')
        <p>割引適用後価格: {{ $product->price * old('quantity', 1) * 0.9 }}円</p>
    @endif

    <button type="submit">注文する</button>
</form>
```

上記の問題のあるコードでは、注文の価格計算、送料計算、割引適用といったビジネスロジックがビューファイル内に存在しています。これにより、もし価格計算方法や送料の設定、割引の適用ルールが変わった場合、ビューファイルを直接修正する必要があります。その結果、コードの再利用性が低下し、管理が難しくなります。

### 改良されたコード

```php
// App/Http/Controllers/OrderController.php

namespace App\Http\Controllers;

use App\Order;
use Illuminate\Http\Request;

class OrderController extends Controller
{
    public function create(Request $request)
    {
        $order = new Order;
        $order->product_id = $request->product_id;
        $order->user_id = $request->user_id;
        $order->quantity = $request->quantity;
        $order->price = $order->calculatePrice($request->quantity, $order->product->price);
        $order->shipping = $order->calculateShipping($request->user->country);
        $order->price = $order->applyDiscount($request->user->membership, $order->price);

        return view('orders.create', ['order' => $order]);
    }
}
```

```php

// resources/views/orders/create.blade.php

<form method="POST" action="/orders">
    @csrf
    <input type="hidden" name="product_id" value="{{ $order->product_id }}">
    <input type="hidden" name="user_id" value="{{ $order->user_id }}">
    <input type="number" name="quantity" value="{{ $order->quantity }}">

    <p>価格: {{ $order->price }}円</p>
    <p>送料: {{ $order->shipping }}円</p>

    @if ($order->user->membership == 'Gold')
        <p>割引適用後価格: {{ $order->price }}円</p>
    @endif

    <button type="submit">注文する</button>
</form>
```

以上のように改良したコードでは、`OrderController`内で価格計算、送料計算、割引適用といった処理を行っています。そして、ビューファイルではそれらの結果を表示するだけになっています。このようにすることで、ビジネスロジックが一箇所にまとまり、コードの再利用性と管理性が大幅に向上しています。また、ビジネスロジックが変更されても一箇所を修正すれば良いため、保守性も向上しています。

## YAGNI 原則

### 問題のあるコード

YAGNI(You Aren't Gonna Need It)原則は、現時点で必要でない機能は開発しないという原則です。これは、開発者が将来的に必要になると予想される機能を先に実装しようとすると、その機能が実際には使用されない場合や、開発者が想定していた要件が変更された場合に、無駄な労力を消費する可能性があるためです。

以下に、この YAGNI 原則が適用されていない例と、その問題点について見ていきましょう。

```php
namespace App\Http\Controllers;

use App\Product;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    public function index(Request $request)
    {
        // 今後、商品をカテゴリ別に表示するかもしれないと予測して、
        // カテゴリ別に商品を取得するコードを書いてしまっている
        if ($request->has('category')) {
            $products = Product::where('category', $request->category)->get();
        } else {
            $products = Product::all();
        }

        return view('products.index', ['products' => $products]);
    }
}
```

上記のコードでは、将来的にカテゴリ別に商品を表示する可能性があると想定し、そのためのコードを先に書いてしまっています。しかし、その機能が実際に必要になるまでは、このコードは必要ないと考えられます。

### 改良されたコード

```php
namespace App\Http\Controllers;

use App\Product;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    public function index(Request $request)
    {
        // 必要な機能だけを実装する
        $products = Product::all();

        return view('products.index', ['products' => $products]);
    }
}
```

解決策としては、現時点で必要な機能のみを実装します。つまり、全ての商品を取得するだけのコードを書きます。将来的にカテゴリ別に商品を表示する機能が必要になったとき、その時点で必要なコードを追加します。
このように、YAGNI 原則に基づく開発を行うことで、無駄なコードを書くことなく、現時点で本当に必要な機能のみを提供することができます。

## マジックナンバー

### 問題のあるコード

「マジックナンバー」とはソースコード内に直接書き込まれている意図不明な数値のことを指します。これは、他の開発者がその数値の意味や目的を理解しにくく、将来的な保守や変更が困難になる要因となります。

例として、次のコードを考えてみましょう。

```php
class UserController extends Controller
{
    public function index()
    {
        // ...

        // マジックナンバーを使用して一覧を取得
        $users = User::paginate(15);

        // ...
    }
}
```

上記のコードでは、一覧表示の際に 15 件のユーザー情報を取得していますが、この「15」という数値が何を意味しているのかはコードからは明らかではありません。

### 改良されたコード

前述のような問題点を踏まえて、マジックナンバーを適切に扱う方法を紹介します。このような問題を解決するための一般的なアプローチは、マジックナンバーを定数に置き換えてその意味を明示することです。以下はその解決策を示す例です。

```php
class UserController extends Controller
{
    // 定数として一覧表示件数を定義
    const USERS_PER_PAGE = 15;

    public function index()
    {
        // ...

        // 定数を使用して一覧を取得
        $users = User::paginate(self::USERS_PER_PAGE);

        // ...
    }
}
```

このように、定数`USERS_PER_PAGE`を使用することで、この数字が一覧表示の件数を示していることが明確になり、コードの可読性が向上します。他の開発者もこの定数の意味を理解しやすくなり、将来的な変更や保守も容易になります。

## 悪魔を呼び寄せる名前

### 問題のあるコード

プログラミングにおいて、クラス名やメソッド名はその機能や役割を明確に表すものであるべきです。しかし、場合によっては名前からその役割や機能が読み取りにくい、あるいは複数の役割を持つクラスを作成してしまうことがあります。これはコードの可読性を低下させ、メンテナンスや拡張の難しさを増大させる原因となります。

```php
namespace App;

use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
    // ...

    public function reserve()
    {
        // 商品を予約する
    }

    public function order()
    {
        // 商品を注文する
    }

    public function ship()
    {
        // 商品を発送する
    }

    public function list()
    {
        // 商品を出品する
    }
}
```

上記のコードでは、`Product`クラスが商品の予約、注文、発送、出品の 4 つの異なる責任を持っています。これはオブジェクト指向の原則の 1 つである「単一責任の原則」に違反しており、各機能の変更時やテスト時に問題が生じる可能性があります。

### 改良されたコード

これらの責任を持つクラスをそれぞれ分けることで、各クラスの責任を明確にし、保守性や拡張性を向上させることができます。

```php
namespace App;

use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
    // ...
}

class ReservationProduct
{
    public function create(Product $product)
    {
        // 商品を予約する
    }
}

class OrderProduct
{
    public function place(Product $product)
    {
        // 商品を注文する
    }
}

class ShipmentProduct
{
    public function execute(Product $product)
    {
        // 商品を発送する
    }
}

class ListingProduct
{
    public function publish(Product $product)
    {
        // 商品を出品する
    }
}
```

このようにクラスを再編成することで、それぞれのクラスが持つ責任が明確になり、他の開発者も理解しやすくなります。また、将来的な機能の追加や変更も柔軟に行うことができるようになります。

## 形容詞で区別が必要なときはクラス化のチャンス

### 問題のあるコード

オブジェクト指向プログラミングにおいて、あるクラスが多様な状態やタイプを持っている場合、そのクラスの責任は多すぎる可能性が高まります。特に、それらの状態やタイプに基づいて異なる処理が行われる場合、クラスの可読性や保守性が低下する危険があります。

一つのクラスや関数が異なる状態やタイプを扱っているとき、その状態やタイプを表す形容詞が存在し、それに基づいて処理が別れている場合、それぞれを別クラスに分けるべきだという原則です。

```php
class Car {
    private $type;

    public function __construct($type) {
        $this->type = $type;
    }

    public function getTopSpeed() {
        if($this->type === "sport") {
            return 200;
        } elseif($this->type === "truck") {
            return 80;
        }
    }
}
```

### 改良されたコード

このコードでは、`Car`クラスが`sport`や`truck`などの形容詞に基づいて動作を変えています。このような場合、異なる振る舞いを持つオブジェクトは別々のクラスに分けるべきです。

```php
abstract class Car {
    abstract public function getTopSpeed();
}

class SportCar extends Car {
    public function getTopSpeed() {
        return 200;
    }
}

class TruckCar extends Car {
    public function getTopSpeed() {
        return 80;
    }
}
```

## コメントで命名をごまかす

### 問題のあるコード

プログラムの保守性や可読性を高めるためには、適切な命名が非常に重要です。名前だけで機能や目的を理解できるコードは、他の開発者が読んだり、将来の自分が読み返したりする際に大きな助けとなります。

```php
class ProductController {
    //...

    // PIDチェック: PIDが存在しなければfalseを返す
    public function check($pid) {
        $product = Product::find($pid);
        return $product ? true : false;
    }

    //...
}
```

このコードでは、メソッド名 `check`が何を示しているのかが一目でわからないため、コメントを頼りにしています。しかし、このようなアプローチは、長期的に見ると保守性の低下や混乱の原因となる可能性があります。

### 改良されたコード

コメントに依存するような名前付けは、そのコードが自己説明的でないことを示しており、保守性を下げる可能性があります。
そこで以下のように命名を見直します。

```php
class ProductController {
    //...

    public function isProductExists($productId) {
        $product = Product::find($productId);
        return $product ? true : false;
    }

    //...
}
```

改良例では、メソッド名を`isProductExists`として、具体的に何を確認するのかを名前だけで伝えるようにしました。これにより、コメントに頼らずともその目的や機能を理解することが可能となります。

## 意図や仕様変更時の注意点を読み手に伝えること

### 問題のあるコード

プログラムの可読性を保つためには、コードの背後にある意図や仕様の変更点に関する注意をコメントとして伝えることが重要です。特に、他の開発者が後にそのコードを参照する場合、明確なガイダンスは作業の効率化や誤解の防止に役立ちます。

```php
class ProductController extends Controller
{
    /**
     * 商品に割引を適用する。
     *
     * @param  int  $productId
     * @param  float  $discount
     * @return Response
     */
    public function applyDiscount($productId, $discount)
    {
        $discount = min($discount, 50);

        $product = Product::find($productId);

        $product->price -= $product->price * ($discount / 100);
        $product->save();

        return response()->json($product);
    }
}
```

現在のコードでは、割引の最大限度に関するビジネスルールが明示されていません。このような状況では、他の開発者がコードを見たときに混乱を引き起こす可能性があります。

### 改良されたコード

現状だと、 `$discount = min($discount, 50);` の真意がわかりません。そこで、`applyDiscount`メソッドにコメントを付けています。このコメントはメソッドの目的を明確にし、パラメータの説明を提供し、さらに重要なビジネスルール（割引は 50%を超えてはならない）を指摘しています。これにより、他の開発者がこのメソッドを使用または変更する際に、その挙動と制限を正しく理解できます。

```php
class ProductController extends Controller
{
    /**
     * 商品に割引を適用する。
     *
     * @param  int  $productId
     * @param  float  $discount
     * @return Response
     */
    public function applyDiscount($productId, $discount)
    {
        // 商品が不採算になるため、割引率は50%を超えないようにしてください。
				// 割引率が50%以上の場合は、50%に設定されます。
        $discount = min($discount, 50);

        $product = Product::find($productId);

        // 割引を適用する
        $product->price -= $product->price * ($discount / 100);
        $product->save();

        return response()->json($product);
    }
}
```

このように、コメントとして意図や仕様変更時の注意点を書くと、読み手により明確なガイダンスを提供することができます。

## 必ず自身のクラスのインスタンス変数を使うこと

### 問題のあるコード

クラスの設計時には、そのクラスが持つべき情報をインスタンス変数として適切に管理することが重要です。この方法は、クラスのコンセプトや設計の一貫性を保つため、また外部からの不適切なアクセスを避けるために役立ちます。

```php
class Rectangle {
    private $width;
    private $height;

    public function setWidth($width) {
        $this->width = $width;
    }

    public function setHeight($height) {
        $this->height = $height;
    }

    public function getArea($width, $height) {
        return $width * $height;
    }
}

$rectangle = new Rectangle;
$rectangle->setWidth(5);
$rectangle->setHeight(4);
echo $rectangle->getArea(5, 4);  // 20
```

上記のコードでは、`getArea`メソッドが幅と高さを引数として受け取るようになっています。このアプローチは、クラス内部でその情報を適切に管理するという原則に反しています。

### 改良されたコード

`getArea`メソッドが幅と高さを直接使用するように改良すれば、より適切なオブジェクト指向の設計を持つことができます。以下はその修正後のコードです。

```php
class Rectangle {
    private $width;
    private $height;

    public function setWidth($width) {
        $this->width = $width;
    }

    public function setHeight($height) {
        $this->height = $height;
    }

    public function getArea() {
        return $this->width * $this->height;
    }
}

$rectangle = new Rectangle;
$rectangle->setWidth(5);
$rectangle->setHeight(4);
echo $rectangle->getArea();  // 20
```

この改良を行うことで、クラスが持つべき情報を正しく内部で管理することが可能になり、外部からの不要な干渉や誤った情報の供給を防ぐことができます。また、このような設計はクラスの再利用や拡張も容易にし、長期的な保守性を向上させます。

## 不変をベースに予期せぬ動作を防ぐ関数にすること

### 問題のあるコード

多くのプログラミング言語やフレームワークにおいて、データの不変性は重要な概念として取り扱われています。不変性を持たせることで、データが予期せぬタイミングや方法で変更されるのを防ぐことができます。

```php
class Product {
    public $price;

    public function setPrice($price) {
        $this->price = $price;
    }
}

$product = new Product;
$product->setPrice(100);

$product->price = 50;  // price is changed after it was set
```

このコードでは、`$product->price`の価格を一度設定した後でも、外部から直接変更が可能となっています。

### 改良されたコード

PHP 8.1 以降、`readonly`修飾子を用いることでプロパティを不変に保つことが可能です。`readonly`修飾子が付与されたプロパティは、オブジェクトが初期化される際のみ値を設定することができ、その後の変更は許されません。

```php
class Product {
    public readonly int $price;

    public function __construct(int $price) {
        $this->price = $price;
    }
}

$product = new Product(100);

$product->price = 50;  // This will cause an error
```

このコードでは、`$price`プロパティに`readonly`修飾子を付けて、一度設定されたらその後変更できないようにしています。これにより`$price`は不変となり、予期せぬ価格変更を防ぐことができます。

## コマンド・クエリ分離

### 問題のあるコード

コマンド・クエリ分離（Command Query Separation, CQS）はオブジェクト指向設計の原則の一つであり、メソッドが状態を変更するコマンドであるか、情報を返すクエリであるかのどちらか一方のみを行うべきだという考え方です。

しかし、現実のコード上ではこの原則が破られることがしばしばあります。

以下のコードがその例です。

```php
class User extends Model {
    public function getOrCreateToken() {
        $token = $this->tokens()->first();

        if (!$token) {
            $token = $this->tokens()->create([
                'token' => bin2hex(random_bytes(30)),
            ]);
        }

        return $token;
    }
}
```

この`getOrCreateToken`メソッドは、コマンド（トークンを作成する）とクエリ（既存のトークンを取得する）の両方の動作を一つのメソッド内で行っています。このようなメソッドは、状態を変更するかどうかがその実行結果によって変わるため、予測が難しく、結果としてバグの原因となりやすいです。

### 改良されたコード

この問題を解決する最もシンプルな方法は、コマンドとクエリを明確に分離することです。以下のようにメソッドを分けることで、各メソッドが一つの責任だけを持つように設計することができます。

```php
$token = $user->getToken();

if (!$token) {
    $token = $user->createToken();
}
```

```php
class User extends Model {
    public function getToken() {
        return $this->tokens()->first();
    }

    public function createToken() {
        return $this->tokens()->create([
            'token' => bin2hex(random_bytes(30)),
        ]);
    }
}
```

この改良により、各メソッドはコマンドまたはクエリのどちらか一方だけを行うようになり、コマンド・クエリ分離の原則に則った設計となります。このような設計は、コードの可読性と予測性を向上させ、結果としてバグの発生を減少させる効果が期待できます。

## 邪悪な構造に陥りがちな User クラス

### 問題のあるコード

Laravel におけるクラスの"邪悪な構造"とは何でしょうか？ その答えの一つは、あるクラスが大量のインスタンス変数やインスタンス関数を持つ場合、それが該当する可能性が高いです。これはクラスが多すぎる責任を持っているという意味で、これは「単一責任の原則」（SRP）に違反しています。この原則は、ソフトウェアの設計原則の一つで、各クラスは 1 つの責任だけを持つべきだと提唱されています。

"邪悪な構造"の具体例として、User クラスが多数のインスタンス変数とインスタンス関数を持つ場合を見てみましょう。

```php
class User extends Authenticatable
{
    protected $fillable = ['name', 'email', 'password', 'birthdate', 'phone', 'address', 'avatar', 'isAdmin', 'isPremiumMember', 'biography'];

    public function calculateAge()
    {
        // 何かの処理
    }

    public function validateEmail()
    {
        // 何かの処理
    }

    public function validatePhone()
    {
        // 何かの処理
    }

    public function sendVerificationEmail()
    {
        // 何かの処理
    }

    public function changePassword()
    {
        // 何かの処理
    }

    public function updateAvatar()
    {
        // 何かの処理
    }

    public function upgradeMembership()
    {
        // 何かの処理
    }

    public function downgradeMembership()
    {
        // 何かの処理
    }
    // その他多数のメソッド...
}
```

この User クラスは、ユーザー関連のさまざまな責務（例：バリデーション、電子メールの送信、パスワードの変更など）を一手に担っています。この結果、クラスが過大になり、コードの可読性やメンテナンス性、そして再利用性が低下するリスクが増えます。

### 改良されたコード

このような問題の解決策の一つは、特定の役割ごとにクラスを細分化することです。これにより、各クラスはその名の通り、単一の責務を持つようになります。

```php
class User extends Authenticatable
{
    protected $fillable = ['name', 'email', 'password', 'birthdate', 'phone', 'address', 'avatar'];

    public function changePassword()
    {
        // 何かの処理
    }

    public function updateAvatar()
    {
        // 何かの処理
    }
}

class UserValidator
{
    public function validateEmail($email)
    {
        // 何かの処理
    }

    public function validatePhone($phone)
    {
        // 何かの処理
    }
}

class UserNotifier
{
    public function sendVerificationEmail($email)
    {
        // 何かの処理
    }
}

class MembershipManager
{
    public function upgradeMembership($user)
    {
        // 何かの処理
    }

    public function downgradeMembership($user)
    {
        // 何かの処理
    }
}
```

上記のリファクタリング例において、User クラスはユーザーの基本情報とそれに関わる基本的な動作を担当しています。一方で、バリデーションや通知の送信、そしてメンバーシップの管理などの専門的なタスクは、それぞれ専門のクラスが担当しています。

このような分割によって、各クラスの役割が明確になり、コードの整理や再利用が容易になります。また、将来的な変更や機能の追加もスムーズに行えるようになります。
