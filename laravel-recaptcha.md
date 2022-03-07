LaravelでreCAPTCHA
----

Laravel標準のユーザー登録機能を使ってる人なら「最近ボットによる自動登録が増えてるなぁ」が分かると思う。  
`/register`なURLとフォームが固定なので登録だけなら簡単。メール確認機能も併用すれば登録より先には進めないけど登録さえもさせたくないなら何らかの対策が必要。 
社内の限られた数のユーザーが登録するだけなら共通の「合言葉」を決めて入力しないと登録できないようにするとか簡易的な対策はあるけど、誰でも登録できる一般的なサイトならreCAPTCHAを使った対策が普通。  

reCAPTCHAで防げなかったら他の方法を検討。

## バージョン
- Laravel 9.x(+Jetstream/Livewire)
- PHP 8.1
- Google reCAPTCHA v2
- biscolab/laravel-recaptcha 5.3

## reCAPTCHA
`biscolab/laravel-recaptcha`を使用。  
https://github.com/biscolab/laravel-recaptcha  
https://laravel-recaptcha-docs.biscolab.com/

## インストール
```
composer require biscolab/laravel-recaptcha
```

.envと.env.exampleに追加。
```
RECAPTCHA_SITE_KEY=
RECAPTCHA_SECRET_KEY=
```

## Googleでキーを取得
https://www.google.com/recaptcha/

よく見る「私はボットではありません」を使うなら`reCAPTCHA v2`→`「私はロボットではありません」チェックボックス`を選択。  
他でもいいので好きなタイプを選ぶ。  
https://developers.google.com/recaptcha/docs/versions

Google側でドメインなどを正しく設定しておく。`localhost`など開発環境のドメインも追加するけどこれは後で削除する。

## キーを設定（開発環境用）
.envで設定。
```
RECAPTCHA_SITE_KEY=
RECAPTCHA_SECRET_KEY=
```

## viewの変更
Jetstreamの場合。  
`layouts/guest.blade.php`の`</head>`直前に`htmlScriptTagJsApi()`追加。
```php
        <!-- Scripts -->
        <script src="{{ mix('js/app.js') }}" defer></script>

        {!! htmlScriptTagJsApi() !!}
    </head>
```

`auth/register.blade.php`の登録ボタンの上辺りに`htmlFormSnippet()`追加。
```php
            <div class="mt-4">
                {!! htmlFormSnippet() !!}
            </div>
```

Jetstreamではない場合はそれぞれの環境に合わせる。

## バリデーションの追加
Jetstreamの場合は`app/Actions/Fortify/CreateNewUser.php`。 
```php
        Validator::make($input, [
            'name' => ['required', 'string', 'max:255'],
            'email' => ['required', 'string', 'email', 'max:255', 'unique:users'],
            'password' => $this->passwordRules(),
            recaptchaFieldName() => recaptchaRuleName(),
            'terms' => Jetstream::hasTermsAndPrivacyPolicyFeature() ? ['required', 'accepted'] : '',
        ])->validate();
```

`recaptchaFieldName() => recaptchaRuleName(),`はどちらも`biscolab/laravel-recaptcha`が用意しているヘルパー。  
ヘルパー使わずに`'g-recaptcha-response' => 'recaptcha',`でもいい。

## 開発環境で一旦確認
Google側でドメインを設定すれば開発環境でも動くのでreCAPTCHA次第で登録が成功・失敗することを手動で確認。

## 開発環境ではreCAPTCHA無効にする(v2)
十分確認したら開発環境ではもう不要。  
ここにテスト用のキーがあるので.envで設定すればreCAPTCHAに関係なくバリデーションは通るようなる。
```
Site key: 6LeIxAcTAAAAAJcZVRqyHh71UMIEGNQ_MXjiZKhI
Secret key: 6LeIxAcTAAAAAGG-vFI1TnRWxMZNFuojJ4WifJWe
```
https://developers.google.com/recaptcha/docs/faq#id-like-to-run-automated-tests-with-recaptcha.-what-should-i-do

```
RECAPTCHA_SITE_KEY=6LeIxAcTAAAAAJcZVRqyHh71UMIEGNQ_MXjiZKhI
RECAPTCHA_SECRET_KEY=6LeIxAcTAAAAAGG-vFI1TnRWxMZNFuojJ4WifJWe
#本番用
#RECAPTCHA_SITE_KEY=
#RECAPTCHA_SECRET_KEY=
```

Google側でlocalhostなどのドメインを削除。

## phpunit用にもテストキー追加
CIでのテスト用。  
phpunit.xml
```xml
        <!--テスト用キー-->
        <env name="RECAPTCHA_SITE_KEY" value="6LeIxAcTAAAAAJcZVRqyHh71UMIEGNQ_MXjiZKhI"/>
        <env name="RECAPTCHA_SECRET_KEY" value="6LeIxAcTAAAAAGG-vFI1TnRWxMZNFuojJ4WifJWe"/>
```

Jetstreamが用意してるテストにも通る。

## 本番環境での設定
.envに本番用キーを設定。

動作確認して完了。

## reCAPTCHA v3を使うには
`biscolab/laravel-recaptcha`はv2がデフォルトなのでconfigファイルを公開してv3を使うように変更。  
後はドキュメントを参照。  
https://laravel-recaptcha-docs.biscolab.com/docs/how-to-use-v3


