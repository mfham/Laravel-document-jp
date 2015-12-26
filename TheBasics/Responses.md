## Basic Responses

もちろん、全てのルートとコントローラはユーザーのブラウザに返却されるいくつかの種類のレスポンスを返すべきです。Laravelはレスポンスを返すいくつか違った方法を提供します。もっとも基本的なレスポンスは単純にルートやコントローラから文字列を返すものです。

```php
Route::get('/', function () {
    return 'Hello World';
});
```

与えられた文字列はフレームワークによって自動的にHTTPレスポンスに変換されます。

#### Response Objects

しかしながら、ほとんどのルートやコントローラアクションに対して、フル`Illuminate\Http\Response`インスタンスもしくは[view](https://laravel.com/docs/5.2/views)を返します。フル`Response`インスタンスを返すことはレスポンスのHTTPステータスコードやヘッダーをカスタマイズできるようにしてくれます。`Response`インスタンスは`Symfony\Component\HttpFoundation\Response`クラスから継承し、HTTPレスポンスを構築するために様々なメソッドを提供します。

```php
use Illuminate\Http\Response;

Route::get('home', function () {
    return (new Response($content, $status))
                  ->header('Content-Type', $value);
});
```
便利のいいように、`response`ヘルパーも使うことができます、

```php
Route::get('home', function () {
    return response($content, $status)
                  ->header('Content-Type', $value);
});
```
注意: 利用できる`Response`メソッドの全てのリストに対して[APIドキュメント](http://laravel.com/api/master/Illuminate/Http/Response.html)や[SymfonyAPIドキュメント](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Response.html)を参照してください。

#### Attaching Headers To Responses

多くのレスポンスメソッドはチェーンが可能で流暢なレスポンス形成が許されているということを心に留めておいてください。例えばユーザーに送られる前にヘッダーの一連をレスポンスに加えるためには`header`メソッドを使います。

```php
return response($content)
            ->header('Content-Type', $type)
            ->header('X-Header-One', 'Header Value')
            ->header('X-Header-Two', 'Header Value');
```

もしくは、レスポンスに追加されるヘッダー配列を指定するために`withHeaders`メソッドを使います。

```php
return response($content)
            ->withHeaders([
                'Content-Type' => $type,
                'X-Header-One' => 'Header Value',
                'X-Header-Two' => 'Header Value',
            ]);
```

#### Attaching Cookies To Responses

レスポンスインスタンスの`cookie`ヘルパーメソッドはクッキーをレスポンスにアタッチするのを簡単にしてくれます。例えば、クッキーを生成しレスポンスインスタンスにアタッチするために`cookie`メソッドを使います。

```php
return response($content)
                 ->header('Content-Type', $type)
                 ->cookie('name', 'value');
```

`cookie`メソッドはもっとクッキーのプロパティをカスタマイズすることを許す追加オプション引数を受け入れます。

```php
->cookie($name, $value, $minutes, $path, $domain, $secure, $httpOnly)
```

デフォルトで、Laravelによって生成された全てのクッキーは暗号化され署名されるので、それらはクライアントによって変更や読み取りができません。もしアプリケーションで生成される特定のクッキーのサブセットのために暗号化を使用可能にしたい場合、`App\Http\Middleware\EncryptCookies`ミドルウェアの`$except`プロパティが使えます。

```php
/**
  * The names of the cookies that should not be encrypted.
  *
  * @var array
  */
protected $except = [
    'cookie_name',
];
```
