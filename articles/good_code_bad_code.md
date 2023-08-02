---
title: "「良いコード/悪いコードで学ぶ設計入門」の理解をPHP(Laravel)で深めよか"
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Laravel", "設計", "品質", "ミノ駆動本"]
published: false
---

# TL;DR

- 「良いコード/悪いコードで学ぶ設計入門」をテーマに、PHP(Laravel)を使用してプログラミング設計原則を詳解
- 各トピックごとに「問題のコード」例と「改良されたコード」例を PHP(Laravel)で提供し、良い設計原則に従う方法を具体的に提示
- これを読めば、設計原則の理解が深まり自身の開発に活かせるはず
- もちろん全トピックを扱うわけではないので、気になった人はぜひ購入を

https://gihyo.jp/book/2022/978-4-297-12783-1

# 第 1 章 悪しき構造の弊害を知覚する

## 1. 理解を困難にする条件分岐のネスト

### 問題のコード

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

問題のコードを見ると、二つの条件、ユーザーが存在するかどうか、そしてユーザーがアクティブかどうかという条件がネストしています。

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

## 2. さまざまな悪魔を招きやすいデータクラス

### 問題のコード

データクラスとは、データの格納と取得のみを行うクラスを指します。これらのクラスがビジネスロジックを持たないために起こる問題は、そのデータを操作するためのロジックがクラス外部に散在してしまう可能性があるという点です。

例えば、以下のようなコードを考えてみましょう。

```php
class User {
    public $name;
    public $email;
}

class UserService {
    public function changeEmail(User $user, $newEmail) {
        // メールアドレスの形式検証
        if (!filter_var($newEmail, FILTER_VALIDATE_EMAIL)) {
            throw new Exception('Invalid email format');
        }

        $user->email = $newEmail;
    }
}
```

### 改良されたコード

このコードでは、`User`クラスは単にデータを保持するだけのデータクラスで、それを操作するためのロジックは`UserService`クラスに存在します。これにより、`User`クラスがそのデータの整合性を自身で保証することができなくなり、全ての操作が`UserService`を通じて行われることが必ずしも保証されていないため、データの不整合が発生するリスクがあります。

これらの問題を解決するためには、開発者がデータとそのデータを操作するロジックを一緒に持つようにクラスを設計する必要があります。

```php
class User {
    public $name;
    public $email;

    public function __construct($name, $email) {
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new Exception('Invalid email format');
        }
        $this->name = $name;
        $this->email = $email;
    }

    public function changeEmail($newEmail) {
        if (!filter_var($newEmail, FILTER_VALIDATE_EMAIL)) {
            throw new Exception('Invalid email format');
        }
        $this->email = $newEmail;
    }
}
```

## 3. 未初期化状態（生焼けオブジェクト）

### 問題のコード

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

## 4. 不正値の混入

### 問題のコード

不正値の混入とは、制約を満たさない値がプログラム中に混入し、その結果、エラーを引き起こしたり、予期しない動作を引き起こしたりする状況を指します。

以下のようなコードを例に考えてみましょう。

```php
class User {
    public $name;
    public $age;

    public function __construct($name, $age) {
        $this->name = $name;
        $this->age = $age;
    }
}

$user = new User('John Doe', -1);
```

### 改良されたコード

上記のコードでは、年齢がマイナスの値となってしまいます。このような値は、一般的なビジネスロジックでは許容されません。

この問題を解決するために、クラスに不正な値が設定されないようにバリデーションを追加します。

```php
class User {
    public $name;
    public $age;

    public function __construct($name, $age) {
        if ($age < 0) {
            throw new InvalidArgumentException('Age cannot be negative');
        }
        $this->name = $name;
        $this->age = $age;
    }
}

try {
    $user = new User('John Doe', -1);
} catch (InvalidArgumentException $e) {
    echo $e->getMessage();
}
```

この改良版のコードでは、コンストラクタが年齢の値がマイナスになることを防ぐバリデーションを行っています。これにより、年齢として不適切な値が`User`クラスに設定されることを防ぎます。

# 第 2 章　設計の初歩

## 1. ベタ書きせず、意味のあるまとまりでメソッド化

### 問題のコード

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

## 2. 関係し合うデータとロジックをクラスにまとめる

### 問題のコード

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

## 1. クラス単体で正常に動作するよう設計する

### 問題のコード

各クラスは単体で正常に動作するように設計することが重要です。そうしなければ、そのクラスは他のクラスと深く結びついてしまい、再利用やテストが難しくなります。また、クラスが他のクラスに依存している場合、その依存関係を理解しなければクラスの動作を完全に理解することは難しくなります。これにより、新たな開発者がコードを理解するのが難しくなるだけでなく、未来の自分がコードを理解するのも難しくなる可能性があります。

以下にそのようなコードを示します。

```php
use App\Services\UserService;

class UserController extends Controller
{
    public function index()
    {
        $userService = new UserService();
        $users = $userService->getAllUsers();

        return view('user.index', compact('users'));
    }
}
```

上記のコードでは、`UserController`が直接`UserService`の新しいインスタンスを生成しています。これにより、テストをする際にモックオブジェクトを使って`UserService`を置き換えることが難しくなり、`UserController`が`UserService`と密結合になってしまいます。

### 改良されたコード

```php
use App\Services\UserService;

class UserController extends Controller
{
    protected $userService;

    public function __construct(UserService $userService)
    {
        $this->userService = $userService;
    }

    public function index()
    {
        $users = $this->userService->getAllUsers();

        return view('user.index', compact('users'));
    }
}
```

改良後のコードでは、依存性の注入を利用して`UserService`を`UserController`に提供します。これにより、`UserController`が`UserService`の具体的な実装に依存しなくなり、コードのテストやメンテナンスが容易になります。また、異なる`UserService`の実装を容易に切り替えることも可能になります。
このように、クラスはそれ自体で完結するよう設計し、依存性を外部から注入することで、テスト性と保守性を向上させることができます。

## 2. 成熟したクラスへ成長させる設計術

### 問題のコード

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

## 1. 可変がもたらす意図せぬ影響

### 問題のコード

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
