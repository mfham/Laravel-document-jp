## Accessing The Request

依存性の注入経由で現在のHTTPリクエストのインスタンスを得るために、コントローラーコンストラクタもしくはメソッド上の`Illuminate\Http\Request`クラスをタイプヒントすべきです。現在のリクエストインスタンスは[サービスコンテナ](http://laravel.com/docs/5.1/container)によって自動的に注入されるでしょう。
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Routing\Controller;

class UserController extends Controller
{
    /**
    * Store a new user.
    *
    * @param  Request  $request
    * @return Response
    */
    public function store(Request $request)
    {
        $name = $request->input('name');

        //
    }
}
```
もしコントローラーメソッドがルートパラメーターからの入力も期待している場合、他の依存性の後にルート引数を簡単に並べます。例えば、ルートが次のように定義されている場合、

```php
Route::put('user/{id}', 'UserController@update');
```
`Illuminate\Http\Request`をタイプヒントしたい、また次のようにコントローラーメソッドを定義することによってルートパラメーター`id`へアクセスしたいかもしれません。

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Routing\Controller;

class UserController extends Controller
{
    /**
    * Update the specified user.
    *
    * @param  Request  $request
    * @param  int  $id
    * @return Response
    */
    public function update(Request $request, $id)
    {
        //
    }
}
```

## Basic Request Information

`Illuminate\Http\Request`インスタンスはアプリケーションに対してHTTPリクエストを調査する様々なメソッドを提供します。Laravelの`Illuminate\Http\Request`は`Symfony\Component\HttpFoundation\Request`を拡張します。このクラスで利用できる2,3以上の役に立つメソッドがあります。

#### Retrieving The Request URI

`path`メソッドはリクエストのURIを返します。もし入ってきたリクエストが`http://domain.com/foo/bar`にアクセスしたら、`path`メソッドは`foo/bar`を返すでしょう。
```php
$uri = $request->path();
```
`is`メソッドは入ってきたリクエストURIが与えられたパターンにマッチするか確かめるのを許可します。このメソッドを使うとき、ワイルドカードとして`*`文字を使っても良いです。
```php
if ($request->is('admin/*')) {
    //
}
```
単なるパスの情報ではなくフルURLを取得するためには、リクエストインスタンスに対して`url`メソッドを使えば良いです。
```php
$url = $request->url();
```

#### Retrieving The Request Method

`method`メソッドはリクエストに対してHTTP動詞を返します。HTTP動詞が与えられた文字列にマッチするか確かめるために`isMethod`メソッドも使うことが出来ます。
```php
$method = $request->method();

if ($request->isMethod('post')) {
    //
}
```

## PSR-7 Requests

PSR-7基準はリクエストとレスポンスを含むHTTPメッセージのためのインターフェースを明記しています。もしPSR-7リクエストのインスタンスを得たい場合、最初にいくつかのライブラリをインストールする必要があるでしょう。Laravelは典型的なLaravelのリクエストとレスポンスをPSR-7互換性の実装に変換するためにSymfonyのHTTPメッセージブリッジコンポーネントを使います。
```php
composer require symfony/psr-http-message-bridge

composer require zendframework/zend-diactoros
```
いったんそれらのライブラリをインストールしたら、ルートやコントローラに対して簡単にリクエストタイプをタイプヒントすることによってPSR-7リクエストを得られるかもしれません。
```php
use Psr\Http\Message\ServerRequestInterface;

Route::get('/', function (ServerRequestInterface $request) {
    //
});
```
もしルートやコントローラーからPSR-7レスポンスインスタンスを返す場合、フレームワークによって自動的にLaravelレスポンスインスタンスへ変換され表示されます。

## Retrieving Input

#### Retrieving An Input Value

いくつか簡単なメソッドを使うとき、`Illuminate\Http\Request`インスタンスから全てのユーザー入力へアクセスできます。入力は全ての動詞に対して同じ方法でアクセスされるので、リクエストに対して使われるHTTP動詞を気にする必要はありません。
```php
$name = $request->input('name');
```
`input`メソッドに第二引数としてデフォルト値を渡します。この値はもしリクエストされた入力値がリクエストで現れなかった場合に返されます。
```php
$name = $request->input('name', 'Sally');
```
配列入力のフォームで動作している場合、配列へのアクセスに"ドット"記法が使えます。
```php
$input = $request->input('products.0.name');
```

#### Determining If An Input Value Is Present

値がリクエストに存在するか決定するために、`has`メソッドを使います。もし値が存在していて__かつ__空文字でない場合、`has`メソッドは`true`を返します。
```php
if ($request->has('name')) {
    //
}
```

#### Retrieving All Input Data

`all`メソッドを使うことで配列のように入力データの全てを得ることも出来ます。
```php
$input = $request->all();
```

#### Retrieving A Portion Of The Input Data

もし入力データのサブセットを得る必要がある場合、`only`メソッドと`except`メソッドを使うことができます。どちらのメソッドも、シングル配列もしくは動的な引数のリストを受け入れます。
```php
$input = $request->only(['username', 'password']);

$input = $request->only('username', 'password');

$input = $request->except(['credit_card']);

$input = $request->except('credit_card');
```

## Old Input

Laravelは次のリクエストの間1つのリクエストからの入力を保つのを許可します。この特徴はバリデーションエラーを見つけた後の再登録フォームに特に役立ちます。しかしながら、もしLaravelの[validation services](http://laravel.com/docs/5.1/validation)を使っている場合、Laravelのビルトインバリデーション設備のいくつかはそれらを自動的に呼ぶため、手動でこれらのメソッドを使う必要はほとんどありません。

#### Flashing Input To The Session

`Illuminate\Http\Request`インスタンス上の`flash`メソッドは、現在の入力を[session](http://laravel.com/docs/5.1/session)に出力するので、アプリケーションへのユーザーの次の入力の間それを利用できます。
```php
$request->flash();
```
`flashOnly`と`flashExcept`メソッドもリクエストデータのサブセットをセッションの中にフラッシュするのに利用できます。
```php
$request->flashOnly('username', 'email');

$request->flashExcept('password');
```

#### Flash Input Into Session Then Redirect

よくリダイレクトに関連する入力を以前のページにフラッシュしたいので、簡単に`wishInput`メソッドを使っているリダイレクトにフラッシュしている入力を連鎖できます。
```php
return redirect('form')->withInput();

return redirect('form')->withInput($request->except('password'));
```

#### Retrieving Old Data

以前のリクエストからフラッシュされた入力を取得するために、`Request`インスタンスの`old`メソッドを使います。`old`メソッドはフラッシュされた入力データを[セッション](http://laravel.com/docs/5.1/session)から抜き出すための便利なヘルパーを提供します。
```php
$username = $request->old('username');
```
Laravelはグローバル`old`ヘルパー関数も提供します。もし古い入力を[Blade template](http://laravel.com/docs/5.1/blade)内で表示したい場合、`old`ヘルパーを使うのがより便利です。
```php
{{ old('username') }}
```
