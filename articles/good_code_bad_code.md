---
title: "「良いコード/悪いコードで学ぶ設計入門」の理解をPHP(Laravel)で深めよか"
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Laravel", "設計", "品質", "ミノ駆動本"]
published: false
---

# TL;DR

- 「良いコード/悪いコードで学ぶ設計入門」をテーマに、PHP(Laravel)を使用してプログラミング設計原則を詳解
- 各トピックごとに「問題のコード」例と「改良されたコード」例を PHP(Laravel)で提供し、良い設計原則に従う方法を具体的に示す
- これを読めば、設計原則の理解が深まり自身の開発に活かせるはず

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

この `getUserFromDB` メソッドは、その名前がデータベースからユーザーを取得するという実装詳細を含んでいます。しかし、メソッドの主な目的は、特定のユーザーの詳細を表示することです。したがって、このような場合には以下のように名前を変更すべきです。

```php
class UserController extends Controller {
    public function show($id) {
        $user = User::find($id);
        return view('user.show', ['user' => $user]);
    }
}
```

この改良されたコードでは、 `show` という名前がビジネスロジックをより直接的に反映しています。
