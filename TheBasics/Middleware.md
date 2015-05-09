## Introduction

HTTPミドルウェアは、アプリケーションに入ってくるHTTPリクエストをフィルタリングするための便利な仕組みを提供します。例えば、Laravelはアプリケーションのユーザーが認証されているか確認するミドルウェアを含みます。もしユーザーが認証されていない場合、ミドルウェアはログイン画面へユーザーをリダイレクトさせます。しかしながら、もしユーザーが人相されている場合、ミドルウェアはリクエストがさらにアプリケーションの中へ進むのを許可します。

もちろん、ミドルウェアは認証の他にも様々なタスクを実行するために書かれます。CORSミドルウェアはアプリケーションの全てのレスポンスに適切なヘッダーを加える責任があるかもしれません。ロギングミドルウェアはアプリケーションに入ってくる全てのリクエストのログを取るかもしれません。

Laravelフレームワークの中に含まれているミドルウェアがいくつかあり、メンテナンス、認証、CSRF防止、その他のミドルウェアを含みます。これらのミドルウェア全て、`app/Http/Middleware`に配置されています。

## Defining Middleware

新しいミドルウェアを作成するために、Aritisanコマンドの`make:middleware`を使います。

```
php artisan make:middleware OldMiddleware
```

このコマンドは`app/Http/Middleware`ディレクトリ内に新しい`OldMiddleware`クラスを配置します。このミドルウェアの中では、`age`が200以上の場合ルートへのアクセスを許可します。さもなければ、ユーザーを"home"URLへリダイレクトさせます。

```php
<?php namespace App\Http\Middleware;

class OldMiddleware {

    /**
    * Run the request filter.
    *
    * @param  \Illuminate\Http\Request  $request
    * @param  \Closure  $next
    * @return mixed
    */
public function handle($request, Closure $next)
{
    if ($request->input('age') < 200)
    {
        return redirect('home');
    }

    return $next($request);
    }

}
```

分かるように、もし与えられた`age`が200より小さい場合、ミドルウェアはクライアントへHTTPリダイレクトを返します。さもなければ、リクエストはさらにアプリケーションの内部へ渡されるでしょう。リクエストをより深いアプリケーション内部へ渡すには(ミドルウェアに"pass"を許可する)、シンプルに`$request`で`$next`コールバックを呼びます。

HTTPリクエストがアプリケーションに届く前に通過しなければならない"layers"の続きとしてミドルウェアを描くのはいいことです。

### Before / After Middleware

ミドルウェアがリクエストの前か後に実行されるかはミドルウェア次第です。このミドルウェアはリクエストがアプリケーションによって操作される**前に**タスクを実行します。

```php
<?php namespace App\Http\Middleware;

class BeforeMiddleware implements Middleware {

    public function handle($request, Closure $next)
    {
        // Perform action

        return $next($request);
    }
}
```

しかしながら、このミドルウェアはアプリケーションにとって操作された**後に**タスクを実行するでしょう。

```php
<?php namespace App\Http\Middleware;

class AfterMiddleware implements Middleware {

    public function handle($request, Closure $next)
    {
        $response = $next($request);

        // Perform action

        return $response;
    }
}
```
