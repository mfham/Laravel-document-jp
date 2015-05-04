## Basic Routing

アプリケーションのためのほとんどのルーティングを`app/Http/routes.php`ファイルで定義でき、`App\Providers\RouteServiceProvider`クラスによってロードされます。ほとんどの基礎的なLaravelのルートは、簡単にURLや`Closure`を受け入れます。

#### Basic Get Route

```php
Route::get('/', function()
{
    return 'Hello World';
});
```

#### Other Basic Routes

```php
Route::post('foo/bar', function()
{
    return 'Hello World';
});

Route::put('foo/bar', function()
{
    //
});

Route::delete('foo/bar', function()
{
    //
});
```
#### Registering A Route That Responds To Any HTTP Verb

```php
Route::any('foo', function()
{
    return 'Hello World';
});
```
しばしば、ルートのためにURLを生成する必要があり、`url`ヘルパーを使うかもしれません。

## CSRF Protection

Laravelはアプリケーションを[クロスサイトリクエストフォージェリ](http://ja.wikipedia.org/wiki/%E3%82%AF%E3%83%AD%E3%82%B9%E3%82%B5%E3%82%A4%E3%83%88%E3%83%AA%E3%82%AF%E3%82%A8%E3%82%B9%E3%83%88%E3%83%95%E3%82%A9%E3%83%BC%E3%82%B8%E3%82%A7%E3%83%AA)から防ぐのを簡単にします。クロスサイトリクエストフォージェリは、認証されたユーザーの代わりに不正なコマンドが実行される悪意のある攻撃の一種です。

Laravelはアプリケーションによって管理されているアクティブなユーザーセッションそれぞれに対して自動的にCSRFトークンを生成します。このトークンはアプリケーションへのリクエストを生成している認証されたユーザーが唯一であることを確かめるために使われます。

#### Insert The CSRF Token Into A Form

```html
<input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">
```
もちろん、Blade[テンプレートエンジン](http://laravel.com/docs/5.0/templates)使うと

```html
<input type="hidden" name="_token" value="{{ csrf_token() }}">
```
POST、PUT、DELETEリクエストで、手動でCSRFトークンを確かめる必要はありません。`VerifyCsrfToken`[HTTPミドルウェア](http://laravel.com/docs/5.0/middleware)は、入力リクエストのトークンがセッションに格納されたトークンとマッチするか確かめます。

#### X-CSRF-TOKEN

POSTパラメータとしてのCSRFトークンを探していることに加えて、ミドルウェアは`X-CSRF-TOKEN`リクエストヘッダーもチェックします。例えば、metaタグの中にトークンを格納できたり、全てのリクエストヘッダーにそれを加えるようjQueryに指示することも出来ます。

```html
<meta name="csrf-token" content="{{ csrf_token() }}" />

$.ajaxSetup({
        headers: {
            'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
        }
    });
```
今、全てのAJAXリクエストは自動的にCSRFトークンを含みます。
```
$.ajax({
    url: "/foo/bar",
})
```

#### X-XSRF-TOKEN

Laravelは`XSRF-TOKEN`クッキーの中にもCSRFトークンを格納します。クッキーの値も使えます。`X-XSRF-TOKEN`リクエストヘッダーにセットするのにクッキーの値を使えます。AngularのようないくつかのJavascriptフレームワークでは、これを自動的にやります。

注意:`X-CSRF-TOKEN`と`X-XSRF-TOKEN`の違いは、Laravelのクッキーは常に暗号化されるので、前者は平文の値を使い、後者は暗号化された値を使います。もしトークンの値を提供するために`csrf_token()`関数を使うなら、`X-CSRF-TOKEN`ヘッダーを使いたくなるでしょう。

## Method Spoofing

HTMLフォームは`PUT`、`PATCH`、`DELETE`動作をサポートしません。だから、HTMLフォームから呼ばれる`PUT`、`PATCH`、`DELETE`ルートを定義するとき、フォームにhidden`_method`フィールドを追加する必要があります。

`_method`フィールドで送られる値はHTTPリクエストメソッドとして使われます。例えば、

```html
<form action="/foo/bar" method="POST">
    <input type="hidden" name="_method" value="PUT">
    <input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">
</form>
```
## Route Parameters

もちろん、ルート内のリクエストURLのセグメントを補足することができます。

#### Basic Route Parameter

```php
Route::get('user/{id}', function($id)
{
    return 'User '.$id;
});
```

注意: ルートパラメーターは`-`文字列を含むことができません。代わりにアンダースコア(`_`)を使ってください。

#### Optional Route Parameters

```php
Route::get('user/{name?}', function($name = null)
{
    return $name;
});
```

#### Optional Route Parameters With Default Value

```php
Route::get('user/{name?}', function($name = 'John')
{
    return $name;
});
```
#### Regular Expression Parameter Constraints

```php
Route::get('user/{name}', function($name)
{
    //
})
->where('name', '[A-Za-z]+');

Route::get('user/{id}', function($id)
{
    //
})
->where('id', '[0-9]+');
```

#### Passing An Array Of Constraints

```php
Route::get('user/{id}/{name}', function($id, $name)
{
    //
})
->where(['id' => '[0-9]+', 'name' => '[a-z]+'])
```

#### Defining Global Patterns

もし正規表現によってルートパラメータにいつも制約を与えたい場合、`pattern`メソッドが使えます。`RouteServiceProvider`の`boot`メソッドの中でこれらのパターンを定義します。

```php
$router->pattern('id', '[0-9]+');
```

一度パターンが定義されたら、そのパラメータを使う全てのルートに適用されます。

```php
Route::get('user/{id}', function($id)
{
    // Only called if {id} is numeric.
});
```

#### Accessing A Route Parameter Value

もしルートの外でルートパラメータにアクセスする必要がある場合、`input`メソッドを使います。

```php
if ($route->input('id') == 1)
{
    //
}
```

`Illuminate\Http\Request`インスタンスを通じて現在のルートパラメータにもアクセスできます。現在のリクエストのためのリクエストインスタンスは`Request`ファサードを通じて、もしくは依存性が注入された`Illuminate\Http\Request`のタイプヒンティングによってアクセスされます。

```php
use Illuminate\Http\Request;

Route::get('user/{id}', function(Request $request, $id)
{
    if ($request->route('id'))
    {
        //
    }
});
```

## Named Routes

名前付きルートは都合よくURLを生成する、もしくは特定のルートに対してリダイレクトするのを可能にします。配列の`as`キーでルートに名前を付けられます。

```php
Route::get('user/profile', ['as' => 'profile', function()
{
    //
}]);
```
コントローラーアクションに対してルート名も付けられます。

```php
Route::get('user/profile', [
    'as' => 'profile', 'uses' => 'UserController@showProfile'
]);
```

今、URL生成もしくはリダイレクトのとき、ルート名を使います。

```php
$url = route('profile');

$redirect = redirect()->route('profile');
```

`currentRouteName`メソッドはハンドリングしている現在のリクエストのルートの名前を返します。

```php
$name = Route::currentRouteName();
```

## Route Groups

時々、ルートの多くはURLセグメント、ミドルウェア、名前空間など、共通の要件を共有します。個々のルートのそれぞれのオプションを明記する代わりに、属性を多くのルートに提供するルートグループを使ってもよいです。
共有された属性は`Route::group`メソッドの最初のパラメーターとして配列フォーマットで明記されます。

### Middleware

ミドルウェアは、グループ属性の配列に`middleware`パラメータでミドルウェアのリストを定義することによって、グループ内の全てのルートに提供される。

```php
Route::group(['middleware' => ['foo', 'bar']], function()
{
    Route::get('/', function()
    {
        // Has Foo And Bar Middleware
    });

    Route::get('user/profile', function()
    {
        // Has Foo And Bar Middleware
    });
});
```

### Namespaces

グループ内の全てのコントローラに対して名前空間を明記するためにグループ属性配列の中で`namespace`パラメータを使えます。

```php
Route::group(['namespace' => 'Admin'], function()
{
    // Controllers Within The "App\Http\Controllers\Admin" Namespace

    Route::group(['namespace' => 'User'], function()
    {
        // Controllers Within The "App\Http\Controllers\Admin\User" Namespace
    });
});
```

注意: デフォルトで、`RouteServiceProvider`は名前空間グループ内に`routes.php`ファイルを含み、`App\Http\Controllers`の名前空間接頭辞を明記せずにコントローラールートを登録できます。

### Sub-Domain Routing

Laravelルートはワイルドカードサブドメインも操作し、ドメインからワイルドカードパラメータを渡します。

#### Registering Sub-Domain Routes

```php
Route::group(['domain' => '{account}.myapp.com'], function()
{

    Route::get('user/{id}', function($account, $id)
    {
        //
    });
});
```

### Route Prefixing

ルートのグループは、グループの属性配列の中で`prefix`オプションを使うことで接頭辞をつけることが出来ます。

```php
Route::group(['prefix' => 'admin'], function()
{
    Route::get('users', function()
    {
        // Matches The "/admin/users" URL
    });
});
```
ルートへの共通のパラメータを渡すために`prefix`パラメーターを使うこともできます。

#### Registering a URL parameter in a route prefix

```php
Route::group(['prefix' => 'accounts/{account_id}'], function()
{
    Route::get('detail', function($account_id)
    {
        //
    });
});
```

接頭辞の中の名前付きパラメータのためにパラメーター制約を定義することでさえできます。

```php
Route::group([
    'prefix' => 'accounts/{account_id}',
    'where' => ['account_id' => '[0-9]+'],
], function() {

    // Define Routes Here
});
```

## Route Model Binding

Laravelモデルバインドはルートにクラスインスタンスを注入するために便利な方法を提供します。例えば、ユーザーIDを注入する代わりに、与えられたIDにマッチする全体のUserクラスインスタンスを注入できます。
与えられたパラメータに対してクラスを明記するために、最初にルーターの`model`メソッドを使ってください。`RouteServiceProvider::boot`メソッドの中に、モデルバインディングを定義すべきです。

#### Binding A Parameter To A Model

```php
public function boot(Router $router)
{
    parent::boot($router);

    $router->model('user', 'App\User');
}
```

次に、`{user}`パラメータを含むルートを定義してください。

```php
Route::get('profile/{user}', function(App\User $user)
{
    //
});
```

`{user}`パラメーターを`App\User`モデルにバインドしたので、`User`インスタンスはルートに注入されます。だから、例えば`profile/1`へのリクエストは1のIDを持つ`User`インスタンスを注入するでしょう。

注意: もしマッチングモデルインスタンスがデータベース内に見つからなかった場合、404エラーが投げれられます。

もしあなた自身の"not found"動作を明記したい場合、第三引数としてクロージャを`model`メソッドに渡します。

```php
Route::model('user', 'User', function()
{
    throw new NotFoundHttpException;
});
```

もしあなた自身の分析ロジックを使いたい場合、`Route::bind`メソッドを使うべきです。`bind`メソッドに渡すクロージャはURLセグメントの値を受け取り、ルートに注入したいクラスのインスタンスを返します。


```php
Route::bind('user', function($value)
{
    return User::where('name', $value)->first();
});
```

## Throwing 404 Errors

ルートから自動的に404エラーを起こす2つの方法があります。1つ目は`abort`ヘルパーを使います。

```php
abort(404);
```

`abort`ヘルパーは簡単に明記されたステータスコードと一緒に`Symfony\Component\HttpFoundation\Exception\HttpException`を投げます。

2つ目は自動的に`Symfony\Component\HttpKernel\Exception\NotFoundHttpException`インスタンスを投げます。

404例外を操作し、これらのエラーに対するカスタムレスポンスを使うためのより多くの情報はドキュメントの[errors](http://laravel.com/docs/5.0/errors#http-exceptions)節で見つかるでしょう。
