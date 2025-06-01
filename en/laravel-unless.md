# PHP Lacks `unless`, so Laravel's Similar Features are Subtly Useful

---

Negation with `!` like in `if(! $test)` is commonly used even within Laravel's internal code, but personally, I prefer to avoid it. It's a usage that relies on PHP's loose type coercion. Within Laravel, it's just an inversion with `!`, so in terms of actual behavior, being particular about it is meaningless.

## Versions
- Laravel 9.x
- PHP 8.1

## Blade's @unless
```php
@unless (Auth::check())
    You are not signed in.
@endunless
```

https://laravel.com/docs/9.x/blade#if-statements

## Collection's doesntContain()
```php
$collection = collect(['name' => 'Desk', 'price' => 100]);

if($collection->doesntContain('Table')) {

}
```

https://laravel.com/docs/9.x/collections#method-doesntcontain

Internally, Laravel just inverts `contains()`.
```php
    public function doesntContain($key, $operator = null, $value = null)
    {
        return ! $this->contains(...func_get_args());
    }
```

## Eloquent / QueryBuilder's doesntExist()
```php
if (DB::table('orders')->where('finalized', 1)->doesntExist()) {
    // ...
}
```

https://laravel.com/docs/9.x/queries#determining-if-records-exist

## Conditionable Trait's when()/unless()
Since it's a trait, it can be used in various places, but it's often used with Collections and QueryBuilders.
```php
$collection = collect([1, 2, 3]);

$collection->unless(true, function ($collection) {
    return $collection->push(4); // This will not be executed
});

$collection->unless(false, function ($collection) {
    return $collection->push(5); // This will be executed
});

$collection->all();

// [1, 2, 3, 5]
// The result is only 5, not 4.
```

`when()` can be used to filter only when a condition is specified in a search form.

```php
$role = $request->input('role');

$users = DB::table('users')
                ->when($role, function ($query, $role) {
                    $query->where('role_id', $role);
                })
                ->get();
```

I don't want to write `when($role` like this.
```php
->when(filled($role), function ($query, $b) use($role)) {
```
Check with `filled()` as the opposite of `empty()`. Since `$role` is not passed directly, pass it separately using `use($role)`.
```php
->unless(blank($role),
```
For `unless`, use `blank()` or `empty()`.

https://github.com/laravel/framework/blob/9.x/src/Illuminate/Conditionable/Traits/Conditionable.php
https://laravel.com/docs/9.x/collections#method-unless
https://laravel.com/docs/9.x/queries#conditional-clauses

Additionally, in this situation, the presence or absence of `return` makes no difference.
```php
->when(filled($role), function ($query) use($role) {
    return $query->where('role_id', $role);
})
```
So, you can also write it with an arrow function.
```php
->when(filled($role), fn ($query) => $query->where('role_id', $role))
```

## abort_unless()
```php
abort_unless(Auth::user()->isAdmin(), 403);
```
```php
abort_if(! Auth::user()->isAdmin(), 403);
```
The documentation also seems to imply that `unless` is preferable to inversion with `!`.

https://laravel.com/docs/9.x/helpers#method-abort-unless

```php
    function abort_unless($boolean, $code, $message = '', array $headers = [])
    {
        if (! $boolean) {
            abort($code, $message, $headers);
        }
    }
```
Here too, Laravel uses `!` internally, but "framework code and userland code are different," so I still want to avoid `!` in userland code.

## missing
The opposite of `has` is `missing`. Used in tests like `assertDatabaseMissing()` and `assertJsonMissing()`, and Request's `$request->missing()`.

```php
if ($request->missing('name')) {
    //
}
```
