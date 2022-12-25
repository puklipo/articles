Laravel入門 いにしえの掲示板を作る パート1
----

20世紀末のインターネットではPerl CGIの掲示板が個人ページ内でのコミュニケーションの主流だった。レンタルサーバーにbbs.cgiを設置した人も多いだろう。（実際には自分で設置せずレンタルが大半）

当時くらいの掲示板を作りながらLaravelの入門レベルの説明をしていく。掲示板作りは入門にちょうどいい。最近の入門はTwitterもどきが多いけど考えれば分かるようにやってることは掲示板でもブログでもTwitterでも似たようなもの。

懐かしいと思った人でも思い浮かべる掲示板はいろいろ違うだろう。よく使われてた掲示板も今から考えれば初心者による拙い作り。実は当時でも上級者によるよくできた掲示板も存在したけど設置する側が初心者で扱える人が少なかった。今回作る掲示板は後者の仕様を思い出して作る。

## 仕様
- ユーザー登録不要で誰でも書き込める掲示板。
- 親投稿に対してコメントも書ける。
- 入力項目はタイトル・本文・名前・メール・アイコン選択・削除編集用パスワード。本文以外は省略可能。昔とは違うのでメールは表示しない。コメントはタイトルなし。
- html入力は一切不可。URL自動リンクも今回はなし。
- 表示順は新しい投稿順かつ新しいコメントが付いた順。
- 投稿したことがある人が掲示板を見に来たら「最近の訪問者」に表示。足跡機能。
- 管理者だけはユーザー登録が必要。管理者以外の登録も禁止はしない。

## 技術的な仕様
- Laravel9.x-10.x, PHP8.1-8.2。Laravelはバージョンアップでどんどん変わるので古いバージョンや将来のバージョンでは記事の内容と合わなくなる。
- スターターキットはBreeze Blade stack
- データベースは使う。
- JavaScriptは使わない。Breezeで使われてるalpine.jsだけは仕方ないので残す。
- CSSはTailwind
- 表示順はupdated_at順。
- 認証・認可機能の説明用に管理者だけ登録。

## 対象読者
この環境構築ができている人。

- [Laravel 開発環境構築 Windows版](https://github.com/pop-culture-studio/articles/blob/main/laravel-installation-windows.md)
- [Laravel 開発環境構築 Mac版](https://github.com/pop-culture-studio/articles/blob/main/laravel-installation-mac.md)

Laravelの標準的な使い方を知りたい人。

「Laravel入門」であって「プログラミング入門」ではないのでプログラミング初心者は完全に対象外。Laravelの前にPHPや他の言語で基礎を覚えるのが先に必須。

## 新規プロジェクト作成
最初に
- WindowsならWSL側のホームに作業用のディレクトリ(`/Sites/`等なんでもいい)を作る。
- Macなら元からある`/Sites/`を使う。

この部分は自由なので別にどこでもいい。説明不足な記事だといきなりホームにLaravelプロジェクトを作ってしまうことがあるので作業場所を最初に決めておく。今後のプロジェクトも全部ここに保存。

ターミナルで作業場所に移動してプロジェクト作成コマンドを実行。  

```shell
cd ~/Sites/
curl -s "https://laravel.build/laravel-bbs" | bash
```
Dockerイメージのビルドで数分かかることもあるのでしばらく待つ。`Thank you!`の表示が出れば完了。
```shell
Thank you! We hope you build something incredible. Dive in with: cd laravel-bbs && ./vendor/bin/sail up
```

新しく作られたプロジェクトのディレクトリに移動。これ以降の作業場所は`~/Sites/laravel-bbs/`
```shell
cd laravel-bbs/
```
「Laravelが正常にインストールできていること」と「ローカルのPHPが正常に動くこと」の確認にLaravelのバージョン表示。
```shell
php artisan --version
```
laravel.buildで作成した場合は全部準備万端なのでそのままsail起動可能。
```shell
sail up -d
```
ブラウザで http://localhost/ が表示できれば新規プロジェクト作成は完了。

終了はstopかdown
```shell
sail down
```

「全部sail(Docker)から使う」なんて不便なことはしないのでローカルのPHPやnpmとsailは併用していく。

### ヒント1
作成方法はlaravel.build以外にも`composer create-project laravel/laravel example-app`や`laravel/installer`を使う方法がある。それぞれ微妙に違うけど基本的にはlaravel.buildを使えばいい。

### ヒント2
laravel.buildで実際に何をやっているかはここで確認。  
https://github.com/laravel/sail-server/blob/master/resources/scripts/php.sh

### ヒント3
「Dockerを知らなくてもSailは使える」なんてことはないのでDockerについても学習する。

## インストール直後のLaravelプロジェクトを確認
ここからはPhpStormでプロジェクトを開いて進める。

掲示板の前にLaravelの考え方の説明が必要。

Laravelへの入門はまずこの段階のプロジェクトを隅々まで観察する。「Laravelはこう使うもの」という方向性を示している。

最初に見るところ

- composer.json : `"laravel/framework": "^9.19",` Laravelのメジャーバージョンを確認。新規なら最新バージョンなので関係ないけど既存のプロジェクトを見る時はどのバージョンなのかが重要。
- routes/web.php : ルーティングを見ればプロジェクトのことは大体分かる。

新規のroutes/web.php
```php
<?php

use Illuminate\Support\Facades\Route;

/*
|--------------------------------------------------------------------------
| Web Routes
|--------------------------------------------------------------------------
|
| Here is where you can register web routes for your application. These
| routes are loaded by the RouteServiceProvider within a group which
| contains the "web" middleware group. Now create something great!
|
*/

Route::get('/', function () {
    return view('welcome');
});
```
`/`は http://localhost/ の最後の`/`のこと。このルーティングは「http://localhost/ がリクエストされたら`/resouces/views/welcome.blade.php`からhtmlを作って表示する」って意味。  
ルーティングの書き方・読み方は覚えるだけなので重要ではない。

この段階で覚えるべきは「**Laravelの仕事はHTTPリクエストを受け取ってレスポンスを返す**」ってこと。これが頭にないと全体の処理の流れを把握できず何をやっているか分からなくなる。

HTTPリクエストを受け取った後最初に来るのはルーティングなのでLaravelの入り口はルーティング。

公式ドキュメントではThe Basicsの一つ目にRoutingがあるくらい重要項目。  
https://laravel.com/docs/9.x/routing

重要なルーティングだけどあまりにも基本すぎて理解せずに使ってる人は多い。書き方・読み方は簡単なので使えるけど「**Laravelの仕事はHTTPリクエストを受け取ってレスポンスを返す**」を理解してないと結局は何のためにルーティングを書いてるのか分からなくなる。

routes/web.phpで「リクエストを受け取ってレスポンスを返す」は完結している。別に全部web.phpに書いてもLaravelは使えるって認識は持っておく。でも全部web.phpに書くと不便なので別のclassに分けたいって要望から出てくるのが次のコントローラー。

### ヒント1
本当の入り口は`/public/index.php`だけど入門段階でもその後でもほとんど気にすることはないのでルーティングが入り口と思っていていい。

### ヒント2
より正確にはLaravelの入り口は2つある。

- Http : `/public/index.php`から`/routes/`→`/app/Http/`に進むHttpのルート。
- Console : artisanから`/app/Console/`に進むConsoleのルート。

HttpとConsoleは明確に分かれる。Http側からartisanコマンドの実行はできるけどConsoleからHttpのclassを使うことはない。artisanコマンドからControllerやRequestを使おうとしてるなら完全に使い方を間違えている。

HttpとConsole以外の`/app/`内のclassは両方から使える。

この辺りも理解せず使うと初心者が混乱するところ。

## コントローラー
コントローラーの前にこの辺でスターターキットbreezeをインストール。
```shell
composer require laravel/breeze --dev

php artisan breeze:install
```

web.phpが書き換わったり色々ファイルが増える。スターターキットはユーザー登録機能が用意されていて便利だけど新規のインストール直後とは変わってしまう。

breezeで増えたファイルを見ればコントローラーの使い方は分かる。

ここで覚えるべきは「**LaravelのコントローラーはContorollerという名前が付いてるだけのただのclass**」「**routes/web.phpに全部書くと長いのでclassに分けてるだけって認識**」

クロージャからコントローラーclassへの変更はPHPの`callable`が分かっていれば`callable`同士の変更だと理解しやすい。どの書き方でも第二引数は全部`callable`。
```php
Route::get('/', function () {
    return view('welcome');
});

Route::get('/', fn () => view('welcome'));

Route::get('/', TestController::class);

Route::get('/', [TestController::class, 'index']);
```

コントローラーの作成は基本的にはシングルアクションコントローラーかリソースコントローラーの2択。初心者がこれ以外のコントローラーの使い方すると一つのコントローラーにいくつもメソッドを増やしてあっという間にファットコントローラーの出来上がり。

```shell
php artisan make:controller Comment/CreateController --invokable
```
```php
use App\Http\Controllers\Comment\CreateController;

Route:get('test', CreateController::class);
```

```shell
php artisan make:controller PostController --resource
```
```php
use App\Http\Controllers\PostController;

Route:resource('post', PostController::class);
```

## ヒント1
「MVC」なんて一切気にしなくていい。Laravelのドキュメント内に「MVC」なんて言葉は一度も出て来ない。

### ヒント2
スターターキットはJetstreamもあるけど入門段階で使うには複雑すぎる。最初はbreezeのblade stackでいい。

### ヒント3
旧バージョンのLaravelではまず`php artisan make:auth`でユーザー登録機能を作成していた。その後`make:auth`が`laravel/ui`に分離された。さらにその後JetstreamとBreezeがスターターキットとして用意されたので`laravel/ui`を使うことはもうない。ころころ変わった経緯があるので古い情報に騙されないように。

### ヒント4
Laravelでは至る所で`callable`を使うので`callable`の理解は必須。

## データベースとEloquent
Perl掲示板でDB使うことはなかった。DB使うようになったのはPHP以降。

Laravelでは当然DBを使う。

LaravelでのDBはQuery BuilderとEloquentがある。本やドキュメントを途中まで読んだ初心者がQuery Builderだけ使ってる事例が多いけどもう少し先まで読もう。LaravelではまずEloquentを使うのが普通。EloquentからQuery Builderにスムーズに繋がるので実際には両方使う。

現実的にLaravelを使う時はEloquentこそが主役。「**Laravelの仕事はHTTPリクエストを受け取ってレスポンスを返す**」のレスポンスはDB内のデータから作ることが当然多い。

なのでEloquentモデルを作る時にコントローラー等の他のファイルも同時に作る使い方をする。最低でも`-m`でマイグレーションファイルは作る。これで作ればテーブル名を間違えることがない。

```shell
php artisan make:model Post --all
php artisan make:model Post -m
```

DB周りは情報量が膨大なのでここで書いても仕方がない。

「Eloquentを使う」と「リレーションを覚える」が最低でも必要なこと。

### ヒント1
初心者に多い間違い。モデルの$tableは不要。PostとpostsならLaravelの規約から自動的に決まるので書かなくていい。

```php
class Post extends Model
{
    protected $table = 'posts';
}
```

`$table`が必要なのは自動では決まらないテーブル名を使ってる時。ただしLaravelで新規に作ってるのにこんな使い方してることが間違いなので合わせたほうがいい。

```php
class Post extends Model
{
    protected $table = 'blogs';
}
```

## フロントのこと
今回はJavaScriptは使わないけどcssのために多少の説明が必要。

- JavaScript : LaravelでのJSは`/resources/js/`に置く。
- CSS : `/resources/js/` Tailwindなのでここを扱うことはほとんどない。Bladeに書く。

「JSやCSSもビルドして使うようになった」はLaravelとは関係ないフロントの常識だけどこれが分かってないとLaravelも使えない。

Bladeのcssを変更したらbuildコマンドを実行。

```shell
npm run build
```

これが面倒な場合、開発中に限りdevコマンドでもいい。
```shell
npm run dev
```
サーバーとして動き続ける。Bladeなどを変更するたびにブラウザが自動でリロードするようになる。
終了はCtrl+C。devコマンドを終了したらbuildで再生成。

今後の説明では省略するけど開発中はdevを動かしている。

### ヒント1
ビルドしたjs/cssファイルは`/public/build/`内に作られる。変更するのはresources内のファイル、`/public/build/`内は絶対に直接変更しない。buildコマンドの実行で全部上書きされるので変更してもすべて消える。

