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

## View Data

#### Passing Data To Views

前の例で見たように、簡単にデータ配列をビューに渡すことができます。
```php
return view('greetings', ['name' => 'Victoria']);
```

この作法で情報を渡す時、`$data`はキーバリューペアの配列です。ビューの中で、キーに対応したそれぞれの値に`<?php echo $key; ?>`のようにアクセスできます。`view`ヘルパー関数に完全なデータ配列を渡す代わりの方法として、個々のデータをビューに渡すために`with`メソッドを使うことができます。
```php
return view('greeting')->with('name', 'Victoria');
```

#### Sharing Data With All Views

たまに、データの一部をアプリケーションによってレンダーされるすべてのビューで共有したいかもしれません。ビューファクトリーの`share`メソッドを使うことができます。一般的に、サービスプロバイダーの`boot`メソッド内で`share`を呼びます。自由にそれらを`AppServiceProvider`に追加したり、それらをしまっておくためにいくつかのサービスプロバイダを生成したりすることができます。

```php
<?php

namespace App\Providers;

class AppServiceProvider extends ServiceProvider
{
    /**
    * Bootstrap any application services.
    *
    * @return void
    */
    public function boot()
    {
        view()->share('key', 'value');
    }

    /**
    * Register the service provider.
    *
    * @return void
    */
    public function register()
    {
        //
    }
}
```

## View Composers

ビューコンポーザーはビューを生成するときに呼ばれるコールバックもしくはクラスメソッドです。もしビューが生成されるたびにビューにデータを結びつけたいなら、ビューコンポーザーはロジックを一つの場所にまとめるのに役に立ちます。
[service provider](https://laravel.com/docs/5.2/providers)の中でビューコンポーザーを登録してみましょう。基礎的な`Illuminate\Contracts\View\Factory`の契約実装にアクセスするために`view`ヘルパーを使います。Laravelはビューコンポーザーのためのデフォルトディレクトリを含まないことを覚えておいてください。どのように望んでも自由にそれらを組織できます。例えば、`App\Http\ViewComposers`ディレクトリを作成できます。

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;

class ComposerServiceProvider extends ServiceProvider
{
    /**
    * Register bindings in the container.
    *
    * @return void
    */
    public function boot()
    {
        // Using class based composers...
        view()->composer(
            'profile', 'App\Http\ViewComposers\ProfileComposer'
        );

        // Using Closure based composers...
        view()->composer('dashboard', function ($view) {
            //
        });
    }

    /**
    * Register the service provider.
    *
    * @return void
    */
    public function register()
    {
        //
    }
}
```

もしビューコンポーザー登録を含むために新しいサービスプロバイダーを作りたいなら、サービスプロバイダを`config/app.php`設定ファイルに`providers`配列に追加する必要があります。
コンポーザーを登録したら、`profile`ビューが実行されるたびに`ProfileComposer@compose`メソッドが実行されます。それでは、コンポーザークラスを定義してみましょう。
```php
<?php

namespace App\Http\ViewComposers;

use Illuminate\View\View;
use App\Repositories\UserRepository;

class ProfileComposer
{
    /**
    * The user repository implementation.
    *
    * @var UserRepository
    */
    protected $users;

    /**
    * Create a new profile composer.
    *
    * @param  UserRepository  $users
    * @return void
    */
    public function __construct(UserRepository $users)
    {
        // Dependencies automatically resolved by service container...
        $this->users = $users;
    }

    /**
    * Bind data to the view.
    *
    * @param  View  $view
    * @return void
    */
    public function compose(View $view)
    {
        $view->with('count', $this->users->count());
    }
}
```
ちょうどビューが生成される前、コンポーザーの`compose`メソッドが`Illuminate\View\View`で呼ばれます。ビューにデータをバインドするために`with`メソッドを使います。
注意: すべてのビューコンポーザーは[`service container`](https://laravel.com/docs/5.2/container)を通じて解決されるので、あなたはコンポーザー構成の中で必要とするどの依存性もタイプヒントできます。

#### Attaching A Composer To Multiple Views

ビューの配列を第一引数として`composer`メソッドに渡すことによって、ビューコンポーザーを1回で複数のビューにアタッチできます。
```php
view()->composer(
    ['profile', 'dashboard'],
    'App\Http\ViewComposers\MyViewComposer'
);
```

`composer`メソッドはワイルドカードとして`*`文字を受け入れ、コンポーザーをすべてのビューにアタッチするのを許可します。
```php
view()->composer('*', function ($view) {
    //
});
```

## View Creators

ビュークリエイターはビューコンポーザーに似ています。しかしながら、それらはビューが生成されようとするまで待つ代わりにビューがインスタンス化されるとき、すぐに発火します。ビュークリエイターを登録するために、`creator`メソッドを使います。
```php
view()->creator('profile', 'App\Http\ViewCreators\ProfileCreator');
```
