# Basic Routing

アプリケーションのためのほとんどのルーティングを```app/Http/routes.php```ファイルで定義でき、```App\Providers\RouteServiceProvider```クラスによってロードされます。ほとんどの基礎的なLaravelのルートは、簡単にURLや```Closure```を受け入れます。

## Basic Get Route

```php
Route::get('/', function()
{
    return 'Hello World';
});
```

## Other Basic Routes

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
## Registering A Route That Responds To Any HTTP Verb

```php
Route::any('foo', function()
{
    return 'Hello World';
});
```
しばしば、ルートのためにURLを生成する必要があり、```url```ヘルパーを使うかもしれません。

# CSRF Protection

Laravelはアプリケーションを[クロスサイトリクエストフォージェリ](http://ja.wikipedia.org/wiki/%E3%82%AF%E3%83%AD%E3%82%B9%E3%82%B5%E3%82%A4%E3%83%88%E3%83%AA%E3%82%AF%E3%82%A8%E3%82%B9%E3%83%88%E3%83%95%E3%82%A9%E3%83%BC%E3%82%B8%E3%82%A7%E3%83%AA)から防ぐのを簡単にします。クロスサイトリクエストフォージェリは、認証されたユーザーの代わりに不正なコマンドが実行される悪意のある攻撃の一種です。

Laravelはアプリケーションによって管理されているアクティブなユーザーセッションそれぞれに対して自動的にCSRFトークンを生成します。このトークンはアプリケーションへのリクエストを生成している認証されたユーザーが唯一であることを確かめるために使われます。

## Insert The CSRF Token Into A Form

```
<input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">
```
もちろん、Blade[テンプレートエンジン](http://laravel.com/docs/5.0/templates)使うと

```
<input type="hidden" name="_token" value="{{ csrf_token() }}">
```
POST、PUT、DELETEリクエストで、手動でCSRFトークンを確かめる必要はありません。```VerifyCsrfToken```[HTTPミドルウェア](http://laravel.com/docs/5.0/middleware)は、入力リクエストのトークンがセッションに格納されたトークンとマッチするか確かめます。


