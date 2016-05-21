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


