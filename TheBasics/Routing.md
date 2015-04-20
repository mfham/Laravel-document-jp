# Basic Routing

アプリケーションのためのほとんどのルーティングを```app/Http/routes.php```ファイルで定義でき、```App\Providers\RouteServiceProvider```クラスによってロードされます。ほとんどの基礎的なLaravelのルートは、簡単にURLや```Closure```を受け入れます。

## Basic Get Route

```php
Route::get('/', function()
{
    return 'Hello World';
});
```
