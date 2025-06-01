# Difficulty Using Laravel and Livewire Pagination Together in an Octane Environment

---

## Versions
- Laravel 9.x
- Livewire 2.x
- Octane 1.x ([Vapor+Octane](https://docs.vapor.build/1.0/projects/environments.html#octane))
- PHP 8.1

## WithPagination
Livewire's pagination doesn't seem to do anything unusual when used normally, but looking closely, `initializeWithPagination()` changes it to Livewire's unique behavior.

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

`Paginator::currentPageResolver` and `Paginator::defaultView` modify static properties here.

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

The fact that they are static properties is what causes problems with Octane.

1. Livewire's pagination is displayed first. At this time, the static properties are overwritten. With Octane, they remain overwritten for the next request.
2. Even if Laravel's normal pagination is displayed, it will be displayed based on the overwritten static properties, resulting in an error. Without Octane, it's reset every time, so there's no impact.

## Trial and Error

### If the view is different
You can specify it at display time, but specifying it everywhere is cumbersome.
```php
{{ $posts->links('pagination::tailwind') }}
```

It seems unlikely that static properties can be overwritten again when displaying Laravel pagination in `AppServiceProvider::boot()`.
```php
Paginator::defaultView('pagination::tailwind');
Paginator::defaultSimpleView('pagination::simple-tailwind');
```

Specify the view when displaying with `links()`.
Since `defaultView()` is the default, if specified, the default will not be used. (The default at this point is the view file overwritten by Livewire).

### Resetting the Resolver
It seems possible to reset with `PaginationState::resolveUsing()`.

https://github.com/laravel/framework/blob/9.x/src/Illuminate/Pagination/PaginationState.php

It's used internally in Laravel here:
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

I thought I could just write it in `AppServiceProvider` similarly, but it didn't solve the problem.
Like the view, reset it in the controller every time it's displayed.

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

Aside from the hassle, Laravel's pagination was fixed, but now Livewire's pagination started throwing errors.

## Reset with Octane's Listener (Best Solution)
Setting a Listener in `config/octane.php` to initialize with each request seems to be the current solution.

https://github.com/laravel/octane/blob/1.x/config/octane.php

```php
        RequestReceived::class => [
            ...Octane::prepareApplicationForNextOperation(),
            ...Octane::prepareApplicationForNextRequest(),
            //
        ],
```

Following the flow, various initialization processes are performed when the `RequestReceived` event occurs.

`prepareApplicationForNextOperation()` includes `PrepareLivewireForNextOperation`.

https://github.com/laravel/octane/blob/1.x/src/Concerns/ProvidesDefaultConfigurationOptions.php

`PrepareLivewireForNextOperation` calls Livewire's `flushState()`.

https://github.com/laravel/octane/blob/1.x/src/Listeners/PrepareLivewireForNextOperation.php

Since `flushState()` doesn't handle pagination, you need to do it yourself.

https://github.com/livewire/livewire/blob/e9f178bc4f1e671e562f9d2251aa07702b2c2260/src/LivewireManager.php#L460

First, create a Listener.

```
sail art make:listener FlushPagination
```
or
```
php artisan make:listener FlushPagination
```

FlushPagination.php is as follows:

```php
<?php

namespace App\Listeners;

use Illuminate\Pagination\PaginationState;
use Illuminate\Pagination\Paginator;

class FlushPagination
{
    /**
     * Create the event listener.
     *
     * @return void
     */
    public function __construct()
    {
        //
    }

    /**
     * Handle the event.
     *
     * @param  object  $event
     * @return void
     */
    public function handle($event)
    {
        Paginator::useTailwind();
        PaginationState::resolveUsing($event->sandbox);
    }
}
```

Add to `config/octane.php`.

```php
        RequestReceived::class => [
            ...Octane::prepareApplicationForNextOperation(),
            ...Octane::prepareApplicationForNextRequest(),
            //
            \App\Listeners\FlushPagination::class,
        ],
```

`useTailwind()` is the same as below. If not using Tailwind, change as you like.

```php
Paginator::defaultView('pagination::tailwind');
Paginator::defaultSimpleView('pagination::simple-tailwind');
```

`$event->sandbox` and `app()` should be the same.

Now, specifying the view every time is unnecessary.
```php
{{ $posts->links() }}
```

`PaginationState::resolveUsing()` in ServiceProviders or controllers is also unnecessary.

No errors occur when navigating between pages using Laravel's pagination and Livewire's pagination.

There are no problems with normal use. It's impossible to confirm if other problems will arise without using it for a while.

## With Octane v1.2.10
https://github.com/laravel/octane/releases/tag/v1.2.10
`PaginationState::resolveUsing($event->sandbox);` has also been enabled on the Octane side, so it's no longer needed in `FlushPagination`.
```php
    public function handle($event)
    {
        Paginator::useTailwind();
    }
}
```
