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
わかるように、`view`ヘルパーに渡される第一引数は`resources/views`ディレクトリの中のビューファイルの名前に対応します。ヘルパーに渡される第二引数は、ビューを利用可能にするデータの配列です。この場合、変数に対して`echo`を実行することによってビューに表示される`name`変数を渡しています。

もちろん、ビューは`resources/views`ディレクトリのサブディレクトリ内で階層になっているかもしれません。ドット記法は階層になったビューの参照に使えるかもしれません。例えば、もしあなたのビューが`resources/views/admin/profile.php`に保存されていたら、あなたはそれをこのように参照できます。
```php
return view('admin.profile', $data);
```

#### Determining If A View Exists

ビューが存在するか測定したい場合、引数なしで`view`ヘルパーを呼んだ後に`exists`メソッドが使えます。このメソッドはもしビューがディスクに存在していれば`true`を返します。
```php
if (view()->exists('emails.customer')) {
    //
}
```
`view`ヘルパーが引数なしで呼ばれたとき、`Illuminate\Contracts\View\Factory`のインスタンスは返され、ファクトリーのメソッドへのアクセスが与えられます。

