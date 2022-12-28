Laravel入門 いにしえの掲示板を作る パート3
----

[パート2](https://github.com/pop-culture-studio/articles/blob/main/laravel-tutorial-bbs-2.md)の続き。

ここからは管理者を作って管理機能を作る。

## ユーザー登録
http://localhost/register で新規ユーザーを作る。Laravelのスターターキットを使えば登録機能は最初から揃ってるのでもはや説明する必要もない。

Breeze(Blade)では登録処理は `app/Http/Controllers/Auth/RegisteredUserController.php` なので必要なら確認。

登録後はそのままログイン済の状態でhttp://localhost/dashboard に移動する。dashboardはログインしてないと表示できない。

DBではid=1のUserが作られた状態。このUserが管理者かどうかなんて情報は持ってない。

今ブラウザで操作してるのは「登録時のメールとパスワードでログインしたid=1のユーザーである」ということしか見てない。

これは「認証」機能。認証で管理者かどうかを判断しようとすると壮大に使い方を間違える。

### ヒント1
ログアウト後に再度ログインする時は http://localhost/login 。今回の掲示板では管理者しか登録しないのでURLをどこかからリンクする必要はない。管理者だけが知っていればいい。でもLaravelのスターターキットを使うと同じURLなのでそのまま残しているとbotによる不正な登録が発生するかもしれない。

管理者用のユーザーを登録後は`routes/auth.php`のregisterを無効にしてもいい。
```php
    // Route::get('register', [RegisteredUserController::class, 'create'])
    //            ->name('register');

    // Route::post('register', [RegisteredUserController::class, 'store']);
```

## 認可機能で管理者を判定する
「Userのidが1だったらadminとして許可する」

`app/Providers/AuthServiceProvider.php`
```php
namespace App\Providers;

use App\Models\User;
use Illuminate\Support\Facades\Gate;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

class AuthServiceProvider extends ServiceProvider
{
    /**
     * The model to policy mappings for the application.
     *
     * @var array<class-string, class-string>
     */
    protected $policies = [
        // 'App\Models\Model' => 'App\Policies\ModelPolicy',
    ];

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('admin', fn (User $user) => $user->id === 1);
    }
}
```
一人ならこれで十分。`$user->email === ''`でメールで判定してもいい。

認証は「ログイン済のユーザーかどうかを見る」。  
認可はさらに「そのログイン済ユーザーに対して○○を許可するかを見る」

簡単な確認方法はbladeにこう書けば管理者のみに表示される。
```php
@can('admin')
   admin
@endcan
```

認可による判定はルーティングでもコントローラーでもbladeでもあらゆる所で可能なので「管理者にだけこの機能を許可する」は認可で行う。

まずは `routes/web.php` でdashboardの表示を管理者のみ許可にしておく。「ログイン済ならどのUserでも表示可」から「adminの条件を満たすUserのみ表示可」に変化。
```php
Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware(['auth', 'verified'])->name('dashboard')->can('admin');
```

### ヒント1
マルチ認証は使わない。認可機能を知らない人は認証機能だけでなんとかしようとしてUserとは別にAdminモデルを使った別の認証を作って無駄に難しいLaravelの使い方をしてしまう。認可機能を使えば簡単なことを難しくしてはいけない。

Laravelには「機能としては存在するけど実際に使うことはほとんどない機能」も多い。マルチ認証もその一つ。

## 投稿の削除機能
「管理者だけ編集や削除が可能」って仕様にすれば簡単だけど今回は投稿ごとにパスワードを入力しているので「正しいパスワードを入力すれば誰でも編集・削除が可能。ただし管理者だけはパスワードなしで可能」という仕様にする。

最初に考えるのは「機能をどこに配置するか」。ある程度できてから新しい機能を追加する時は使いやすさを考えて配置していく。

- 編集や削除は頻繁に使う機能ではないので目立たせない。
- breezeにはdropdown Bladeコンポーネントがあるのでこれを使ってメニューを作る。

bladeだけで仮のデザインを作ってから各機能を作っていく。

一番最初の掲示板を作り始める段階ではどんな画面にするかはある程度頭の中にあるのでルーティングやPostモデルから作る。開発が進んで動いてる画面ができた段階になったらデザインが先なこともある。

「削除のルート」はまだないので仮。  
`resources/views/home/post-menu.blade.php`
```php
<x-dropdown align="left" width="w-20">
    <x-slot name="trigger">
        <button class="inline-flex items-center px-3 py-0 border border-transparent text-sm leading-4 font-medium rounded-md text-gray-500 bg-white hover:text-gray-700 focus:outline-none transition ease-in-out duration-150">
            <div class="m-0">
                <svg class="fill-current h-4 w-4" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20">
                    <path fill-rule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd" />
                </svg>
            </div>
        </button>
    </x-slot>

    <x-slot name="content">
        <x-dropdown-link :href="route('post.edit', $post)">
            編集
        </x-dropdown-link>

        <x-dropdown-link :href="route('post.edit', $post)">
            削除...
        </x-dropdown-link>
    </x-slot>
</x-dropdown>
```

postsではtitleの所にメニューを配置。  
`resources/views/home/posts.blade.php`
```php
                <div class="flex space-x-3">
                    <h2 class="font-bold">
                        {{ $post->title ?? 'NO TITLE' }}
                    </h2>

                    <div class="text-gray-400 font-medium">
                        #{{ $post->id }}
                    </div>

                    <div>
                        @include('home.post-menu')
                    </div>
                </div>
```

この段階でもブラウザで操作はできる。

「削除の確認画面」から作っていく。Laravelのリソースコントローラーは一通りの機能が揃っているけど削除前の確認はない。JavaScriptが使えるならモーダルで確認画面を出すけど今回は使わないので確認用のルートを作る。

まずはコントローラーから。PostControllerにメソッドを増やしがちだけどこれはしないほうがいい。「Laravelのリソースコントローラーが持つメソッドはindex()からdestroy()まで」が当然の前提。余計なメソッドを増やすと後から見た時に分かりにくい。確認画面のためだけのコントローラーを作っていい。
```shell
php artisan make:controller PostDeleteConfirmController -i

   INFO  Controller [app/Http/Controllers/PostDeleteConfirmController.php] created successfully.
```

どのPostかの情報が必要なので引数にPost
```php
class PostDeleteConfirmController extends Controller
{
    /**
     * Handle the incoming request.
     *
     * @param  Request  $request
     * @param  Post  $post
     * @return \Illuminate\Http\Response
     */
    public function __invoke(Request $request, Post $post)
    {
        return view('home.post-delete-confirm')->with(compact('post'));
    }
}
```

`/routes/web.php`
```php
Route::get('post/{post}/delete', PostDeleteConfirmController::class)->name('post.delete');
```

`resources/views/home/post-delete-confirm.blade.php`
```php
<x-app-layout>
    <div class="py-12">
        <div class="max-w-3xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 text-gray-900">
                    <h2 class="text-lg font-bold">本当に削除しますか？コメントも削除されます。</h2>
                    <form method="post" action="{{ route('post.destroy', $post) }}" class="p-6">
                        @csrf
                        @method('delete')

                        <h2 class="text-lg font-medium text-gray-900">
                            {{ $post->title ?? 'NO TITLE' }}
                        </h2>

                        <p class="mt-1 text-sm text-gray-600">
                            {!! nl2br(e($post->content)) !!}
                        </p>

                        <div class="mt-6">
                            <x-input-label for="password" value="削除するには投稿時のパスワードを入力してください"/>
                            @can('admin')
                                <div class="text-red-600">管理者はパスワード不要</div>
                            @endcan

                            <x-text-input
                                id="password"
                                name="password"
                                type="password"
                                class="mt-1 block w-full"
                                placeholder="Password"
                            />

                            <x-input-error :messages="$errors->get('password')" class="mt-2"/>
                        </div>

                        <div class="mt-6 flex justify-end">
                            <a href="{{ route('home') }}" class="inline-flex items-center px-4 py-2 bg-white border border-gray-300 rounded-md font-semibold text-xs text-gray-700 uppercase tracking-widest shadow-sm hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 disabled:opacity-25 transition ease-in-out duration-150">キャンセル</a>

                            <x-danger-button class="ml-3">
                                削除
                            </x-danger-button>
                        </div>
                    </form>

                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```

仮だったメニューを確認へのルートに。  
`post-menu.blade.php`
```php
        <x-dropdown-link :href="route('post.delete', $post)">
            削除...
        </x-dropdown-link>
```

ここまでで確認画面の表示はできる。  
次は実際に削除する部分。PostControllerのdestroy()
```php
    public function destroy(Post $post)
    {
        //
    }
```
Postだけ指定されてるけどパスワードのバリデーションが必要なのでフォームリクエストを作る。

```shell
php artisan make:request DestroyPostRequest
```

`app/Http/Requests/DestroyPostRequest.php`
```php
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
            'password' => [
                function ($attribute, $value, $fail) {
                    if (! $this->user()?->can('admin') && ! Hash::check($value, $this->post->password)) {
                        $fail('パスワードが違います。');
                    }
                },
            ],
        ];
    }
```

管理者でないかつパスワードの確認に失敗したらバリデーション失敗扱い。後でテストでしっかり確認。

DestroyPostRequestで確認してるのでPostControllerのdestroy()はそのまま削除。
```php
    /**
     * Remove the specified resource from storage.
     *
     * @param  DestroyPostRequest  $request
     * @param  Post  $post
     * @return Response
     */
    public function destroy(DestroyPostRequest $request, Post $post)
    {
        $post->delete();

        return to_route('home');
    }
```

ブラウザで「管理者ならパスワードなしでも削除できる」、「ログアウト状態なら正しいパスワードを入力しないと削除できない」ことを確認。

こんなのいちいち手動で確認したくないのでテストを書く。

`tests/Feature/PostTest.php`
```php
    public function test_delete_confirm()
    {
        $post = Post::factory()->create();

        $response = $this->get(route('post.delete', $post));

        $response->assertSuccessful();
    }

    public function test_admin_can_delete_post_without_password()
    {
        $admin = User::factory()->create(['id' => 1]);
        $post = Post::factory()->create();

        $response = $this->actingAs($admin)->delete(route('post.destroy', $post), [
            'password' => null,
        ]);

        $response->assertRedirect();

        $this->assertDatabaseCount('posts', 0)
             ->assertDatabaseMissing('posts', [
                 'id' => $post->id,
             ]);
    }

    public function test_guest_can_delete_post_with_password()
    {
        $post = Post::factory()->create();

        $response = $this->delete(route('post.destroy', $post), [
            'password' => 'password',
        ]);

        $response->assertRedirect();

        $this->assertDatabaseCount('posts', 0)
             ->assertDatabaseMissing('posts', [
                 'id' => $post->id,
             ]);
    }

    public function test_guest_cannot_delete_post_without_password()
    {
        $post = Post::factory()->create();

        $response = $this->delete(route('post.destroy', $post), [
            'password' => null,
        ]);

        $response->assertRedirect()
                 ->assertInvalid(['password']);

        $this->assertDatabaseCount('posts', 1)
             ->assertDatabaseHas('posts', [
                 'id' => $post->id,
             ]);
    }
```

編集やコメントの削除・編集機能は同じ説明になるのでもう省略。

リソースコントローラーでの基本的なCRUDは毎回同じ作業で退屈。Laravelでも便利にする機能はない。