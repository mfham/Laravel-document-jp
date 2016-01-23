## Basic Usage

ビューはアプリケーションによってもたらされるHTMLを含み、コントローラー・アプリケーションロジックをプレゼンテーションロジックから分けます。ビューは`resources/views`ディレクトリ内に格納されます。
単純なビューはこのように見えるかもしれません。

```php
<!-- View stored in resources/views/greeting.php -->

<html>
    <body>
        <h1>Hello, <?php echo $name; ?></h1>
    </body>
</html>
```

このビューは`resources/views/greeting.php`に格納されるので、グローバル`view`ヘルパー関数を使っているそれを返すかもしれません。

```php
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});
```

