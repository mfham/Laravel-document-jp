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

