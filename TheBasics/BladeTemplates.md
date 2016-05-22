## Introduction

Bladeはシンプルで、しかしLaravelによって提供されるパワフルなテンプレートエンジンです。他の人気なPHPテンプレートエンジンとは違って、Bladeはビュー内で普通のPHPを使うことから制限しません。全てのBladeビューは普通のPHPコードにコンパイルされ、変更されるまでキャッシュされ、それはBladeが本質的にアプリケーションへのオーバーヘッドがゼロであることを意味します。Bladeビューファイルは`.blade.php`ファイル拡張子を使い、典型的に`resources/views`ディレクトリ内に保存される。

## Template Inheritance

#### Defining A Layout

Bladeを使う主要な利益の2つは、テンプレート継承とセクションです。始めるため、簡単な例を見てみましょう。"master"ページレイアウトを調査してみます。ほとんどのウェブアプリケーションは同じ一般的なレイアウトを様々なページを通じて維持するので、このレイアウトを単一のBladeビューとして定義するのに都合が良いです。
```php
<!-- Stored in resources/views/layouts/master.blade.php -->

<html>
    <head>
        <title>App Name - @yield('title')</title>
    </head>
    <body>
        @section('sidebar')
            This is the master sidebar.
        @show

    <div class="container">
        @yield('content')
    </div>
    </body>
</html>
```

わかるように、このファイルは典型的なHTMLマークアップを含みます。しかしながら、`@section`と`@yield`ディレクティブに気をつけて下さい。`@section`ディレクトリは名前が意味するように、コンテンツのセクションを定義します。一方、`@yield`ディレクティブは与えられたコンテンツを表示するのに使われます。
今アプリケーションのためにレイアウトを定義したので、レイアウトを継承した子ページを定義しましょう。

#### Extending A Layout

子ページを定義するとき、どのレイアウトが継承すべき子ページであるか指定するためにBladeの`@extends`ディレクトリを使うかもしれません。Bladeレイアウトを`@extends`するビューはコンテンツをレイアウトの`@section`ディレクティブを使っているセクションに注入します。上記の例で分かるように、それらのセクションの内容は`@yield`を使っているレイアウトの中で表示されるので、覚えておいてください。

```php
<!-- Stored in resources/views/child.blade.php -->

@extends('layouts.master')

@section('title', 'Page Title')

@section('sidebar')
    @parent

    <p>This is appended to the master sidebar.</p>
@endsection

@section('content')
    <p>This is my body content.</p>
@endsection
```

この例で、`sidebar`セクションは内容をレイアウトのサイドバーに追加するため(上書きというよりも)に`@parent`ディレクティブを利用しています。
もちろん、普通のPHPビューのように、Bladeビューはグローバル`view`ヘルパー関数を使っているルートから返されるかもしれません。
```php
Route::get('blade', function () {
    return view('child');
});
```

## Displaying Data

波括弧の中で変数をらっぷすることでBladeビューに渡されるデータを表示するかもしれません。例えば、次のルートが与えられると、
```php
Route::get('greeting', function () {
    return view('welcome', ['name' => 'Samantha']);
});
```
このように`name`変数の中身を表示できます。
```php
Hello, {{ $name }}.
```
もちろん、ビューに渡された変数の内容を表示するに限りません。PHP関数の結果も表示できます。実際、あなたが望むどんなPHPコードでもBladeのecho文内に入れることができます。
```php
The current UNIX timestamp is {{ time() }}.
```

注意: Bladeの`{{ }}`文はXSS攻撃を防ぐために自動的にPHPの`htmlentities`関数を通して送られます。

#### Blade & JavaScript Frameworks

多くのJavaScriptフレームワークは、ブラウザ内で表示されるべき与えられた表現を示すために波括弧も使っているので、Bladeのレンダリングエンジンに表現はそのままであるべきと知らせるために`@`記号を使います。例えば、
```php
<h1>Laravel</h1>

Hello, @{{ name }}.
```

この例の中で、`@`記号はBladeによって取り除かれます。しかしながら、`{{name}}`表現はBladeエンジンによってそのままにされ、あなたのJavaScriptフレームワークによって代わりにレンダリングできるようにします。

#### Echoing Data If It Exists

時々、変数を表示したいかもしれないが、変数がセットされているかわからないかもしれません。冗長なPHPコードでこれをこのように表現できます。
```php
{{ isset($name) ? $name : 'Default' }}
```

しかしながら、三項演算子を書く代わりに、Bladeは次の便利なショートカットを提供します。
```php
{{ $name or 'Default' }}
```

この例の中で、もし`$name`変数が存在する場合、その値は表示されるでしょう。しかしながら、もし存在しない場合、`Default`という文字列が表示されるでしょう。

#### Displaying Unescaped Data

デフォルトで、Bladeの`{{ }}`文はXSS攻撃を防ぐために自動的にPHPの`htmlentities`関数を通して送られます。もしデータをエスケープしたくない場合、次のシンタックスが使えます。
```php
Hello, {!! $name !!}.
```

注意: アプリケーションのユーザーによって提供される内容を表示する時はとても気をつけて下さい。どんなHTMLもエスケープできるようにいつも二重波括弧シンタックスを使ってください。

## Control Structures

テンプレート継承とデータ表示に加えて、Bladeは条件文やループのような普通のPHP制御構造のために便利なショートカットも提供します。これらのショートカットは、とてもきれいな、PHP制御構造で動く簡潔な方法で、一方でPHPの対応するものとなじみのあるままでもあります。

#### If Statements

`@if`、`@elseif`、`@else`、そして`@endif`ディレクティブを使っている`if`文を構築できます。これらのディレクティブはPHPの対応するものと同様に動きます。
```php
@if (count($records) === 1)
    I have one record!
@elseif (count($records) > 1)
    I have multiple records!
@else
    I don't have any records!
@endif
```

便利のために、Bladeは`@unless`ディレクティブも提供します。

```php
@unless (Auth::check())
    You are not signed in.
@endunless
```

与えられたレイアウトセクションが`@hasSection`ディレクティブを使っているなんらかの中身を持っているか測定できます。

```php
<title>
    @hasSection('title')
        @yield('title') - App Name
    @else
        App Name
    @endif
</title>

```

#### Loops

条件文に加えて、BladeはPHPのサポートされたルーブ構造で動くために簡単なディレクティブを提供します。
もう一度言いますが、それらのディレクティブはそれぞれPHPの対応したものと同様に動きます。

```php
@for ($i = 0; $i < 10; $i++)
    The current value is {{ $i }}
@endfor

@foreach ($users as $user)
    <p>This is user {{ $user->id }}</p>
@endforeach

@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>No users</p>
@endforelse

@while (true)
    <p>I'm looping forever.</p>
@endwhile
```

ループを使うとき、ループを終わらせるもしくは現在のイテレーションをスキップする必要があるかもしれません。

```php
@foreach ($users as $user)
    @if($user->type == 1)
        @continue
    @endif

    <li>{{ $user->name }}</li>

    @if($user->number == 5)
        @break
    @endif
@endforeach
```

1行の中にディレクティブ宣言と一緒に条件を含めることもできます。

```php
@foreach ($users as $user)
    @continue($user->type == 1)

    <li>{{ $user->name }}</li>

    @break($user->number == 5)
@endforeach
```

#### Including Sub-Views

Bladeの`@include`ディレクティブは存在しているビュー内からBladeビューを簡単に含めるようにします。親ビューで利用出来るすべての変数はインクルードされたビューで利用できます。
```php
<div>
    @include('shared.errors')

    <form>
        <!-- Form Contents -->
    </form>
</div>
```

インクルードされたビューは親ビュー内で利用出来るすべての変数を継承しますが、余分なデータの配列をインクルードされたビューに渡すこともできます。

```php
@include('view.name', ['some' => 'data'])
```

注意: キャッシュされたビューの位置を参照するので、Bladeビューの中で`__DIR__`と`__FILE__`定数を使うのは避けるべきです。

#### Rendering Views For Collections

ループを結合し、Bladeの`@each`ディレクティブで1行に含めてもよいです。
```php
@each('view.name', $jobs, 'job')
```

第一引数は、配列もしくはコレクション内でそれぞれの要素をレンダリングするための部分ビューです。第二引数は、イテレートしたい配列もしくはコレクションで、一方第三引数は、ビュー内で現在のイテレーションに割り当てられる変数名です。だから、例えば、もし`jobs`の配列をイテレートしているとき、典型的に部分ビュー内で`job`変数としてそれぞれのjobにアクセスしたいでしょう。
`@each`ディレクティブに第四引数を渡すこともできます。この引数はもし与えられた配列が空である場合にレンダリングされるビューを決定します。
```php
@each('view.name', $jobs, 'job', 'view.empty')
```

#### Comments

Bladeはビュー内でコメントを定義することもできます。しかしながら、HTMLのコメントとは違って、Bladeのコメントはアプリケーションによって返されるHTML内に含まれません
```php
{{-- This comment will not be present in the rendered HTML --}}
```

## Stacks

Bladeは他のビューもしくはレイアウト内でどこか他にレンダリングされる名前付きスタックにプッシュすることもできます。
```php
@push('scripts')
    <script src="/example.js"></script>
@endpush
```

必要に応じて何度でも同じスタックにプッシュできます。スタックをレンダリングするために、`@stack`記法を使ってください。
```php
<head>
    <!-- Head Contents -->

    @stack('scripts')
</head>
```

## Service Injection

`@inject`ディレクティブはLaravelの[service container](https://laravel.com/docs/5.2/container)からサービスを検索するのに使われます。`@inject`に渡された第一引数はサービスが入るであろう変数の名前で、一方で第二引数は解決したいサービスのクラス・インターフェース名です。
```php
@inject('metrics', 'App\Services\MetricsService')

<div>
    Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
</div>
```

## Extending Blade

Bladeはあなた自身でカスタムしたディレクティブを定義することでさえできます。ディレクティブを登録するために`directive`メソッドを使えます。Bladeコンパイラがディレクティブに出会ったとき、そのパラメータと一緒に提供されたコールバックを呼びます。
次の例は、与えられた`$var`を整形する`@datetime($var)`ディレクティブを作成します。
```php
<?php

namespace App\Providers;

use Blade;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
    * Perform post-registration booting of services.
    *
    * @return void
    */
    public function boot()
    {
        Blade::directive('datetime', function($expression) {
            return "<?php echo with{$expression}->format('m/d/Y H:i'); ?>";
        });
    }

    /**
    * Register bindings in the container.
    *
    * @return void
    */
    public function register()
    {
        //
    }
}
```

分かるように、Laravelの`with`ヘルパー関数はこのディレクティブの中で使われました。`with`ヘルパーは簡単に与えられたオブジェクト・値を返し、便利なメソッドチェーンのために許可します。このディレクティブによって生成される最終的なPHPはこちらです。
```php
<?php echo with($var)->format('m/d/Y H:i'); ?>
```

Bladeディレクティブのロジックを更新した後、キャッシュされたBladeビューの全てを削除する必要があります。キャッシュされたBladeビューは`view:clear`Aritisanコマンドを使って削除できます。

