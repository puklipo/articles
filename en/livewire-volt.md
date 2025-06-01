# Comparison of Standard Livewire Usage and Volt

---

## Versions
- Laravel 10.x
- Livewire 3.2.6 https://github.com/livewire/livewire
- Volt 1.6.0 https://github.com/livewire/volt

Information as of December 2023, as features are still being added recently.

## Documentation
https://livewire.laravel.com/docs/volt

## Three Styles
You can use all three styles simultaneously within a single Laravel project. You can use Volt functional for simple pages and the standard way for complex pages. It's also common to initially create something with Volt functional and then refactor it to the standard way as it becomes more complex.

### Livewire Standard
The usual way of using Livewire with separate PHP and Blade files.
Implementation example: Jetstream's Livewire stack.

Understanding the standard way is essential before using Volt.
This article summarizes how to write features from the standard way using Volt.

### Volt functional
A new way to use Livewire with only Blade files. In Vue.js terms, a single-file Livewire component.
Implementation example: Breeze's livewire-functional stack. https://github.com/laravel/breeze/tree/1.x/stubs/livewire-functional/resources/views/livewire

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

It's similar to Laravel Folio and was announced together, but it's unrelated to Folio.

### Volt Class-based
Volt adapted to a style using anonymous classes, similar to the standard way.
Implementation example: Breeze's livewire stack. https://github.com/laravel/breeze/tree/1.x/stubs/livewire/resources/views/livewire

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
It feels like adding this just made things more complicated. Laravel allowing multiple ways of doing things is already a source of confusion, and adding more options is clearly bad.

## Component Creation Command

### Livewire Standard
```
php artisan make:livewire counter
```

### Volt functional
```
php artisan make:volt counter
```
Since it's a single file, for subsequent components, it's more practical to duplicate an existing component and modify it rather than using the command.

### Volt Class-based
Same as functional.

## File Locations

### Livewire Standard
- PHP files in `app/Livewire/`
- Blade files in `resources/views/livewire`

### Volt functional
Only Blade files in `resources/views/livewire`.

Blade files placed in `resources/views/pages` can also be used. This likely anticipates use cases like creating a page for Folio and then wanting to add functionality, converting it to Volt. It's unclear if Folio's file-based routing and Volt can be used simultaneously as I haven't tested it.

### Volt Class-based
Same as functional.

## Properties and mount

### Livewire Standard
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

## Actions

### Livewire Standard
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
Almost the same as standard, so omitted hereafter.

## Validation
### Livewire Standard
Specify with the `Validate` attribute.

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
Specify with `rules()`.

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

## Computed Properties
### Livewire Standard
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

## Pagination
### Livewire Standard
Easy to forget to add `WithPagination`.

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

## File Uploads
### Livewire Standard
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

## Full-page component Layout Specification

### Livewire Standard
Specify in `render()` or with the `Layout` attribute on the class. Layouts are generally fixed, so specifying with an attribute should be fine.

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
Specify with `layout()`.

```php
use function Livewire\Volt\{layout, state};

state('users');

layout('components.layouts.admin');
```

## Full-page component title Specification
Only `title` has special treatment.

First, prepare `$title` in the layout file.
```php
<title>{{ $title ?? config('app.name') }}</title>
```

### Livewire Standard
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
Incidentally, although removed from the Livewire 3 documentation, there is also `layoutData()` to pass data other than title to the layout.
```php
    public function render()
    {
        return view('livewire.post-show')
            ->layoutData(['foo' => 'bar']);
    }
```

There's also a method to specify with the `Title` attribute, but it can only be used for fixed titles, so it's rarely used.

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

A fixed title like `title('Post');` is also possible, but this will also rarely be used.

### Volt Class-based
This one is special and specified with `rendering()`.
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

## Full-page component Routing
### Livewire Standard
Specify the PHP file.
```php
use App\Livewire\CreatePost;

Route::get('/posts/create', CreatePost::class)->name('post.create');
```

### Volt functional
Specify the Blade file. The return value of `Volt::route()` is a Route, so specifying `name` etc. is possible as usual.
```php
use Livewire\Volt\Volt;

Volt::route('/users', 'user-index')->name('user.index');
```

## URL Query Parameters

### Livewire Standard
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

## Event Listeners
### Livewire Standard
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

## That's all
I've summarized the features you'll likely use often. For anything more, please refer to the documentation.
