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
