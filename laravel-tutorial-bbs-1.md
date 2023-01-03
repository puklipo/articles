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
- 本番環境での公開はしない。開発環境までの記事。

## 対象読者
この環境構築ができている人。昔からLaravelは開発環境構築段階で躓かないように徹底して整備しているのでここで躓く人は対象外。

- [Laravel 開発環境構築 Windows版](https://github.com/pop-culture-studio/articles/blob/main/laravel-installation-windows.md)
- [Laravel 開発環境構築 Mac版](https://github.com/pop-culture-studio/articles/blob/main/laravel-installation-mac.md)

Laravelの標準的な使い方を知りたい人。Laravelは同じ機能でも複数の書き方ができて「初心者はこう書くけど慣れた人は別の書き方する機能」が多い。

ドキュメントを読めば分かることは詳しく説明しない。

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

これ以降出てくる`sail ...`コマンドは`sail up -d`で起動した状態でしか使えない。

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

例えば.envの`DB_HOST=mysql`はDockerを知らない人が見たらなぜこれで動くのか分からない。LaravelではなくDockerの知識が必要な箇所。Laravelのドキュメントではこういう部分の説明はほとんどない。

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

一応`/public/index.php`の中身を確認すればコードからも「リクエストを受け取ってレスポンスを返す」が理解できる。
```php
$response = $kernel->handle(
    $request = Request::capture()
)->send();
```

`/public/index.php`を変更することは一切ないので頭の中に入れておくだけでいい。

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

## 初期設定
laravel.buildでプロジェクトを作った場合、設定することはほとんどないけど`config/app.php`のtimezoneだけは変更したほうがいいかもしれない。
```php
    'timezone' => 'Asia/Tokyo',
```

他は`.env`のAPP_NAMEくらい。
```
APP_NAME=Laravel
```
好みに応じてログの保存方法をdailyにしてもいい。
```
LOG_CHANNEL=daily
```

Sailを使うならこれ以外の設定は不要なので余計な変更は何もしなくていい。余計な変更をして動かなくなってる人が多い。

### ヒント1
開発中はキャッシュ系のコマンドは一度も使わなくていい。

```php
php artisan config:cache
```

一体どこの誰が教えてるんだろうってくらいひたすら間違えた使い方を続けてる人が大量にいる。「configや.envを変更する度に`config:cache`を実行」なんてことは一切しなくていい。最初から一度も実行しなければキャッシュされることはない。`config:cache`を実行するのは本番環境のみ。

### ヒント2
`env()`はconfigファイル内でしか使えない。`config:cache`でキャッシュした状態ではenv()はnullしか返さない。configファイル以外で使うとenv()は正常に動かない。

- 上のヒント1と合わせて、開発環境にてしなくていいconfig:cacheを実行してるせいでenv()がnullで正常に動かない。「キャッシュしてたらenv()は常にnull」を知らないと原因も分からない。
- 開発環境ではキャッシュしてなくてenv()は動いてたけど、本番環境でキャッシュしたら急に動かなくなった。これも知らないと原因が分からない。

`config:cache`と`env()`は特に間違えてる人が多い箇所。そして何度説明しても理解できない。初心者だけでなく稼働中の実プロダクトでも平気でenv()が使われてた例もある。本番環境でenv()が正常に動く=config:cacheしてないってことなのでこれも間違った使い方。

## エラーログ
Laravelから何かエラーが出た時は`storage/logs/laravel.log`もしくは`storage/logs/`内にログが残っているのでここを見る。

PHPコード内で`info()`などを使った時もここのログに残る。上手く動かなくて変数の中身を確認したいような時はこれでいい。
```php
info('test');
```

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

今後の説明では省略するけど開発中はdevを動かしている。JavaScriptに関しても「JavaScriptは書かないけどbreezeのBladeコンポート内で使われている」ので勝手に削除したりせずそのままの状態で使う。

### ヒント1
ビルドしたjs/cssファイルは`/public/build/`内に作られる。変更するのはresources内のファイル、`/public/build/`内は絶対に直接変更しない。buildコマンドの実行で全部上書きされるので変更してもすべて消える。

## 掲示板を作り始める
最低限説明することだけでも多い。

ここまででルーティング＆コントローラー→Eloquentモデルで必要なデータを取得→レスポンスを返す基本的な処理の流れは説明できたのでそろそろ掲示板作りへ。

コメントではない親の投稿から。

最初に決めるのはルーティング。この段階で頭の中では大まかな設計ができている。
- ルーティングは`/post`からPostController
- モデルはPost。id,title,content,name,email,icon,password
- `/`はHomeController。post.indexは使わずhomeを使う。

最初に作るのはモデル。今回は削除・編集もできるのでallで全部指定。

```shell
php artisan make:model Post --all

   INFO  Model [app/Models/Post.php] created successfully.

   INFO  Factory [database/factories/PostFactory.php] created successfully.

   INFO  Migration [database/migrations/2022_12_25_122411_create_posts_table.php] created successfully.

   INFO  Seeder [database/seeders/PostSeeder.php] created successfully.  

   INFO  Request [app/Http/Requests/StorePostRequest.php] created successfully.

   INFO  Request [app/Http/Requests/UpdatePostRequest.php] created successfully.

   INFO  Controller [app/Http/Controllers/PostController.php] created successfully.

   INFO  Policy [app/Policies/PostPolicy.php] created successfully.

```

色々生成されるけどまず見るのはmigrationsとModelsとControllers。

## postsテーブルのマイグレーション
DBの定義を決める。

Postモデルに対応するpostsテーブル用のマイグレーションが作られているので項目を追加していく。

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->id();
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('posts');
    }
};
```

開発初期段階なら別に「後から変えればいい」。
```php
    public function up()
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->id();

            $table->string('title')->nullable();
            $table->text('content');
            $table->string('name')->nullable();
            $table->string('email')->nullable();
            $table->string('icon')->nullable();
            $table->string('password')->nullable();

            $table->timestamps();
        });
    }
```

マイグレーションを実行してDBに反映。DBに接続することは必ずsailで実行する。
```shell
sail art migrate
```

PhpStormかSequel AceでDBを見て変更されてることを確認。

### ヒント1
Laravelを使う前にDBとSQLの知識も当然のように必須。

### ヒント2
開発中に最初のマイグレーションファイルを変更したくなることは多い。DBを全部消してもいい初期段階なら新しいマイグレーションを追加ではなく元のマイグレーションを変更して`migrate:refresh`コマンドで作り直してもいい。

```shell
sail art migrate:refresh
```

## Postモデル
`/app/Models/Post.php`でこの段階で必要な変更は$fillableだけ。

```php
class Post extends Model
{
    use HasFactory;

    /**
     * @var array
     */
    protected $fillable = [
        'title',
        'content',
        'name',
        'email',
        'icon',
        'password'
    ];
}
```

## ルーティング
`/routes/web.php`
```php
use App\Http\Controllers\PostController;

Route::resource('post', PostController::class);
```

postに関するルートが追加される。

```shell
php artisan route:list --path=post

  GET|HEAD        post ..................... post.index › PostController@index
  POST            post ..................... post.store › PostController@store
  GET|HEAD        post/create ............ post.create › PostController@create
  GET|HEAD        post/{post} ................ post.show › PostController@show
  PUT|PATCH       post/{post} ............ post.update › PostController@update
  DELETE          post/{post} .......... post.destroy › PostController@destroy
  GET|HEAD        post/{post}/edit ........... post.edit › PostController@edit
```

http://localhost/post を表示はできるけどControllerから何もレスポンスを返してないのでまだ何もない。

## ホームも作る
```shell
php artisan make:controller HomeController -i

   INFO  Controller [app/Http/Controllers/HomeController.php] created successfully.  
```
ルーティングは最初の`/`をHomeControllerに変更。
```php
use App\Http\Controllers\HomeController;

Route::get('/', HomeController::class)->name('home');
```

HomeControllerから`view('welcome')`を返せばとりあえず今の段階では元と同じ。後でまた変更。

```php
class HomeController extends Controller
{
    /**
     * Handle the incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function __invoke(Request $request)
    {
        return view('welcome');
    }
}
```

## ホームに投稿フォームを作る
ここまでブラウザでの画面の変化がないのでそろそろ目に見える投稿フォームを作る。

その前にbreezeのレイアウトファイルを修正。breezeはログインして使う前提なので今回のようにログイン不要で使う場合は少し困る。ログインしてなくても使えるようにapp.blade.phpを修正。

- navigationはログイン中のみ表示。
- header部分はアプリ名を表示。ここはなんでもいいので自由に変更すればいい。

`/resouces/views/layouts/app.blade.php`
```html
        <div class="min-h-screen bg-gray-100">
            @auth
                @include('layouts.navigation')
            @endauth

            <!-- Page Heading -->
                <header class="bg-white shadow">
                    <div class="max-w-7xl mx-auto py-6 px-4 sm:px-6 lg:px-8">
                        <h2 class="font-semibold text-xl text-gray-800 leading-tight">
                            <a href="{{ route('home') }}">
                                {{ config('app.name') }}
                            </a>
                        </h2>
                    </div>
                </header>

            <!-- Page Content -->
            <main>
                {{ $slot }}
            </main>
        </div>
```

ホーム用のviewはdashboard.blade.phpをコピーしてhome.blade.phpを作る。
```html
<x-app-layout>
    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 text-gray-900">
                    {{ __("Home!") }}
                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```

HomeControllerでhome.blade.phpを返す。
```php
class HomeController extends Controller
{
    public function __invoke(Request $request)
    {
        return view('home');
    }
}
```

ログインしてなくても http://localhost/ が表示できてやっとhome.blade.phpに投稿フォームを書く準備ができた。

フォームはregister.blade.phpなどを参考に。細かい見た目は後からいくらでも変更できるので優先度は低い。
```html
<x-app-layout>
    <div class="py-12">
        <div class="max-w-5xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 text-gray-900">
                    <form method="POST" action="{{ route('post.store') }}">
                        @csrf

                        <!-- Title -->
                        <div class="mt-4">
                            <x-input-label for="title" :value="__('タイトル')" />
                            <x-text-input id="title" class="block mt-1 w-full" type="text" name="title" :value="old('title')" autofocus />
                            <x-input-error :messages="$errors->get('title')" class="mt-2" />
                        </div>

                        <!-- Content -->
                        <div class="mt-4">
                            <x-input-label for="content" :value="__('メッセージ')" />
                            <textarea id="content" class="block mt-1 w-full border-gray-300 focus:border-indigo-500 focus:ring-indigo-500 rounded-md shadow-sm" type="text" name="content" required>{{ old('content') }}</textarea>
                            <x-input-error :messages="$errors->get('content')" class="mt-2" />
                        </div>

                        <!-- Name -->
                        <div class="mt-4">
                            <x-input-label for="name" :value="__('名前')" />
                            <x-text-input id="name" class="block mt-1 w-full" type="text" name="name" :value="old('name', request()->cookie('name'))" />
                            <x-input-error :messages="$errors->get('name')" class="mt-2" />
                        </div>

                        <!-- Email Address -->
                        <div class="mt-4">
                            <x-input-label for="email" :value="__('メール（公開されません）')" />
                            <x-text-input id="email" class="block mt-1 w-full" type="email" name="email" :value="old('email', request()->cookie('email'))" />
                            <x-input-error :messages="$errors->get('email')" class="mt-2" />
                        </div>

                        <!-- Password -->
                        <div class="mt-4">
                            <x-input-label for="password" :value="__('削除用パスワード')" />

                            <x-text-input id="password" class="block mt-1 w-full"
                                          type="password"
                                          name="password"
                                          autocomplete="password" />

                            <x-input-error :messages="$errors->get('password')" class="mt-2" />
                        </div>

                        <div class="flex items-center justify-end mt-4">
                            <x-primary-button class="ml-4">
                                {{ __('送信') }}
                            </x-primary-button>
                        </div>
                    </form>

                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```

アイコン選択はまだないけど投稿フォームはできた。次は送信後の保存。

### ヒント1
LaravelでのURLは`action="{{ route('post.store') }}"`のように`route()`を使うことがほとんど。

たまに`{{ action([PostController::class, 'store']) }}`みたいなaction()を使ってる初心者を見るけどどこでこんな変な使い方を覚えるのか分からない。action()なんて今まで一度も使ったことないくらい全く使わない。

### ヒント2
Laravelでは「結果は同じだけど複数の書き方ができる」ってことが多い。Laravelを使い込むほど効率的な書き方を選ぶようになる。

## PostControllerのstore()
新規投稿の場合、フォームから送信されたデータはstore()でDBに保存。

書き方はいろいろあるけどcreate()を使うのが一番いい。ここではpasswordを暗号化するため先にmergeでpasswordを変更してから保存している。
```php
    /**
     * Store a newly created resource in storage.
     *
     * @param  \App\Http\Requests\StorePostRequest  $request
     * @return \Illuminate\Http\Response
     */
    public function store(StorePostRequest $request)
    {
        $request->merge([
            'password' => bcrypt($request->input('password')),
        ]);

        Post::create($request->only([
            'title',
            'content',
            'name',
            'email',
            'icon',
            'password',
        ]));

        cookie()->queue('name', $request->input('name'), 60*24*30);
        cookie()->queue('email', $request->input('email'), 60*24*30);

        return back();
    }
```

Postモデル作成時にallを指定したのでStorePostRequestも自動で作られている。バリデーションはStorePostRequestで行うのでこっちも変更が必要。

今回はログインしてなくても投稿できるのでauthorize()をtrueに。今回に限らずここのauthorize()で判定することは少ない。ログインしてるかどうかはもっと前の段階で判定。
```php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rules\Password;

class StorePostRequest extends FormRequest
{
    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array<string, mixed>
     */
    public function rules()
    {
        return [
            'title' => ['nullable', 'string', 'max:255'],
            'content' => ['required', 'string', 'max:4000'],
            'name' => ['nullable', 'string', 'max:255'],
            'email' => ['nullable', 'string', 'email', 'max:255'],
            'icon' => ['nullable', 'string', 'max:255'],
            'password' => ['nullable', Password::defaults()],
        ];
    }
}
```

これでブラウザから投稿してDBに保存まではできた。まだ表示はないのでDB内を直接見れば投稿できてるか確認できる。

### ヒント1
create()で作成するためにはPostモデルの$fillableでの許可が必要。許可したカラムだけ入力できる。この辺りQuery BuilderとEloquentの違いを理解してないと予期しない動作になって危険。$fillableはEloquentの機能。

### ヒント2
初心者にものすごく多い間違い。`$request->all()`は厳禁。$fillableがあるとはいえall()を使ってるような初心者では必ず危険な状態になる。
```php
        Post::create($request->all());
```

`$request->only()`を使うか一つずつ指定。

```php
       Post::create([
            'title' => $request->input('title'),
            'content' => $request->input('content'),
            'name' => $request->input('name'),
            'email' => $request->input('email'),
            'icon' => $request->input('icon'),
            'password' => bcrypt($request->input('password')),
       ]);
```

バリデーション後なので`$request->validated()`なら許容範囲だけど使ってもいい場面かはよく考える必要がある。

## 投稿の表示
パート1の最後に表示までは完成させる。

HomeController。updated_at順でページネーション用に取得するごく普通のコード。
```php
namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\Request;

class HomeController extends Controller
{
    /**
     * Handle the incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function __invoke(Request $request)
    {
        $posts = Post::latest('updated_at')->paginate();

        return view('home')->with(compact('posts'));
    }
}
```

home.blade.phpの後半に投稿の表示部分を追加。ここも見た目はどうでもいい。好きな見た目にすればいい。  
htmlは許可しないけど改行だけは反映させるために`{!! nl2br(e($post->content)) !!}`ではe()でエスケープ→nl2br()で改行をbrタグに→{!! !!}でエスケープしない表示。
```php
    <div class="py-12">
        <div class="max-w-5xl mx-auto sm:px-6 lg:px-8">
            @forelse($posts as $post)
                <div class="flex flex-row">
                    <div class="p-6 my-3 w-1/4">
                        <div class="text-lg font-bold">{{ $post->name ?? 'NO NAME' }}</div>

                        <time class="mt-6" datetime="{{ $post->created_at }}">{{ $post->created_at }}</time>
                    </div>
                    <div class="grow my-3 bg-white overflow-hidden shadow-sm sm:rounded-lg">
                        <div class="p-6 text-lg text-gray-900">
                            <h2 class="font-bold inline">{{ $post->title ?? 'NO TITLE' }}</h2><span
                                class="text-gray-300 font-medium ml-3">#{{ $post->id }}</span>
                            <p class="py-2">
                                {!! nl2br(e($post->content)) !!}
                            </p>
                        </div>
                    </div>
                </div>
            @empty
                投稿はまだありません。
            @endforelse

            {{ $posts->links() }}
        </div>
    </div>
```

### ヒント1
`compact()`はPHPの標準関数。`['posts' => $posts]`と同じ意味。Laravelでコントローラーからviewに変数を渡す時などにcompact()はよく使う。別に使わなくてもいい。

これでも同じ。
```php
return view('home')->with(['posts' => $posts]);
```

### ヒント2
「{!! !!}とe()」はLaravelの機能、nl2br()はPHPの機能。普通に混在して使うのでPHPの知識も当然のように必須。

### ヒント3
Laravelはviewのbladeファイルを作る部分は簡単にする機能はないので自分でファイルを作るか他のファイルをコピーする。中のhtmlはスターターキットを使ってるなら既存のhtmlを真似すればいい。

htmlのサポートが控えめなのは「レスポンスとして返すのはhtmlに限らない」から。  
Eloquentモデルをそのまま返した場合、自動的にjsonに変換されてレスポンスとして返す。
```php
class HomeController extends Controller
{
    public function __invoke(Request $request)
    {
        $posts = Post::latest('updated_at')->paginate();

        return $posts;
    }
}
```
Laravel以外のフレームワークではコントローラーとviewが自動的に決まるものもあるけどそれはレスポンスをhtmlと決め打ちしている。「jsonを返すAPIとしてだけ使う」事例も多いだろう現代ではhtmlに決め打ちせず何を返すかはコントローラーで毎回指定するほうがいい。

## 処理の流れを振り返る
http://localhost/ へのGETリクエスト  
→ルーティングのRoute::get('/')  
→HomeControllerにてPostのデータをhome.blade.phpに渡して作ったhtmlをレスポンスとして返す。

レスポンスを返したら処理の流れは一度終了。ここから次の流れにも続いてると考えると認識を間違える。

homeの投稿フォームからroute('post.store')にPOSTリクエストを送信  
→ルーティングのRoute::resource('post')の内のpost.store  
→PostControllerのstore()でPostモデルに保存。  
→back()で前のページに戻る。

「前のページに戻る」がここでのレスポンス。レスポンスを返したのでここは終了。  

前のページは http://localhost/ なので次の処理は「http://localhost/ へのGETリクエスト」で最初に戻る。

どこからどこまでが一連の処理の流れかを意識すれば「リクエストを受け取ってレスポンスを返すまで」の基本中の基本を間違えない。

## パート2へ
[パート2](https://github.com/pop-culture-studio/articles/blob/main/laravel-tutorial-bbs-2.md)ではテストと投稿へのコメントを作っていく。
