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

## Controller Middleware

[Middleware](http://laravel.com/docs/5.1/middleware)はこのようなコントローラーのルートに割り当てられるかもしれません。
```php
Route::get('profile', [
    'middleware' => 'auth',
    'uses' => 'UserController@showProfile'
]);
```
しかしながら、コントローラーのコンストラクタ内でミドルウェアを指定するのがもっと便利です。コントローラーのコンストラクタから`middleware`メソッドを使うことで、簡単にミドルウェアをコントローラーに割り当てられます。ミドルウェアをコントローラークラス上の一つの確かなメソッドに制限することでさえできます。
```php
class UserController extends Controller
{
    /**
    * Instantiate a new UserController instance.
    *
    * @return void
    */
    public function __construct()
    {
        $this->middleware('auth');

        $this->middleware('log', ['only' => ['fooAction', 'barAction']]);
        $this->middleware('subscribed', ['except' => ['fooAction', 'barAction']]);
    }
}
```

## RESTful Resource Controllers

リソースコントローラーはリソース周りのRESTfulコントローラーをビルドするのが困難ではありません。例えば、アプリケーションによって蓄えられる"photos"に関してのHTTPリクエストを扱うコントローラを生成したいと望んでいるかもしれません。`make:controller`Artisanコマンドを使うことで、素早くこのようなコントローラーを生成できます。
```php
php artisan make:controller PhotoController
```
Artisanコマンドは`app/Http/Controllers/PhotoController.php`にコントローラーファイルを生成します。コントローラーは利用可能なリソース操作のそれぞれのメソッドを含みます。
次に、リソースフルなルートをコントローラーに登録します。
```php
Route::resource('photo', 'PhotoController');
```
このシングルルート宣言は、photoリソース上の様々なRESTfullアクションを扱う複数のルートを生成します。同様に、生成されたコントローラーはそれらそれぞれのスタブメソッドを持ち、それらがどのURIや動詞を扱うかを知らせる注釈も含むでしょう。

#### Actions Handled By Resource Controller

| Verb      | Path                  | Action  | Route Name    |
|:----------|:----------------------|:--------|:--------------|
| GET       | `/photo`              | index   | photo.index   |
| GET       | `/photo/create`       | create  | photo.create  |
| POST      | `/photo`              | store   | photo.store   |
| GET       | `/photo/{photo}`      | show    | photo.show    |
| GET       | `/photo/{photo}/edit` | edit    | photo.edit    |
| PUT/PATCH | `/photo/{photo}`      | update  | photo.update  |
| DELETE    | `/photo/{photo}`      | destory | photo.destory |

#### Partial Resource Routes

リソースルートを宣言するとき、ルートを扱うためにアクションのサブセットを指定できます。
```php
Route::resource('photo', 'PhotoController',
                ['only' => ['index', 'show']]);

Route::resource('photo', 'PhotoController',
                ['except' => ['create', 'store', 'update', 'destroy']]);
```

#### Naming Resource Routes

デフォルトで、全てのリソースコントローラーアクションはルート名を持ちます。しかしながら、オプションで`name`配列を渡すことでそれらの名前をオーバーライド出来ます。
```php
Route::resource('photo', 'PhotoController',
                ['names' => ['create' => 'photo.build']]);
```

#### Nested Resources

ときどき、"ネストした"リソースのためのルートを定義する必要があるかもしれません。例えば、photoリソースはphotoに付属するかも知れない複数の"コンポーネント"を持つかもしれません。リソースコントローラーを"ネスト"させるために、ルート宣言の中で"dot"表記を使います。
```php
Route::resource('photos.comments', 'PhotoCommentController');
```
このルートは次のようなURL:`photos/{photos}/comments/{comments}`でアクセスされるかもしれない"ネストした"リソースを登録します。
```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;

class PhotoCommentController extends Controller
{
    /**
    * Show the specified photo comment.
    *
    * @param  int  $photoId
    * @param  int  $commentId
    * @return Response
    */
    public function show($photoId, $commentId)
    {
        //
    }
}
```

#### Supplementing Resource Controllers

もしデフォルトのリソースルート以上にリソースコントローラーへルートを追加する必要が出てきた場合、`Route::resource`を呼ぶ前にそれらを定義すべきです。さもなければ、`resource`メソッドによって定義されたルートは意図せずにあなたの追加したルートに優先してとられるかもしれません。
```php
Route::get('photos/popular', 'PhotoController@method');

Route::resource('photos', 'PhotoController');
```

## Implicit Controllers

Laravelはコントローラークラス内の全てのアクションを操作するシングルルートを簡単に定義できるようにしています。最初に、`Route::controller`メソッドを使ってルートを定義します。`controller`メソッドは2つの引数を受け入れます。第一引数はコントローラーが操作するベースURIで、第二引数はコントローラーのクラス名です。
```php
Route::controller('users', 'UserController');
```
次に、コントローラーにメソッドを追加するだけです。メソッドネームは応答するHTTP動詞から始まり、続いてURIタイトルケースバージョンが続くべきです。
```php
<?php

namespace App\Http\Controllers;

class UserController extends Controller
{
    /**
    * Responds to requests to GET /users
    */
    public function getIndex()
    {
        //
    }

    /**
    * Responds to requests to GET /users/show/1
    */
    public function getShow($id)
    {
        //
    }

    /**
    * Responds to requests to GET /users/admin-profile
    */
    public function getAdminProfile()
    {
        //
    }

    /**
    * Responds to requests to POST /users/profile
    */
    public function postProfile()
    {
        //
    }
}
```
上の例を見ると分かるように、`index`メソッドはコントローラーによって操作されるルートURI、このケースでは`users`に対応します。

#### Assigning Route Names

もしコントローラー上のルートのいくつかを[名前付けたい](http://laravel.com/docs/5.1/routing#named-routes)場合、`controller`絵ソッドの第三引数として名前の配列を渡せばよいです。

```php
Route::controller('users', 'UserController', [
    'getShow' => 'user.show',
]);
```
