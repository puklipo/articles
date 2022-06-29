Laravel mixからlaravel-vite-pluginに移行する
----

## はじめに
laravel-vite-plugin  
https://github.com/laravel/vite-plugin  
移行ドキュメントはここ。  
https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md  
公式ドキュメント  
https://laravel.com/docs/vite  
Vite  
https://ja.vitejs.dev/

今後はLaravel mixからViteに置き換わるけど全部の機能を置き換えられるわけではないから「Laravel mixを使ってる既存のプロジェクト」では無理に変える必要はない。  
あくまでもLaravelの想定している普通の使い方ならViteで十分。  
mixでしかできないこともまだあるだろうし、そもそもmixさえ使ってないなら関係ない。

## バージョン
- Laravel 9.19.0以降
- Vite 2.9
- laravel-vite-plugin 0.2.3

## 対象
Laravel 8.x/9.x頃のLaravel公式スターターキットを使ったごく普通のLaravelプロジェクト。

## laravel-vite-pluginの中身
Laravel mixはwebpackを簡単に使えるように色々盛って作られてたけどvite-pluginはもっとシンプル。  
「Viteのプラグイン」でしかなく「Laravel用に自動で設定」程度。1ファイルしかない。  
https://github.com/laravel/vite-plugin/blob/main/src/index.ts

Laravel mixはLaravel外でもよく使ってたけどlaravel-vite-pluginは使う理由はなくViteだけ直接使えばいい。

## 移行作業

### Viteのインストール
0.2.3時点の移行ドキュメントにはないけど`autoprefixer`も必要。今後のバージョンでpackage.jsonに追加されたら不要。
```
npm install --save-dev vite laravel-vite-plugin autoprefixer
```

### vite.config.js ファイルを作る
このファイルで色々設定する部分をプラグインが代行してるだけ。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
// import react from '@vitejs/plugin-react';
// import vue from '@vitejs/plugin-vue';

export default defineConfig({
    plugins: [
        laravel([
            'resources/css/app.css',
            'resources/js/app.js',
        ]),
        // react(),
        // vue({
        //     template: {
        //         transformAssetUrls: {
        //             base: null,
        //             includeAbsolute: false,
        //         },
        //     },
        // }),
    ],
});
```

### npm scriptを変更
```json
"scripts": {
    "dev": "vite",
    "build": "vite build"
}
```

### require()からimportに変更
これを参考に。  
https://github.com/laravel/laravel/pull/5895/files

tailwind.config.jsのrequire()は変更不要。

### InertiaやMIX_から始まる環境変数を使ってるなら
移行ドキュメントを参考に変更。

### SPAならJSからCSSを読み込む
resources/js/app.js
```diff
  import './bootstrap';
+ import '../css/app.css';
```

### mix()から@vite()に変更
```diff
- <link rel="stylesheet" href="{{ mix('css/app.css') }}">
- <script src="{{ mix('js/app.js') }}" defer></script>
+ @vite(['resources/css/app.css', 'resources/js/app.js'])
```

@vite()はLaravel 9.19で追加。

### ReactやVueを使ってるなら
移行ドキュメントを参考に変更。

### Laravel mixを削除
```
npm remove laravel-mix
```
webpack.mix.jsも不要。
```
rm webpack.mix.js
```

public内のビルド済みJS/CSSなど不要なファイルも残っていれば削除。

### Tailwind用にpostcss.config.jsが必要
```
npx tailwindcss init -p
```
`postcss-import`は不要なはずだけどもし必要ならこう指定。
```js
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
    'postcss-import': {},
  },
}
```
app.cssで@importを使ってるなら`postcss-import`が必要。  
@tailwindを使ってるなら不要。

### .gitignoreに追加はそれぞれの方針次第
デプロイ時にビルドするなら追加。
```
/public/build
```
ビルド済みJS/CSSファイルをリポジトリに含めるなら追加しない。

### コマンド
Laravel mixでのwatchは
```
npm run dev
```

prodは
```
npm run build
```

ビルドしたファイルは`public/build/`に作られる。

## 足りない機能
Vite+laravel-vite-pluginでできないことは他のViteプラグイン（またはRollupプラグイン）で追加できるのでもう少し情報が揃えばmixを完全に置き換えられるだろう。

例えばmix.copy()の代わりはこの辺りを使えば良さそう。

- https://github.com/sapphi-red/vite-plugin-static-copy
- https://github.com/mistjs/vite-plugin-copy-files

Vite標準のやり方はpublicディレクトリからのコピーだけどLaravelでは使いにくいのでlaravel-vite-pluginで無効化されている。

- https://ja.vitejs.dev/guide/assets.html#public-%E3%83%87%E3%82%A3%E3%83%AC%E3%82%AF%E3%83%88%E3%83%AA
