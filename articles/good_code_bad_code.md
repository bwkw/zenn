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

## 1. 意味不明な命名

### 問題のコード

メソッドやクラスの命名においては、技術的な詳細や使用している技術そのものを表す名前（「技術駆動命名(Technology-driven naming)」という）よりも、ビジネスロジックを反映した名前を使用すべきという考え方があります。

以下に、技術駆動命名のコード例を示します。

```php
class UserController extends Controller {
    public function getUserFromDB($id) {
        $user = User::find($id);
        return view('user.show', ['user' => $user]);
    }
}
```

### 改良されたコード

この`getUserFromDB`メソッドは、その名前がデータベースからユーザーを取得するという実装詳細を含んでいます。しかし、メソッドの主な目的は、特定のユーザーの詳細を表示することです。したがって、このような場合には以下のように名前を変更することが望ましいです。

```php
class UserController extends Controller {
    public function show($id) {
        $user = User::find($id);
        return view('user.show', ['user' => $user]);
    }
}
```

この改良されたコードでは、`show`という名前がビジネスロジックをより直接的に反映しています。

## 2. 理解を困難にする条件分岐のネスト

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

## 3. さまざまな悪魔を招きやすいデータクラス

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

## 4. 未初期化状態（生焼けオブジェクト）

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

## 5. 不正値の混入

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
    public function index()
    {
        $users = User::all();
        foreach ($users as $user) {
            $user->age = Carbon::createFromFormat('Y-m-d', $user->birthdate)->diff(Carbon::now())->format('%y');
        }
        return view('user.index', compact('users'));
    }
}
```

このコードでは、全てのユーザーの年齢を計算しています。しかし、この年齢計算のロジックが直接 index メソッド内に記述されているため、その意図を理解するのが難しく、再利用も困難です。

```php
class UserController extends Controller
{
    public function index()
    {
        $users = User::all();
        foreach ($users as $user) {
            $user->age = $this->calculateAge($user->birthdate);
        }
        return view('user.index', compact('users'));
    }

    // 本来、Controllerに実装するものではない
    private function calculateAge($birthdate)
    {
        return Carbon::createFromFormat('Y-m-d', $birthdate)->diff(Carbon::now())->format('%y');
    }
}
```

改善後のコードでは、年齢計算のロジックを`calculateAge`メソッドに切り出しました。これにより、コードの意図がより明確になり、再利用も容易になりました。また、このメソッドは個別にテストすることも可能になります。

このように、適切にメソッド化を行うことでコードの可読性と保守性を向上させ、再利用やテストも容易になります。
