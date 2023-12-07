Livewireの普通の使い方とVoltの比較
----

## バージョン
- Laravel 10.x
- Livewire 3.2.6 https://github.com/livewire/livewire
- Volt 1.6.0 https://github.com/livewire/volt

最近でも機能が追加されてるので2023年12月時点の情報。

## ドキュメント
https://livewire.laravel.com/docs/volt

## 3つのスタイル
一つのLaravelプロジェクト内で3つのスタイルを同時に使える。簡単なページはVolt functional、複雑なページは普通などで使い分けしてもいい。最初はVolt functionalで作ったけど複雑になってきたので普通の使い方に作り変えるなんてこともよく発生する。

### Livewire スタンダード
PHPファイルとBladeファイルが分かれた普通の使い方。
実装例：JetstreamのLivewireスタック。

Voltの前に普通の使い方の理解は必須。
この記事は普通の使い方でのこの機能はVoltでどう書くのかって情報のまとめ。

### Volt functional
BladeファイルだけでLivewireを使う新しい方法。Vue.js風の表現ならシングルファイルLivewireコンポーネント。
実装例：Breezeのlivewire-functionalスタック。 https://github.com/laravel/breeze/tree/1.x/stubs/livewire-functional/resources/views/livewire

```php
<?php
 
use function Livewire\Volt\{state};
 
state(['count' => 0]);
 
$increment = fn () => $this->count++;
 
?>
 
<div>
    <h1>{{ $count }}</h1>
    <button wire:click="increment">+</button>
</div>
```

Laravel Folioと似てるし一緒に発表されたけどFolioとは関係ない。

### Volt Class-based
Voltを普通の使い方に近い無名classを使うスタイルにしたもの。
実装例：Breezeのlivewireスタック。 https://github.com/laravel/breeze/tree/1.x/stubs/livewire/resources/views/livewire

```php
<?php
 
use Livewire\Volt\Component;
 
new class extends Component {
    public $count = 0;
 
    public function increment()
    {
        $this->count++;
    }
} ?>
 
<div>
    <h1>{{ $count }}</h1>
    <button wire:click="increment">+</button>
</div>
```
複雑化しただけでこれの追加は余計だった気がする。

## コンポーネント作成コマンド

### Livewire スタンダード
```
php artisan make:livewire counter
```

### Volt functional
```
php artisan make:volt counter
```
ファイル一つなので2回目からはコマンドを使うより既存のコンポーネントを複製して書き換えが現実的な使い方。

### Volt Class-based
functionalと同じ。

## ファイルの場所

### Livewire スタンダード
- `app/Livewire/`にPHPファイル
- `resources/views/livewire`にBladeファイル

### Volt functional
`resources/views/livewire`にBladeファイルのみ。

`resources/views/pages`に置いたBladeファイルも使える。Folio用に作ったページだけど機能を追加したくなってVoltに変更、みたいな使い方を想定しているのだろう。FolioのファイルベースルーティングとVoltを同時に使えるかは試してないので不明。

### Volt Class-based
functionalと同じ。

## プロパティとmount

### Livewire スタンダード
```php
<?php
 
namespace App\Livewire;

use App\Models\Post;
use Livewire\Component;
 
class PostIndex extends Component
{
    public $posts;
    
    public function mount()
    {
        $this->posts = Post::all(); 
    }
}
```

### Volt functional
```php
<?php
use App\Models\Post;
use function Livewire\Volt\state;
use function Livewire\Volt\mount;

state('posts');

mount(function () {
    $this->posts = Post::all(); 
});
?>
<div></div>
```

### Volt Class-based
```php
<?php

use App\Models\Post;
use Livewire\Volt\Component;
 
new class extends Component {
    public $posts;
 
    public function mount()
    {
        $this->posts = Post::all(); 
    }
}
?>
<div></div>
```

## アクション

### Livewire スタンダード
```php
namespace App\Livewire;
 
use Livewire\Component;
use App\Models\Post;
 
class CreatePost extends Component
{
    public $title = '';
 
    public $content = '';
 
    public function save()
    {
        Post::create([
            'title' => $this->title,
            'content' => $this->content,
        ]);
 
        return redirect()->to('/posts');
    }
}
```

```php
<form wire:submit="save"> 
    <input type="text" wire:model="title">
 
    <textarea wire:model="content"></textarea>
 
    <button type="submit">Save</button>
</form>
```

### Volt functional
```php
<?php

use App\Models\Post;
use function Livewire\Volt\state;
 
state(['title', 'content']);
 
$save = function () {
    Post::create([
        'title' => $this->title,
        'content' => $this->content,
    ]);
 
    return redirect()->to('/posts');
};
?>

<div>
    <form wire:submit="save"> 
        <input type="text" wire:model="title">
 
        <textarea wire:model="content"></textarea>
 
        <button type="submit">Save</button>
    </form>
</div>
```

### Volt Class-based
スタンダードとほぼ同じなので以降は省略。

## バリデーション
### Livewire スタンダード
`Validate`アトリビュートで指定。

```php
use Livewire\Attributes\Validate;
use Livewire\Component;
use App\Models\Post;
 
class CreatePost extends Component
{
    #[Validate('required|min:3')] 
    public $title = '';
 
    #[Validate('required|min:3')] 
    public $content = '';
}
```

### Volt functional
`rules()`で指定。

```php
<?php
 
use function Livewire\Volt\{rules};
 
rules(['name' => 'required|min:6', 'email' => 'required|email']);
 
$submit = function () {
    $this->validate();
 
    // ...
};
 
?>
 
<form wire:submit.prevent="submit">
    //
</form>
```

## Computed プロパティ
### Livewire スタンダード
```php
use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\User;
 
class ShowUser extends Component
{ 
    #[Computed]
    public function count()
    {
        return User::count();
    }
```

### Volt functional
```php
<?php
 
use App\Models\User;
use function Livewire\Volt\{computed};
 
$count = computed(function () {
    return User::count();
});
 
?>
 
<div>
    {{ $this->count }}
</div>
```

## ページネーション
### Livewire スタンダード
`WithPagination`の追加を忘れやすい。

```php
<?php
 
namespace App\Livewire;
 
use Livewire\WithPagination;
use Livewire\Component;
use App\Models\Post;
 
class ShowPosts extends Component
{
    use WithPagination;
 
    public function render()
    {
        return view('show-posts', [
            'posts' => Post::paginate(10),
        ]);
    }
}
```

### Volt functional
```php
<?php
 
use function Livewire\Volt\{with, usesPagination};
 
usesPagination();
 
with(fn () => ['posts' => Post::paginate(10)]);
 
?>
 
<div>
    @foreach ($posts as $post)
        //
    @endforeach
 
    {{ $posts->links() }}
</div>
```

## ファイルアップロード
### Livewire スタンダード
```php
<?php
 
namespace App\Livewire;
 
use Livewire\Component;
use Livewire\WithFileUploads;
use Livewire\Attributes\Validate;
 
class UploadPhoto extends Component
{
    use WithFileUploads;
 
    #[Validate('image|max:1024')]
    public $photo;
 
    public function save()
    {
        $this->photo->store('photos');
    }
}
```

### Volt functional
```php
use function Livewire\Volt\{state, usesFileUploads};
 
usesFileUploads();
 
state(['photo']);
 
$save = function () {
    $this->validate([
        'photo' => 'image|max:1024',
    ]);
 
    $this->photo->store('photos');
};
```

## Full-page component レイアウト指定

### Livewire スタンダード
render()かclassにLayoutアトリビュートで指定。レイアウトは基本的に固定なのでアトリビュートでの指定でいいだろう。

```php
<?php
 
namespace App\Livewire;
 
use Livewire\Attributes\Layout;
use Livewire\Component;
 
class CreatePost extends Component
{
    // ...
 
    #[Layout('layouts.app')] 
    public function render()
    {
        return view('livewire.create-post');
    }
}
```

```php
<?php
 
namespace App\Livewire;
 
use Livewire\Attributes\Layout;
use Livewire\Component;
 
#[Layout('layouts.app')] 
class CreatePost extends Component
{
    // ...
}
```

### Volt functional
layout()で指定。

```php
use function Livewire\Volt\{layout, state};
 
state('users');
 
layout('components.layouts.admin');
```

## Full-page component title指定
titleだけ特別な扱い。

先にレイアウトファイルに`$title`を用意。
```php
<title>{{ $title ?? config('app.name') }}</title>
```

### Livewire スタンダード
```php
<?php
 
namespace App\Livewire;
 
use Livewire\Attributes\Layout;
use Livewire\Component;
 
class Post extends Component
{
    public $post;
 
    public function render()
    {
        return view('livewire.post-show')
            ->title($this->post->title); 
    }
}
```
ちなみにLivewire3のドキュメントからは消えてるけどtitle以外のデータをレイアウトに渡す`layoutData()`もある。
```php
    public function render()
    {
        return view('livewire.post-show')
            ->layoutData(['foo' => 'bar']); 
    }
```

`Title`アトリビュートで指定する方法もあるけど固定のtitleでしか使えないので使うことはほとんどない。

```php
use Livewire\Attributes\Title;
use Livewire\Component;
 
class CreatePost extends Component
{
    // ...
 
    #[Title('Create Post')] 
    public function render()
    {
        return view('livewire.create-post');
    }
}
```

### Volt functional

```php
use function Livewire\Volt\{state, title};
 
state('post');
 
title(fn() => $this->post->title);
```

固定のtitleなら`titie('Post');`も可能だけどこれもほとんど使わないだろう。

### Volt Class-based
ここだけ特殊で`rendering()`で指定。
```php
<?php
 
use Illuminate\View\View;
use Livewire\Volt\Component;
 
new class extends Component {
    public function rendering(View $view): void
    {
        $view->title('Create Post');
 
        // ...
    }
 
    // ...
```

## Full-page component ルーティング
### Livewire スタンダード
PHPファイルを指定。
```php
use App\Livewire\CreatePost;
 
Route::get('/posts/create', CreatePost::class)->name('post.create');
```

### Volt functional
Bladeファイルを指定。`Volt::route()`の戻り値はRouteなので通常通りnameの指定なども可能。
```php
use Livewire\Volt\Volt;
 
Volt::route('/users', 'user-index')->name('user.index');
```

## URLクエリパラメータ

### Livewire スタンダード
```php
<?php
 
namespace App\Livewire;
 
use Livewire\Attributes\Url;
use Livewire\Component;
 
class ShowUsers extends Component
{
    #[Url] 
    public $search = '';
```

### Volt functional
```php
<?php
 
use function Livewire\Volt\{state};
 
state(['search'])->url();
```

## イベントリスナー
### Livewire スタンダード
```php
use Livewire\Component;
use Livewire\Attributes\On; 
 
class Dashboard extends Component
{
    #[On('post-created')] 
    public function updatePostList($title)
    {
        // ...
    }
}
```
### Volt functional
```php
use function Livewire\Volt\{on};
 
on(['post-created' => function () {
    //
}]);
```

## 以上
よく使うだろう機能はまとめたのでこれ以上はドキュメントを参照。

アドベントカレンダー用の記事。
https://qiita.com/advent-calendar/2023/laravel

ここは長文を想定してないけど最近は外部に書いてないのでここに書く。
ドメインを破棄した時用にGitHubにも残している。
https://github.com/pop-culture-studio/articles/blob/main/livewire-volt.md
