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

## Registering Middleware

### Global Middleware

もしアプリケーションへの全てのHTTPリクエストの間ミドルウェアを動作させたい場合、シンプルにミドルウェアクラスを`app/Http/Kernel.php`クラスの`$middleware`プロパティ内に並べます。

### Assigning Middleware To Routes

もしミドルウェアに特定のルートをアサインしたい場合、最初に`app/Http/Kernel.php`ファイルの中でミドルウェアに記号キーをアサインしなければなりません。デフォルトで、このクラスの`$routeMiddleware`プロパティはLaravelに含まれるミドルウェアのエントリーを含みます。あなた自身のものを追加するために、簡単にそれをこのリストに追加し、決めたキーにアサインすればよいです。
例:

```php
// Within App\Http\Kernel Class...

protected $routeMiddleware = [
   'auth' => 'App\Http\Middleware\Authenticate',
   'auth.basic' => 'Illuminate\Auth\Middleware\AuthenticateWithBasicAuth',
   'guest' => 'App\Http\Middleware\RedirectIfAuthenticated',
];
```

いったんHTTPカーネルの中でミドルウェアが定義されたら、ルートオプション配列の中で`middleware`キーを使ってもいいです。

```php
Route::get('admin/profile', ['middleware' => 'auth', function () {
    //
}]);
```

## Middleware Parameters

ミドルウェアは追加のカスタムパラメータも受け取ることが出来ます。例えば、与えられたアクションを実行する前に認証されたユーザーが与えられた"role"をもっているか確かめる必要がある場合、追加引数としてrole名を受け取る`RoleMiddleware`を生成できます。

追加のミドルウェアパラメータは`$next`引数の後でミドルウェアに渡されます。

```php
<?php

namespace App\Http\Middleware;

use Closure;

class RoleMiddleware
{
    /**
     * Run the request filter.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @param  string  $role
     * @return mixed
     */
    public function handle($request, Closure $next, $role)
    {
        if (! $request->user()->hasRole($role)) {
            // Redirect...
        }

        return $next($request);
    }

}
```
ミドルウェア名とパラメータを`:`で分けることによってルートを定義するとき、ミドルウェアパラメータは明記されるかもしれない。複数のパラメータはカンマで区切られます。

```php
Route::put('post/{id}', ['middleware' => 'role:editor', function ($id) {
    //
}]);
```

## Terminable Middleware

ときどき、ミドルウェアはHTTPレスポンスがすでにブラウザへ送られた後にいくつかの仕事をする必要があるかもしれません。例えば、Laravelに含まれている"session"ミドルウェアは、レスポンスがブラウザへ送られた後にストレージへセッションデータを書き込みます。これを成し遂げるために、ミドルウェアに`terminate`メソッドを追加することによってミドルウェアに"terminable"として定義します。

```php

<?php namespace Illuminate\Session\Middleware;

use Closure;

class StartSession
{
    public function handle($request, Closure $next)
    {
        return $next($request);
    }

    public function terminate($request, $response)
    {
        // Store the session data...
    }
}
```
`terminate`メソッドはリクエストとレスポンスの両方を受け取るべきです。いったん期限付きミドルウェアを定義したら、HTTPカーネルの中でグローバルミドルウェアのリストにそれを追加すべきです。
