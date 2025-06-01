# Laravel 11: Undocumented Ways to Use bootstrap/app.php

---

This is a summary for a frequently asked question on overseas Q&A sites.

It's rarely seen in Japan. The Laravel community in Japan is polarized between **beginners who ask questions about what's already in the documentation** and **veterans who don't need to ask questions**. There's a lack of people in the intermediate stage who are trying to do slightly more complex things not covered in the documentation.

## Version
- Laravel 11
- PHP 8.3

Always check the version when discussing Laravel. Features might not exist in older versions, or their usage might change in future versions.

## Basic Flow Before bootstrap/app.php
The entry point for Laravel is:
For HTTP: `public/index.php`
```php
// Bootstrap Laravel and handle the request...
(require_once __DIR__.'/../bootstrap/app.php')
    ->handleRequest(Request::capture());
```

For Console: `artisan`
```php
// Bootstrap Laravel and handle the command...
$status = (require_once __DIR__.'/bootstrap/app.php')
    ->handleCommand(new ArgvInput);
```

`bootstrap/app.php`, which is loaded by both, has become a crucial file in Laravel 11.

## Normal Usage of bootstrap/app.php
Actually, the normal usage isn't well documented. It's mostly in the Laravel 11 release notes and scattered throughout the documentation for specific features.
https://laravel.com/docs/11.x/releases

Important prerequisite: **`bootstrap/app.php` is before Laravel boots.** Features that are readily available after booting cannot be used here.
If you don't understand this, you'll try to use `config()` etc. in `bootstrap/app.php` and get errors.

(Technically, since `basePath` is set initially, `base_path()` can be used. However, it's likely unused to avoid the misconception that other helpers are also available.)

## The Code to Investigate is ApplicationBuilder
What `Application::configure()` in `bootstrap/app.php` returns is:
```php
use Illuminate\Foundation\Application;

return Application::configure(basePath: dirname(__DIR__))
```
`Illuminate\Foundation\Configuration\ApplicationBuilder`
```php
    public static function configure(?string $basePath = null)
    {
        $basePath = match (true) {
            is_string($basePath) => $basePath,
            default => static::inferBasePath(),
        };

        return (new Configuration\ApplicationBuilder(new static($basePath)))
            ->withKernels()
            ->withEvents()
            ->withCommands()
            ->withProviders();
    }
```
https://github.com/laravel/framework/blob/11.x/src/Illuminate/Foundation/Application.php

`bootstrap/app.php` is mostly `ApplicationBuilder`, so you just need to examine this.
https://github.com/laravel/framework/blob/11.x/src/Illuminate/Foundation/Configuration/ApplicationBuilder.php

This was all introductory.

## To Interject Processing After Booting, Use booted()
Although `bootstrap/app.php` is before Laravel boots, only `booted()` is after booting. Here, all Laravel features can be used.
The answer to most questions like "I want to do something special in bootstrap/app.php" is this.

```php
use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        //
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })
    ->booted(function (Application $app) {
        info($app->version());
        info(config('app.name'));
    })->create();
```

It's hard to see with a simple example, but you can do anything at this `booted` stage.
"Special things" include "ignoring all middleware set with `withMiddleware` and reconfiguring them."
I don't know why anyone would want to do that, but such questions have actually existed.

Since it's not something you'd normally do, I won't explain the details.

```php
    ->booted(function (Application $app) {
        $kernel = $app->make(Kernel::class);

        $middleware = (new Middleware)
            ->redirectGuestsTo(fn () => route('login'));

        // $middleware->...

        $kernel->setGlobalMiddleware($middleware->getGlobalMiddleware());
        $kernel->setMiddlewareGroups($middleware->getMiddlewareGroups());
        $kernel->setMiddlewareAliases($middleware->getMiddlewareAliases());

        if ($priorities = $middleware->getMiddlewarePriority()) {
            $kernel->setMiddlewarePriority($priorities);
        }

        $app->instance(Kernel::class, $kernel);
    })
```

Only those who are intimately familiar with Laravel's internals would use this, so it can't be documented.

## There are also registered() and booting()
Laravel's boot process follows this order. `bootstrap/app.php` has corresponding methods, so choose based on when you want to interject processing. There are things you can and cannot do at each timing.

- Execute `register()` of all ServiceProviders
- app's `registered()`
- app's `booting()`
- Execute `boot()` of all ServiceProviders
- app's `booted()`

The middleware example is easier to understand with `registered()`, but:
```php
    ->registered(function (Application $app) {
        $app->afterResolving(Kernel::class, function ($kernel) {
            $middleware = (new Middleware)
                ->redirectGuestsTo(fn () => route('login'));

            // $middleware->...

            $kernel->setGlobalMiddleware($middleware->getGlobalMiddleware());
            $kernel->setMiddlewareGroups($middleware->getMiddlewareGroups());
            $kernel->setMiddlewareAliases($middleware->getMiddlewareAliases());

            if ($priorities = $middleware->getMiddlewarePriority()) {
                $kernel->setMiddlewarePriority($priorities);
            }
        });
    })
```

Realistically, it's better to write long code in a ServiceProvider than in `bootstrap/app.php`.
If you want to "execute after all ServiceProviders," using `booted()` has its purpose, but it's for very specific uses.

Writing it in AppServiceProvider is the same.
```php
class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        info('1. AppServiceProvider register');

        $this->booting(function () {
            info('4. AppServiceProvider booting');
        });

        $this->booted(function () {
            info('5. AppServiceProvider booted');
        });

        $this->app->registered(function () {
            info('2. app registered');
        });

        $this->app->booting(function () {
            info('3. app booting');
        });

        $this->app->booted(function () {
            info('6. AppServiceProvider@register app booted');
        });
    }

    public function boot(): void
    {
        $this->app->booted(function () {
            info('7. AppServiceProvider@boot app booted');
        });
    }
}
```

## withBindings() and withSingletons()
You can just use ServiceProviders, so there's likely little need to write these in `bootstrap/app.php`.

## withMiddleware()
Although scattered throughout the documentation, it should be mostly explained.
https://laravel.com/docs/11.x/middleware

If you want to know the details, you have no choice but to look at the code.
https://github.com/laravel/framework/blob/11.x/src/Illuminate/Foundation/Configuration/Middleware.php

## withExceptions()
It's explained in the documentation, but some parts might still be unclear.
https://laravel.com/docs/11.x/errors

Again, you have no choice but to look at the code.
https://github.com/laravel/framework/blob/11.x/src/Illuminate/Foundation/Configuration/Exceptions.php
https://github.com/laravel/framework/blob/11.x/src/Illuminate/Foundation/Exceptions/Handler.php

- To change the response for each type of exception, use `$exceptions->render()`
- To change the processing just before returning the final response, use `$exceptions->respond()`

Knowing this much should be sufficient for commonly seen questions.
