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
