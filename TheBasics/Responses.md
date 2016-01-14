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

## Other Response Types

`response`ヘルパーは他の種類のレスポンスインスタンスを便利に生成するために使われるかもしれません。`response`ヘルパーが引数なしで呼ばれる時、`Illuminate\Contracts\Routing\ResponseFactory`[contract](https://laravel.com/docs/5.2/contracts)の実装が返されます。この契約はいくつかのヘルパーメソッドにレスポンスを生成するのを供給します。

#### View Responses

もしレスポンスステータスとヘッダーを超えてコントロールする必要があるが、レスポンスコンテンツのように[view](https://laravel.com/docs/5.2/views)を返す場合、viewメソッドが使えます:

```php
return response()
            ->view('hello', $data)
            ->header('Content-Type', $type);
```

もちろん、もしカスタムHTTPステータスコードやカスタムヘッダーを渡す必要がない場合、単純にグローバル`view`ヘルパー関数を使うべきです。

#### JSON Responses

`json`メソッドは与えられた配列を`json_encode`PHP関数を使っているJSONへ変換するだけでなく自動的に`Content-Type`ヘッダーを`application/json`へセットします。

```php
return response()->json(['name' => 'Abigail', 'state' => 'CA']);
```

もしJSONPレスポンスを生成したい場合、`setCallback`に加えて`json`メソッドを使います:

```php
return response()
            ->json(['name' => 'Abigail', 'state' => 'CA'])
            ->setCallback($request->input('callback'));
```

#### File Downloads

`download`メソッドはユーザーのブラウザに強制的に与えられたパスのファイルをダウンロードさせるレスポンスを生成するのに使われます。`download`メソッドは第二引数としてファイル名を受け付け、それはユーザーによってダウンロードされるファイル名を決めます。最後に、第三引数としてHTTPヘッダーの配列を渡します。

```php
return response()->download($pathToFile);

return response()->download($pathToFile, $name, $headers);
```

注意: ファイルダウンロードを管理するSymfonyHttpFoundationは、ダウンロードされるファイルがASCIIファイル名であることを必要とします。

## Redirects

リダイレクトレスポンスは`Illuminate\Http\RedirectResponse`クラスのインスタンスで、それはユーザーを他のURLにリダイレクトさせるのに必要な適切なヘッダーも含んでいます。`RedirectResponse`インスタンスを生成する方法がいくつかあります。一番シンプルなメソッドはグローバルな`redirect`ヘルパーメソッドを使うことです。

```php
Route::get('dashboard', function () {
    return redirect('home/dashboard');
});
```
