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
