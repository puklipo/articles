Laravel/Livewireページネーションでページ移動後に自動スクロールさせる
----

## バージョン
- Laravel 9.x (+Jetstream 2.x)
- Livewire 2.x
- Apline.js 3.x
- PHP 8.1

## Livewireのページネーション
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

Laravelのページネーションとほとんど同じ感覚で使えて再読み込みなしでページの表示が切り替わる。

これだけなら簡単だけど実際に使っていると「長いページの下方でページ移動するとそのまま表示が変わるだけなので若干不便」。上の方にスクロールさせたくなる。

## dispatchBrowserEvent()とalpine.jsでスクロール
まずPHP側。

`dispatchBrowserEvent()`でPHP側からJSにイベントを発行できる。  
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

`updatedPage()`でページが切り替わったらイベントを発行。（`?page=2`のように`page`を使ってる場合に`updatedPage()`なので複数のページネーションを使ってるなら調整が必要。）


次にview側。

dispatchBrowserEvent()からのイベントを`@page-updated.window`の形で受信できる。これはalpine.jsの機能。  
https://alpinejs.dev/essentials/events#listening-for-events-on-window

```php
<div x-data @page-updated.window="">
    @foreach ($posts as $post)
        ...
    @endforeach
 
    {{ $posts->links() }}
</div>
```

`x-data`も必須(忘れやすい)

最後にイベント受信時にスクロール。

```php
<div x-data @page-updated.window="$el.scrollIntoView({behavior: 'smooth'})">
    @foreach ($posts as $post)
        ...
    @endforeach
 
    {{ $posts->links() }}
</div>
```

`$el.scrollIntoView({behavior: 'smooth'})`

`$el`このelement自身。この場合はdiv。alpine.jsの機能。  
`<div x-data @page-updated.window="document.querySelector('#test').scrollIntoView({behavior: 'smooth'})" id="test">`などでも同じはず。

`scrollIntoView`このelementの位置までスクロールする。普通のJavaScript。  
ただし`{behavior: 'smooth'}`はSafariで非対応。つまりiOSのChromeでも非対応。対応させたい場合はpolyfillを使う。  
https://github.com/iamdustan/smoothscroll

`resources/js/app.js`などで
```js
require('./bootstrap');

import Alpine from 'alpinejs';

window.Alpine = Alpine;

Alpine.start();

import smoothscroll from 'smoothscroll-polyfill';

smoothscroll.polyfill();
```
**（追記：Safari15.4で対応されたのでpolyfillは不要になった）**


`@page-updated.window`を書く場所を変えれば好きな位置にスクロールできる。

これで「Livewireのページネーションでページ移動した時に上にスクロール」が実現できる。

## Back to top
個人的にはあまり使わないけど「トップに戻る」ボタンなんかもこの記事の内容で実現できる。

## 追記
Livewire 3.0.6以降は自動でスクロールするのでこの記事はすべて無視していい。
