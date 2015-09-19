## Introduction

一つの`routes.php`内で全てのリクエストのロジック操作の定義をする代わりに、これらの振る舞いをコントローラークラスを使うことにまとめてもよいです。コントローラーは関連するHTTPリクエスト操作ロジックを一つのクラスに集めることができます。コントローラーは典型的に`app/HttpControllers`ディレクトリに格納されます。

## Basic Controllers

ベーシックコントローラークラスのサンプルがあります。全てのLaravelのコントローラーは、デフォルトLaravelに含まれるベースコントローラークラスを拡張すべきです。

```php
<?php

namespace App\Http\Controllers;

use App\User;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
    * Show the profile for the given user.
    *
    * @param  int  $id
    * @return Response
    */
    public function showProfile($id)
    {
        return view('user.profile', ['user' => User::findOrFail($id)]);
    }
}
```

このようにコントローラーアクションにルーティングできます。

```php
Route::get('user/{id}', 'UserController@showProfile');
```

今、リクエストが特定のルートURLにマッチした時、`UserController`クラス上の`showProfile`メソッドは実行されます。もちろん、ルートパラメータはメソッドに渡されます。

#### Controllers & Namespaces

コントローラーのルートを定義するとき、全コントローラー名前空間を明記する必要がないことに注意するのはとても重要です。名前空間の"root"である`App\Http\Controllers`の後にくるクラス名の一部だけ定義すれば良いです。デフォルトで、`RouteServiceProvider`は、ルートコントローラー名前空間を含んでいるルートグループ内の`routes.php`ファイルをロードします。

もしネストさせるもしくはより深いPHP名前空間を使っているコントローラーを`App\Http\Controllers`ディレクトリ内にまとめることを選ぶ場合、`App\Http\Controllers`ルート名前空間と比較して、簡単に特定のクラス名を使うことができます。だから、もしフルコントローラークラスが`App\Http\Controllers\Photos\AdminController`である場合、このようにルートを登録してもよいです。
```php
Route::get('foo', 'Photos\AdminController@method');
```

#### Naming Controller Routes

クロージャルートのように、コントローラールート上の名前を指定したいかもしれません。
```php
Route::get('foo', ['uses' => 'FooController@method', 'as' => 'name']);
```
一度名前をコントローラールートに割り当てたら、アクションに対して簡単にURLを生成することが出来ます。コントローラーアクションに対してURLを生成するために、`action`ヘルパーメソッドを使います。もう一度、ベースの`App\Http\Controllers`名前空間の後に来るコントローラークラス名の部分を指定だけ必要です。
```php
$url = action('FooController@method');
```
名前付けられたコントローラールートへのURLを生成するために、`route`ヘルパーも使えます。
```php
$url = route('name');
```

