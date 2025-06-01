# Auto-scroll After Page Change in Laravel/Livewire Pagination

---

## Versions
- Laravel 9.x (+Jetstream 2.x)
- Livewire 2.x
- Alpine.js 3.x
- PHP 8.1

## Livewire Pagination
https://laravel-livewire.com/docs/2.x/pagination

```php
use Livewire\WithPagination;

class ShowPosts extends Component
{
    use WithPagination;

    public function render()
    {
        return view('livewire.show-posts', [
            'posts' => Post::paginate(10),
        ]);
    }
}
```
```php
<div>
    @foreach ($posts as $post)
        ...
    @endforeach

    {{ $posts->links() }}
</div>
```

It can be used almost the same way as Laravel's pagination, and the page display switches without a full reload.

This alone is simple, but when actually using it, "if you change pages at the bottom of a long page, the display just changes as is, which is slightly inconvenient." You'll want to scroll to the top.

## Scrolling with dispatchBrowserEvent() and Alpine.js
First, the PHP side.

You can dispatch an event from the PHP side to JS using `dispatchBrowserEvent()`.
https://laravel-livewire.com/docs/2.x/events#browser

```php
use Livewire\WithPagination;

class ShowPosts extends Component
{
    use WithPagination;

    public function updatedPage($page)
    {
        $this->dispatchBrowserEvent('page-updated');
    }

    public function render()
    {
        return view('livewire.show-posts', [
            'posts' => Post::paginate(10),
        ]);
    }
}
```

When the page changes in `updatedPage()`, an event is dispatched. (Since `updatedPage()` is used when `page` is used like `?page=2`, adjustments are needed if using multiple paginations.)

Next, the view side.

Events from `dispatchBrowserEvent()` can be received in the format `@page-updated.window`. This is an Alpine.js feature.
https://alpinejs.dev/essentials/events#listening-for-events-on-window

```php
<div x-data @page-updated.window="">
    @foreach ($posts as $post)
        ...
    @endforeach

    {{ $posts->links() }}
</div>
```

`x-data` is also required (easy to forget).

Finally, scroll when the event is received.

```php
<div x-data @page-updated.window="$el.scrollIntoView({behavior: 'smooth'})">
    @foreach ($posts as $post)
        ...
    @endforeach

    {{ $posts->links() }}
</div>
```

`$el.scrollIntoView({behavior: 'smooth'})`

`$el` refers to the element itself, in this case, the div. This is an Alpine.js feature.
It should be the same with something like `<div x-data @page-updated.window="document.querySelector('#test').scrollIntoView({behavior: 'smooth'})" id="test">`.

`scrollIntoView` scrolls to the position of this element. This is standard JavaScript.
However, `{behavior: 'smooth'}` is not supported in Safari. This means it's also not supported in Chrome on iOS. If you want to support it, use a polyfill.
https://github.com/iamdustan/smoothscroll

In `resources/js/app.js` or similar:
```js
require('./bootstrap');

import Alpine from 'alpinejs';

window.Alpine = Alpine;

Alpine.start();

import smoothscroll from 'smoothscroll-polyfill';

smoothscroll.polyfill();
```
**(Update: Safari 15.4 now supports this, so the polyfill is no longer needed.)**

You can scroll to any position by changing where you write `@page-updated.window`.

This allows you to achieve "scroll to top when changing pages in Livewire pagination."

## Back to top
Personally, I don't use it much, but a "Back to top" button can also be implemented with the content of this article.

## Addendum
Livewire 3.0.6 and later automatically scroll, so you can ignore this entire article.
