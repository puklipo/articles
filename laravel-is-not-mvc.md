---
title: "LaravelはMVCではない"
emoji: "©️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Laravel"]
published: true
---

# LaravelはMVCではない

Zennの本で書いてたけど記事にもする。

https://zenn.dev/pcs_engineer/books/re-laravel-1

https://zenn.dev/pcs_engineer/books/re-laravel-2

## 前提
MVCのことは（PCアプリ文脈のMVCとWebフレームワーク文脈のMVCの違いまで含めて）理解している前提。その上でLaravelを使う時にMVCのことは無視していいという話。

MVCを理解してない人がMVCを無視しても「間違ったMVCを覚えてる人」よりもひどいコードを書いてしまうだけ。コントローラーに書くべきコードを`@php ～ @endphp`でbladeに書いてる初心者を大量に見てきた。そんな段階の人は流石にMVCを覚えるのが先。

## Laravelのモデルはデータベース・Eloquentのこと
`Illuminate\Database\Eloquent\Model` この名前空間ですべて説明されている。
https://github.com/laravel/framework/blob/10.x/src/Illuminate/Database/Eloquent/Model.php

データベース以外の責務はない。

Eloquentは元から機能が多すぎるのでUserやPostモデルには余計な機能を増やさず宣言的に書くのがいい。リレーション・アクセッサー・スコープなどのLaravel標準機能の定義しか書かない。（他のフレームワークから来た人がよく間違えてるけどバリデーションはモデルの責務ではない）

「ビジネスロジックはモデルに書くべき」？　それはLaravelのモデルの話ではない。

## Laravelのコントローラーはルーティングの一部
`Illuminate\Routing\Controller`
https://github.com/laravel/framework/blob/10.x/src/Illuminate/Routing/Controller.php

ルートファイルに全部書く代わりにControllerクラスに書いて整理してるだけ。

> Instead of defining all of your request handling logic as closures in your route files, you may wish to organize this behavior using "controller" classes.
https://laravel.com/docs/10.x/controllers

「コントローラーには複雑な処理を書いてはいけない」？　Laravelのドキュメントにそんなことは書かれてない。
むしろ複雑な場合はシングルアクションコントローラーで作ることを推奨している。ルーティングからの処理はコントローラーに全部書いていい。同じ処理を他の場所でも使う必要が出てきた段階で別クラスに分離すればいい。（artisanコマンドなら同じ処理を使い回すことが多いだろうから最初からサービスクラスを使う例で書いてある）

Controllerという名前だけどMVVMのViewModelと考えたほうが実態に近い。ViewModelなのでビジネスロジックも書く場所。MVCと思い込むとモデルとコントローラーの役割が逆なので間違える。LaravelではViewModelなんて呼んでないので引き続きコントローラーとして扱うけど。

## MVCよりも
MVCなんてどうでもいいので「単一責任の原則」を徹底すべき。コントローラーに長い処理が書かれることがFat Controllerではない。全く関係ない処理を後からどんどん追加していった結果がFat Controller。「UserControllerに関係ないPostの処理を書いてる」例なんていくらでも実在する。

「PostServiceに分離してFat Controllerを解決した」とこれをUserControllerに書いてたら意味がない。
```php:UserController
public function postIndex(PostService $postService)
{
    $posts = $postService->list();
    return view('post.index')->with(compact('posts'));
}
```
PostControllerで直接Postモデルから取得してるほうが遥かにマシ。Laravelなら簡単にテストが書けるので最初はこれでいい。
```php:PostController
public function index()
{
    $posts = Post::latest()->paginate();
    return view('post.index')->with(compact('posts'));
}
```
これの次の段階は「スコープやトレイトなどのLaravelとPHPの機能を使って整理」

別クラスに分離するのはさらに先の段階。

昔（Laravel4から5初期）はRepositoryとかServiceとか作ってたけど今はもうそんな使い方は全くしてない。

## 他のフレームワークの話を勝手にLaravelに持ち込んではいけない
Rails(7.x)はMVCと明記しているのでMVC。
https://guides.rubyonrails.org/action_controller_overview.html

CakePHP(4.x)もMVCと明記。
https://book.cakephp.org/4/en/controllers.html
`Cake\Controller\Controller`

Laravelのドキュメントには「MVC」なんて一度も出て来ない。
