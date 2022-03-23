Octane環境下ではLaravelとLivewireのページネーションを併用しにくい
----

## バージョン
- Laravel 9.x
- Livewire 2.x
- Octane 1.x [(Vapor+Octane)](https://docs.vapor.build/1.0/projects/environments.html#octane)
- PHP 8.1

## WithPagination
Livewireのページネーションは普通に使うだけなら変わったことしてないように見えるけど詳しく見ると`initializeWithPagination()`でLivewire独自の動作に変更している。  

https://github.com/livewire/livewire/blob/master/src/WithPagination.php

```php
        if (class_exists(CursorPaginator::class)) {
            CursorPaginator::currentCursorResolver(function ($pageName){
                if (! isset($this->paginators[$pageName])) {
                    $this->paginators[$pageName] = request()->query($pageName, '');
                }
                return Cursor::fromEncoded($this->paginators[$pageName]);
            });
        }

        Paginator::currentPageResolver(function ($pageName) {
            if (! isset($this->paginators[$pageName])) {
                $this->paginators[$pageName] = request()->query($pageName, 1);
            }

            return (int) $this->paginators[$pageName];
        });

        Paginator::defaultView($this->paginationView());
        Paginator::defaultSimpleView($this->paginationSimpleView());
```

`Paginator::currentPageResolver`や`Paginator::defaultView`はここでstaticプロパティを変更している。

https://github.com/laravel/framework/blob/9.x/src/Illuminate/Pagination/AbstractPaginator.php

```php
    /**
     * The current page resolver callback.
     *
     * @var \Closure
     */
    protected static $currentPageResolver;
    
    /**
     * Set the current page resolver callback.
     *
     * @param  \Closure  $resolver
     * @return void
     */
    public static function currentPageResolver(Closure $resolver)
    {
        static::$currentPageResolver = $resolver;
    }
```

Octaneだとstaticプロパティなことが困る原因。

1. Livewireのページネーションを先に表示。この時にstaticプロパティが上書きされる。Octaneでは次のリクエストでも上書きされたまま。
2. Laravelの通常のページネーションを表示しても上書きされたstaticプロパティを元に表示されるのでエラーになる。Octaneでなければ毎回リセットされるので影響はない。

## 完璧な解決策はなさそう

### viewが違うのは
表示時に指定すればいいけど全部で指定は面倒。
```php
{{ $posts->links('pagination::tailwind') }}
```

AppServiceProvider::boot()ではLaravelページネーションを表示時にstaticプロパティを再度上書きはできそうにない。
```php
Paginator::defaultView('pagination::tailwind');
Paginator::defaultSimpleView('pagination::simple-tailwind');
```

links()で表示するタイミングでviewを指定する。  
defaultView()はデフォルトなので指定すればデフォルトは使われない。（この時点でのデフォルトはLivewireが上書きしたviewファイル）

### Resolverのリセット
`PaginationState::resolveUsing()`でリセットはできそう。

https://github.com/laravel/framework/blob/9.x/src/Illuminate/Pagination/PaginationState.php

Laravel内部ではここで使われている。
```php
    /**
     * Register the service provider.
     *
     * @return void
     */
    public function register()
    {
        PaginationState::resolveUsing($this->app);
    }
```

https://github.com/laravel/framework/blob/9.x/src/Illuminate/Pagination/PaginationServiceProvider.php

同じようにAppServiceProviderに書けばいいかと思ったけど解決しなかった。  
viewと同じく表示時にコントローラーで毎回リセット。

```php
use Illuminate\Pagination\PaginationState;

//...

    public function index(Request $request)
    {
        PaginationState::resolveUsing(app());

        $posts = Post::latest()->paginate();

        return view('post.index')->with(compact('posts'));
    }
```

面倒なことを除けばLaravel側のページネーションは直ったけど今度はLivewire側のページネーションでエラーが出るようになった。

後は`config/octane.php`のlistenersでリセットする方法がありそうだけど今はここまで。

https://github.com/laravel/octane/blob/1.x/config/octane.php

## 現状の解決策
「Livewireのページネーションを使うプロジェクトではページネーションを使うすべてのページでLivewireを使う」  
ページネーションを使わないページでは関係ない。Laravelの通常のページネーションを使わないようにする。
