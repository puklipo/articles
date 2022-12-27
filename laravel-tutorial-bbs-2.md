Laravel入門 いにしえの掲示板を作る パート2
----

[パート1](https://github.com/pop-culture-studio/articles/blob/main/laravel-tutorial-bbs-1.md)の続き。

パート1ではとりあえず動くものができるまでを優先したけどパート2まで進んだのでまずは現代の常識としてテストを書く。

## テストを書く意味
Laravel9とPHP8.1で掲示板を作っている今書いてるこのコードは問題なく動いてる。でもLaravel10やPHP8.2にバージョンアップしたら？

作ってる今はブラウザを操作して動作確認してるけど数年後でも同じことはできない。人間が手動で操作してることを自動化するのがテスト。

滅多にないけどLaravel9.10.0から9.11.0へのバージョンアップで急に壊れるようなこともある。この場合はすぐに修正された9.11.1がリリースされるけど修正前のバージョンでデプロイしたサイトは壊れて大慌て。デプロイ前にテストして壊れてないことを確認するためのテスト。

## Sail環境でのテスト
sailインストール時にphpunit.xmlが変更されている。

`sail test`だけ使うならこれでいいけどPhpStormからテストを実行したいので元に戻す。
```xml
        <env name="DB_DATABASE" value="testing"/>
```

変更後のphpunit.xml。インメモリSQLiteを使う。本物のDBを使わないのでsail内でも外でも使えて速い。ただしSQLiteなので若干癖がある。SQLiteではテストできない使い方をしている場合は本物のDBを使う。
```xml
        <env name="DB_CONNECTION" value="sqlite"/>
        <env name="DB_DATABASE" value=":memory:"/>
```

`/tests/`内のExampleTest.phpを少し変更。`use RefreshDatabase;`を追加。

```php
namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    use RefreshDatabase;

    /**
     * A basic test example.
     *
     * @return void
     */
    public function test_the_application_returns_a_successful_response()
    {
        $response = $this->get('/');

        $response->assertStatus(200);
    }
}
```
PhpStormからphpunitを実行か、`sail test`を実行すればExampleTestとbreezeでインストールされたテストが全部成功する。ここで失敗するならどこかの設定を間違えている。

## ホームのテスト
ExampleTestをそのままホーム用のテストにする。

まずはファクトリーを作る。  
ファクトリー：テスト用にデータベースにダミーデータを作る機能。

Postモデルを作った時の--allでPostFactoryも作られている。  
`database/factories/PostFactory.php`
```php
class PostFactory extends Factory
{
    /**
     * Define the model's default state.
     *
     * @return array<string, mixed>
     */
    public function definition()
    {
        return [
            'title' => fake()->text(),
            'content' => fake()->text(),
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'password' => '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', // password
        ];
    }
}
```
ファクトリーを使うようにExampleTestを変更。  
ファクトリーによって「ダミーデータを20件DBに追加」するのでpostsテーブルにデータがある状態でテストができる。  
一つ目のテストはファクトリーを使ってないので投稿がないことをテスト。2つ目は投稿があることをテスト。
```php
    public function test_the_application_returns_a_successful_response()
    {
        $response = $this->get('/');

        $response->assertStatus(200)
                 ->assertSeeText('投稿はまだありません');
    }

    public function test_home_with_posts()
    {
        Post::factory()->count(20)->create();

        $response = $this->get('/');

        $response->assertStatus(200)
                 ->assertDontSeeText('投稿はまだありません')
                 ->assertViewHas('posts', fn ($posts) => $posts->total() === 20);

        $this->assertDatabaseCount('posts', 20);
    }
```

## home.blade.php を分割
homeが長くなってるので別ファイルに分割。

投稿フォームは`resources/views/home/form.blade.php`
```php
<form method="POST" action="{{ route('post.store') }}">
    @csrf

    <!-- Title -->
    <div class="mt-4">
        <x-input-label for="title" :value="__('タイトル')"/>
        <x-text-input id="title" class="block mt-1 w-full" type="text" name="title"
                      :value="old('title')" autofocus/>
        <x-input-error :messages="$errors->get('title')" class="mt-2"/>
    </div>

    <!-- Content -->
    <div class="mt-4">
        <x-input-label for="content" :value="__('メッセージ')"/>
        <textarea id="content"
                  class="block mt-1 w-full border-gray-300 focus:border-indigo-500 focus:ring-indigo-500 rounded-md shadow-sm"
                  type="text" name="content" required>{{ old('content') }}</textarea>
        <x-input-error :messages="$errors->get('content')" class="mt-2"/>
    </div>

    <!-- Name -->
    <div class="mt-4">
        <x-input-label for="name" :value="__('名前')"/>
        <x-text-input id="name" class="block mt-1 w-full" type="text" name="name"
                      :value="old('name', request()->cookie('name'))"/>
        <x-input-error :messages="$errors->get('name')" class="mt-2"/>
    </div>

    <!-- Email Address -->
    <div class="mt-4">
        <x-input-label for="email" :value="__('メール（公開されません）')"/>
        <x-text-input id="email" class="block mt-1 w-full" type="email" name="email"
                      :value="old('email', request()->cookie('email'))"/>
        <x-input-error :messages="$errors->get('email')" class="mt-2"/>
    </div>

    <!-- Password -->
    <div class="mt-4">
        <x-input-label for="password" :value="__('削除用パスワード')"/>

        <x-text-input id="password" class="block mt-1 w-full"
                      type="password"
                      name="password"
                      autocomplete="password"/>

        <x-input-error :messages="$errors->get('password')" class="mt-2"/>
    </div>

    <div class="flex items-center justify-end mt-4">
        <x-primary-button class="ml-4">
            {{ __('送信') }}
        </x-primary-button>
    </div>
</form>
```

postsは`resources/views/home/posts.blade.php`
```php
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
```

homeは
```php
<x-app-layout>
    <div class="py-12">
        <div class="max-w-5xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 text-gray-900">
                    @include('home.form')
                </div>
            </div>
        </div>
    </div>

    <div class="py-12">
        <div class="max-w-5xl mx-auto sm:px-6 lg:px-8">
            @include('home.posts')
        </div>
    </div>
</x-app-layout>
```

ブラウザで見る前にテストを実行してもbladeを分割しただけなのでテストは失敗しない。

home.blade.phpで`@include('home.post')`みたいな小さな書き間違いをするとテストは失敗する。

何かを変えた時に壊れてないことを確認するのがテスト。

## 投稿のテスト
新しいテストを作る時は別のテストファイルをコピーしてもいいし、artisanコマンドで作ってもいい。
```shell
php artisan make:test PostTest
```

投稿に成功するテストとcontentが空で失敗するテスト。
```php
namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\WithFaker;
use Tests\TestCase;

class PostTest extends TestCase
{
    use RefreshDatabase;

    public function test_store_successful()
    {
        $response = $this->post(route('post.store'), [
            'title' => 'test title',
            'content' => 'test content',
            'name' => 'test name',
            'email' => 'test@localhost',
            'password' => 'password',
        ]);

        $response->assertRedirect();

        $this->assertDatabaseCount('posts', 1)
             ->assertDatabaseHas('posts', [
                 'title' => 'test title',
                 'content' => 'test content',
                 'name' => 'test name',
                 'email' => 'test@localhost',
             ]);
    }

    public function test_store_invalid()
    {
        $response = $this->post(route('post.store'), [
            'title' => 'test title',
            'content' => '',
            'password' => 'password',
        ]);

        $response->assertRedirect()
                 ->assertInvalid(['content']);

        $this->assertDatabaseCount('posts', 0)
             ->assertDatabaseMissing('posts', [
                 'title' => 'test title',
             ]);
    }
}
```

ホームの表示と投稿のテストはできたのでひとまずはこのくらいでいい。

PhpStormのカバレッジ付きでテストを実行すればHomeControllerやPostController::store()は緑になっている。緑の箇所は成功するテストで一度は通っている。

Laravelの開発作業はコードを書く→テストで緑にするの繰り返し。

### ヒント1
「ステップ実行しながら変数の中身を見る」なんてことはしない。ステップ実行は今のバージョンで問題ないことしか証明できない。

自分で書いたコードなら変数の中身は分かっているので確認する必要がない。一時的に確認する時は`dd()` `dump()` `info()`などを使えばいい。

## 投稿へのコメント機能を作る
まずはざっと設計を考える。

- ルーティングは`/post/{post}/comment`からCommentController
- 表示はhomeで投稿ごとに全コメントを表示。個別のコメント表示はしない。
- モデルはComment
- テーブルはcomments。post_id,content,name,email,icon,password
- 今回は簡単にするため1行コメントにする。
- passwordは必須にする。投稿のpasswordも後で必須に変更。でも今回は編集・削除機能までは作らない。
- PostとCommentは一対多

作るのはモデルから。今回もallで全部作っている。
```shell
php artisan make:model Comment --all
```

commentsのマイグレーション。postsとほぼ同じなのでコピペして外部キー部分を追加。
```php
    public function up()
    {
        Schema::create('comments', function (Blueprint $table) {
            $table->id();

            $table->foreignId('post_id')
                  ->constrained()
                  ->cascadeOnUpdate()
                  ->cascadeOnDelete();

            $table->string('content');
            $table->string('name')->nullable();
            $table->string('email')->nullable();
            $table->string('icon')->nullable();
            $table->string('password');

            $table->timestamps();
        });
    }
```

実行。
```shell
sail art migrate
```

Commentモデルの$fillableもPostとtitle以外同じ。
```php
class Comment extends Model
{
    use HasFactory;

    /**
     * @var array
     */
    protected $fillable = [
        'content',
        'name',
        'email',
        'icon',
        'password'
    ];
}
```

### ヒント1
今のLaravelなら外部キーは簡潔に書ける。でもLaravelでの書き方だけ知っていても意味がない。「データベースの外部キーとは何か」はLaravel以前の段階で知っておくこと。こういうLaravelを使おうとしてる人なら当然知ってるだろう常識はドキュメントには書かれてない。

## PostとCommentのリレーション
Laravelの入門段階の場合、リレーションの理解が最も重要。 
重要だけど一番複雑だしドキュメントも長い。  
https://laravel.com/docs/9.x/eloquent-relationships

何年使ってても暗記は無理なので「常にドキュメントを見ながら使う」でいい。

とはいえ今回のPostとCommentのリレーションは頻繁に使うのですぐに覚えられる。

- 一つのPostは多数のCommentを持つ。一対多のリレーション。
- 逆にあるCommentは一つのPostにしか属してない。
- 「どのPostか」の情報であるpost_idはCommentが持つ。

Postモデル。多数のCommentを持つので複数形のcomments
```php
    /**
     * @return \Illuminate\Database\Eloquent\Relations\HasMany
     */
    public function comments()
    {
        return $this->hasMany(Comment::class);
    }
```

Commentモデル。一つのPostに属しているので単数形のpost。`belongTo`は「属している」とか「所有されている」とかの意味。
```php
    /**
     * @return \Illuminate\Database\Eloquent\Relations\BelongsTo
     */
    public function post()
    {
        return $this->belongsTo(Post::class);
    }
```

Eloquentモデルに書いてるのはリレーションを使うための定義。

使い方  
ある一つのPostが持つCommentを新しい順に取得。
```php
$post = Post::find(1);
$comments = $post->comments()->latest()->get();
```
「リレーションの定義」ではなくPHPで考えれば「Post classにcomments()メソッドを追加」したので`$post->comments()`の使い方に違和感はない。

`$post->comments()`の動作は`Comment::where('post_id', $post->id)`と大体同じ。  
こう書いても結果は同じ。リレーションを知らない初心者はこの使い方をする。
```php
$comments = Comment::where('post_id', $post->id)->latest()->get();
```
結果が同じなのにリレーションを使う理由はこの先の使い方ではリレーションを使うほうが簡単だから。大して変わらないコードで結果も同じなのはこの段階だけの話。

### ヒント1
「Laravelのリレーション機能」だけ見ればPHPのコードで何かやっているけどLaravel内部ではデータベースとSQLで実現している。Laravelの機能は「複雑な機能を簡単に使えるようにしているだけ」なこともある。この辺りを理解してないともっと簡単に使えるのに無駄に難しい使い方をしてしまう。

### ヒント2
Laravelというかプログラミングの基礎として単数・複数、大文字・小文字、半角・全角などの細かい所まで気を払わなければならない。これができてない人にプログラミングは不可能。

### ヒント3
複数の書き方ができるLaravelの厄介なところで`()`なしの`$post->comments`で使う方法もある。動作としてはlatest()などの条件を指定しない場合と同じと思えばいい。
```php
$comments = $post->comments;
$comments = $post->comments()->get();
```
これを`()`の有無だけの違いで覚えるとLaravelでしか役に立たない知識で終わるので`$post->comments`で使った時に実際にどんな動作をするかは把握しておく。

## Commentのルーティング
コメントは投稿に対して送信するだけなのでonlyでstoreだけ定義。
```php
use App\Http\Controllers\CommentController;

Route::resource('post.comment', CommentController::class)
     ->only(['store']);
```

コメントは常にどれかのPostに属しているのでネストしたリソースコントローラーとして`'post.comment'`で定義。

```shell
php artisan route:list --path=post/{post}/comment

  POST       post/{post}/comment post.comment.store › CommentController@store
```

CommentControllerの修正。post部分を追加。store()以外も修正が必要だけど今回は使わないのでそのまま、もしくはもう削除してもいい。
```php
use App\Models\Post;

//

    /**
     * Store a newly created resource in storage.
     *
     * @param  StoreCommentRequest  $request
     * @param  Post  $post
     * @return Response
     */
    public function store(StoreCommentRequest $request, Post $post)
    {
        //
    }
```

### ヒント1
`/post/1/comment`へのリクエストでは`public function store(Post $post)`に来た時点で自動的に「id=1の$post」が入ってくる。これもLaravelでは当然のように使う機能。

こんな使い方はしない。
```php
public function store($id)
{
    $post = Post::find($id);
}
```

## Commentの投稿
ここはPostと同じなのでざっと。

- home.posts の各投稿にコメントのフォームを作る。
- CommentControllerのstore()でコメントの保存。
- home.posts でコメントの表示。

最初から別ファイルに分割。  
`route('post.comment.store', $post)`は普通に書いてるけど間違った使い方しやすい箇所。`route('post.comment.store', ['post' => $post->id])`みたいな使い方してる例が多いけどもっと簡単に`$post`を渡せばいい。
`resources/views/home/comment.blade.php`
```php
<h3 class="text-lg font-bold">コメント</h3>

<ul class="ml-3 space-y-1">
    @forelse($post->comments as $comment)
        <li>
            <span class="font-bold">{{ $comment->name ?? 'NO NAME' }}</span>
            <span class="mx-1">『{{ $comment->content }}』</span>
            <time class="text-gray-200" datetime="{{ $comment->created_at }}">{{ $comment->created_at }}</time>
        </li>
    @empty
        <span class="text-gray-300">コメントはありません。</span>
    @endforelse
</ul>

<details class="mt-3">
    <summary>コメントを書く</summary>
    <div class="p-6 bg-gray-100">
        <form method="POST" action="{{ route('post.comment.store', $post) }}">
            @csrf

            <!-- Content -->
            <div class="mt-4">
                <x-input-label for="content" :value="__('コメント')"/>
                <x-text-input id="content"
                              class="block mt-1 w-full"
                              type="text" name="content" :value="old('content')" required/>
                <x-input-error :messages="$errors->get('content')" class="mt-2"/>
            </div>

            <!-- Name -->
            <div class="mt-4">
                <x-input-label for="name" :value="__('名前')"/>
                <x-text-input id="name" class="block mt-1 w-full" type="text" name="name"
                              :value="old('name', request()->cookie('name'))"/>
                <x-input-error :messages="$errors->get('name')" class="mt-2"/>
            </div>

            <!-- Email Address -->
            <div class="mt-4">
                <x-input-label for="email" :value="__('メール（公開されません）')"/>
                <x-text-input id="email" class="block mt-1 w-full" type="email" name="email"
                              :value="old('email', request()->cookie('email'))"/>
                <x-input-error :messages="$errors->get('email')" class="mt-2"/>
            </div>

            <!-- Password -->
            <div class="mt-4">
                <x-input-label for="password" :value="__('削除用パスワード')"/>

                <x-text-input id="password" class="block mt-1 w-full"
                              type="password"
                              name="password"
                              required
                              autocomplete="password"/>

                <x-input-error :messages="$errors->get('password')" class="mt-2"/>
            </div>

            <div class="flex items-center justify-end mt-4">
                <x-primary-button class="ml-4">
                    {{ __('送信') }}
                </x-primary-button>
            </div>

        </form>
    </div>
</details>
```

`resources/views/home/posts.blade.php`
```php
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

            <div class="p-6 border-t">
                @include('home.comment')
            </div>
        </div>
    </div>
@empty
    投稿はまだありません。
@endforelse

{{ $posts->links() }}
```

`app/Http/Requests/StoreCommentRequest.php`
```php
    public function rules()
    {
        return [
            'content' => ['required', 'string', 'max:255'],
            'name' => ['nullable', 'string', 'max:255'],
            'email' => ['nullable', 'string', 'email', 'max:255'],
            'icon' => ['nullable', 'string', 'max:255'],
            'password' => ['required', Password::defaults()],
        ];
    }
```

CommentControllerのstore()。Postとほぼ同じ。リレーションを使っているので`$post->comments()->create()`だけでpost_idが自動的にセットされた新しいCommentが保存される。
```php
    /**
     * Store a newly created resource in storage.
     *
     * @param  StoreCommentRequest  $request
     * @param  Post  $post
     * @return Response
     */
    public function store(StoreCommentRequest $request, Post $post)
    {
        $request->merge([
            'password' => bcrypt($request->input('password')),
        ]);

        $post->comments()->create($request->only([
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

Laravelではこういう使い方をして欲しいけどLaravelが勝手に色々やってくれて実現してる機能なので内部で何が起きてるのか理解してないと中々使えない。

HomeControllerでPost取得時にコメントも新しい順に取得。ここもリレーションの機能。Query Builderで考えるとjoinとか使いそうになるけどEloquentのリレーションでの使い方はこうなる。
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
        $posts = Post::with([
            'comments' => fn ($comment) => $comment->latest(),
        ])->latest('updated_at')
          ->paginate();

        return view('home')->with(compact('posts'));
    }
}
```

各投稿ごとにcommentsを持った結果を得られる。コードで書いたほうが分かりやすい。
```php
foreach($posts as $post) {
    foreach($post->comments as $comment) {
        $comment->name;
    }
}
```

### ヒント1
厳密にはwith()なしでもpost->commentsは使えるけどN+1問題が発生するのでwithなどでEagerロードするのが基本。

## Commentが送信されたら親の投稿の更新時間も変更
Commentモデル。リレーションのpostの更新時間も更新する定義。
```php
    /**
     * @var array
     */
    protected $touches = ['post'];
```

これで親の投稿時間順もしくはコメントが付いた順での表示。

ブラウザで親の投稿を書いたりコメントを書いたりして動作確認する。自動テストは書くけど手動での動作確認もする。テストで作ったデータはすぐ消えるのである程度データがある状態を残すには手動で操作も必要。

## コメントのテスト
この時点で元のテストが失敗しないことを確認してからコメントのテストを書く。

`database/factories/CommentFactory.php`
```php
    public function definition()
    {
        return [
            'content' => fake()->text(),
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'password' => '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', // password
        ];
    }
```

`tests/Feature/CommentTest.php`はPostTestをコピーでもいい。
```php
class CommentTest extends TestCase
{
    use RefreshDatabase;

    public function test_posts_with_comments()
    {
        Post::factory()->count(20)->hasComments(5)->create();

        $response = $this->get('/');

        $response->assertSuccessful()
                 ->assertDontSeeText('コメントはありません')
                 ->assertViewHas('posts', fn ($posts) => $posts->first()->comments()->count() === 5);

        $this->assertDatabaseCount('comments', 100);
    }

    public function test_store_successful()
    {
        $post = Post::factory()->create();

        $response = $this->post(route('post.comment.store', $post), [
            'content' => 'test content',
            'name' => 'test name',
            'email' => 'test@localhost',
            'password' => 'password',
        ]);

        $response->assertRedirect();

        $this->assertDatabaseCount('comments', 1)
             ->assertDatabaseHas('comments', [
                 'post_id' => $post->id,
                 'content' => 'test content',
                 'name' => 'test name',
                 'email' => 'test@localhost',
             ]);
    }

    public function test_store_invalid()
    {
        $post = Post::factory()->create();

        $response = $this->post(route('post.comment.store', $post), [
            'content' => '',
            'password' => '',
        ]);

        $response->assertRedirect()
                 ->assertInvalid(['content', 'password']);

        $this->assertDatabaseCount('comments', 0)
             ->assertDatabaseMissing('comments', [
                 'content' => '',
             ]);
    }
}
```

CommentControllerのstore()が緑なら十分。

## 投稿のpasswordも必須に変更
ここまで来るとDBにデータが増えていて全部消してやり直しはしたくない。以降は新しいマイグレーションを作ってDBを変更する。本番環境では当然消せないので常に新しいマイグレーションを作るのが本来の使い方。元のマイグレーションを編集して全部やり直しが可能なのは開発初期段階だけ。

DBのカラムを変更するには先にcomposerで`doctrine/dbal`のインストールが必要。
```shell
composer require doctrine/dbal
```

マイグレーションを作って
```shell
php artisan make:migration change_password_posts_table
```

nullableでないように変更。up()で変更と、down()で「変更を元に戻す時の定義」も必要。
```php
return new class extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::table('posts', function (Blueprint $table) {
            $table->string('password')->nullable(false)->change();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::table('posts', function (Blueprint $table) {
            $table->string('password')->nullable()->change();
        });
    }
};
```

sailで実行。
```shell
sail art migrate
```

StorePostRequestのバリデーションでpassword必須にする。
```php
    public function rules()
    {
        return [
            'title' => ['nullable', 'string', 'max:255'],
            'content' => ['required', 'string', 'max:4000'],
            'name' => ['nullable', 'string', 'max:255'],
            'email' => ['nullable', 'string', 'email', 'max:255'],
            'icon' => ['nullable', 'string', 'max:255'],
            'password' => ['required', Password::defaults()],
        ];
    }
```

テストはこれまでもpasswordを指定していたので失敗はしない。

一応PostTestにpasswordがない場合のテストを追加。
```php
    public function test_store_invalid_missing_password()
    {
        $response = $this->post(route('post.store'), [
            'content' => 'test',
            'password' => null,
        ]);

        $response->assertRedirect()
                 ->assertInvalid(['password']);

        $this->assertDatabaseCount('posts', 0)
             ->assertDatabaseMissing('posts', [
                 'content' => 'test',
             ]);
    }
```

DBへの追加や変更は必ず発生するのでマイグレーションで変更もLaravelでの開発の日常作業。

## パート3へ
掲示板としてはある程度の機能ができたので公開するつもりのサイトならこの時点でもう公開する。削除・編集機能なんていざとなればDBを直接触ればいいという割り切り。
