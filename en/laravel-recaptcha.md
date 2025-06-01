# reCAPTCHA in Laravel

---

If you're using Laravel's standard user registration feature, you've probably noticed an increase in automated bot registrations recently.
Since the `/register` URL and form are fixed, registration itself is easy. If you also use the email confirmation feature, they can't proceed beyond registration, but if you want to prevent even registration, some countermeasures are necessary.
If only a limited number of users within a company will register, simple measures like setting a common "password" that must be entered to register exist. However, for a general site where anyone can register, using reCAPTCHA is a common countermeasure.

If reCAPTCHA doesn't work, consider other methods.

## Versions
- Laravel 9.x (+Jetstream/Livewire)
- PHP 8.1
- Google reCAPTCHA v2
- biscolab/laravel-recaptcha 5.3

## reCAPTCHA
Uses `biscolab/laravel-recaptcha`.
https://github.com/biscolab/laravel-recaptcha
https://laravel-recaptcha-docs.biscolab.com/

### Installation
```
composer require biscolab/laravel-recaptcha
```

Add to .env and .env.example.
```
RECAPTCHA_SITE_KEY=
RECAPTCHA_SECRET_KEY=
```

## Get Keys from Google
Site key and secret key.
https://www.google.com/recaptcha/

If you want to use the common "I'm not a robot" checkbox, select `reCAPTCHA v2` -> `"I'm not a robot" Checkbox`.
Other types are fine too, so choose the one you like.
https://developers.google.com/recaptcha/docs/versions

Correctly set up your domain on the Google side. Add development environment domains like `localhost`, but delete these later.

## Set Keys (for Development Environment)
Set in .env.
```
RECAPTCHA_SITE_KEY=
RECAPTCHA_SECRET_KEY=
```

## View Changes
For Jetstream:
Add `htmlScriptTagJsApi()` right before `</head>` in `layouts/guest.blade.php`.
```php
        <!-- Scripts -->
        <script src="{{ mix('js/app.js') }}" defer></script>

        {!! htmlScriptTagJsApi() !!}
    </head>
```

Add `htmlFormSnippet()` around the registration button in `auth/register.blade.php`.
```php
            <div class="mt-4">
                {!! htmlFormSnippet() !!}
            </div>
```

If not using Jetstream, adapt to your specific environment.

## Add Validation
For Jetstream, in `app/Actions/Fortify/CreateNewUser.php`.
```php
        Validator::make($input, [
            'name' => ['required', 'string', 'max:255'],
            'email' => ['required', 'string', 'email', 'max:255', 'unique:users'],
            'password' => $this->passwordRules(),
            recaptchaFieldName() => recaptchaRuleName(),
            'terms' => Jetstream::hasTermsAndPrivacyPolicyFeature() ? ['required', 'accepted'] : '',
        ])->validate();
```

Both `recaptchaFieldName()` and `recaptchaRuleName()` are helpers provided by `biscolab/laravel-recaptcha`.
You can also use `'g-recaptcha-response' => 'recaptcha',` instead of the helpers.

## Temporary Confirmation in Development Environment
If you set the domain on the Google side, it will work even in the development environment, so manually check that registration succeeds or fails depending on reCAPTCHA.

## Disable reCAPTCHA in Development Environment (v2)
Once thoroughly confirmed, it's no longer needed in the development environment.
Test keys are available here, so setting them in .env will allow validation to pass regardless of reCAPTCHA.
```
Site key: 6LeIxAcTAAAAAJcZVRqyHh71UMIEGNQ_MXjiZKhI
Secret key: 6LeIxAcTAAAAAGG-vFI1TnRWxMZNFuojJ4WifJWe
```
https://developers.google.com/recaptcha/docs/faq#id-like-to-run-automated-tests-with-recaptcha.-what-should-i-do

```
RECAPTCHA_SITE_KEY=6LeIxAcTAAAAAJcZVRqyHh71UMIEGNQ_MXjiZKhI
RECAPTCHA_SECRET_KEY=6LeIxAcTAAAAAGG-vFI1TnRWxMZNFuojJ4WifJWe
#For production
#RECAPTCHA_SITE_KEY=
#RECAPTCHA_SECRET_KEY=
```

Remove domains like localhost from the Google side.

## Add Test Keys for phpunit as well
For testing in CI.
phpunit.xml
```xml
        <!--Test Keys-->
        <env name="RECAPTCHA_SITE_KEY" value="6LeIxAcTAAAAAJcZVRqyHh71UMIEGNQ_MXjiZKhI"/>
        <env name="RECAPTCHA_SECRET_KEY" value="6LeIxAcTAAAAAGG-vFI1TnRWxMZNFuojJ4WifJWe"/>
```

This will also pass the tests provided by Jetstream.

## Production Environment Settings
Set the production keys in .env.

Confirm it works, and you're done.

## Using reCAPTCHA v2 invisible
Publish the config file:
```
php artisan vendor:publish --provider="Biscolab\ReCaptcha\ReCaptchaServiceProvider"
```
Change config/recaptcha.php to use `invisible`.
```php
'version'                      => 'invisible',
```

`htmlScriptTagJsApi()` is the same as for v2.

For `auth/register.blade.php`, add an id to the form:
```php
<form id="{{ getFormId() }}">
```
And add data-sitekey etc. to the button (for Jetstream):
```php
<x-jet-button class="ml-4 g-recaptcha"
              data-callback="biscolabLaravelReCaptcha"
              data-sitekey="{{ config('recaptcha.api_site_key') }}">
    {{ __('Register') }}
</x-jet-button>
```
Using `htmlFormButton()` with Jetstream results in many classes, so adding data-sitekey yourself seems better.

https://laravel-recaptcha-docs.biscolab.com/docs/how-to-use-v2#recaptcha-v2-invisible

## Using reCAPTCHA v3
`biscolab/laravel-recaptcha` defaults to v2, so publish the config file and change it to use v3.
Then refer to the documentation.
https://laravel-recaptcha-docs.biscolab.com/docs/how-to-use-v3
