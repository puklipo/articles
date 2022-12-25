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

homeで`@include('home.post')`みたいな小さな書き間違いをするとテストは失敗する。

何かを変えた時に壊れてないことを確認するのがテスト。

