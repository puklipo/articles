---
title: "Laravel is Not MVC"
emoji: "©️"
type: "tech" # tech: Technical article / idea: Idea
topics: ["Laravel"]
published: true
---

# Laravel is Not MVC

I wrote about this in my Zenn book, but I'm also making it an article.

https://zenn.dev/pcs_engineer/books/re-laravel-1

https://zenn.dev/pcs_engineer/books/re-laravel-2

## Prerequisite
This assumes an understanding of MVC (including the differences between MVC in the context of PC applications and MVC in the context of web frameworks). The point is that you can ignore MVC when using Laravel.

If someone who doesn't understand MVC ignores MVC, they will only write worse code than "someone who remembers MVC incorrectly." I've seen many beginners writing code that should be in a controller directly in a Blade template using `@php ... @endphp`. People at that stage should learn MVC first.

## Laravel's Model is About the Database/Eloquent
`Illuminate\Database\Eloquent\Model` This namespace explains everything.
https://github.com/laravel/framework/blob/10.x/src/Illuminate/Database/Eloquent/Model.php

It has no responsibilities other than the database.

Eloquent already has too many features, so it's best to write User and Post models declaratively without adding extra functionality. Only define standard Laravel features like relationships, accessors, and scopes. (People coming from other frameworks often make the mistake of thinking validation is the model's responsibility, but it's not.)

"Business logic should be written in the model"? That's not talking about Laravel's model.

## Laravel's Controller is Part of Routing
`Illuminate\Routing\Controller`
https://github.com/laravel/framework/blob/10.x/src/Illuminate/Routing/Controller.php

It's just organizing the request handling logic into Controller classes instead of writing everything in route files.

> Instead of defining all of your request handling logic as closures in your route files, you may wish to organize this behavior using "controller" classes.
https://laravel.com/docs/10.x/controllers

"You shouldn't write complex processing in controllers"? The Laravel documentation doesn't say that.
Rather, it recommends creating single-action controllers for complex cases. All processing from routing can be written in the controller. If the same processing needs to be used elsewhere, then it can be separated into another class. (Artisan commands often reuse the same processing, so examples are written using service classes from the beginning.)

Although it's named Controller, it's closer in reality to a ViewModel in MVVM. As a ViewModel, it's also the place to write business logic. If you assume it's MVC, the roles of model and controller are reversed, leading to mistakes. Laravel doesn't call it a ViewModel, so we continue to treat it as a controller.

## More Important Than MVC
Forget about MVC; thoroughly apply the "Single Responsibility Principle." Writing long processing in a controller is not what makes a Fat Controller. A Fat Controller is the result ofどんどん adding completely unrelated processing later. There are countless real-world examples of "writing Post processing that is unrelated to UserController."

It's meaningless if you write this in UserController and say, "I solved the Fat Controller by separating it into PostService."
```php:UserController
public function postIndex(PostService $postService)
{
    $posts = $postService->list();
    return view('post.index')->with(compact('posts'));
}
```
It's far better to fetch directly from the Post model in PostController. With Laravel, it's easy to write tests, so this is fine at first.
```php:PostController
public function index()
{
    $posts = Post::latest()->paginate();
    return view('post.index')->with(compact('posts'));
}
```
The next step after this is "organizing using Laravel and PHP features like scopes and traits."

Separating into another class is a step further down the line.

In the past (from Laravel 4 to early 5), I used to create Repositories and Services, but I don't use them that way at all anymore.

## Don't Arbitrarily Bring Concepts from Other Frameworks into Laravel
Rails (7.x) explicitly states it's MVC, so it is MVC.
https://guides.rubyonrails.org/action_controller_overview.html

CakePHP (4.x) also explicitly states it's MVC.
https://book.cakephp.org/4/en/controllers.html
`Cake\Controller\Controller`

The word "MVC" never appears in the Laravel documentation.
