# Migrating from Laravel Mix to laravel-vite-plugin

---

## Introduction
laravel-vite-plugin
https://github.com/laravel/vite-plugin
Migration documentation is here:
https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md
Official documentation:
https://laravel.com/docs/vite
Vite:
https://vitejs.dev/

Vite will replace Laravel Mix in the future, but it doesn't replace all features, so there's no need to forcibly change existing projects that use Laravel Mix.
Vite is sufficient for typical Laravel usage as intended by the framework.
There are likely still things that only Mix can do, and if you're not even using Mix, this doesn't concern you.

## Versions
- Laravel 9.19.0 or later
- Vite 2.9
- laravel-vite-plugin 0.2.3

## Target
A very standard Laravel project using the official Laravel starter kits around Laravel 8.x/9.x.

## Inside laravel-vite-plugin
Laravel Mix was built with a lot of features to make webpack easier to use, but vite-plugin is simpler.
It's just a "Vite plugin" and mainly provides "automatic configuration for Laravel." It's only one file.
https://github.com/laravel/vite-plugin/blob/main/src/index.ts

While Laravel Mix was often used outside of Laravel, there's no reason to use laravel-vite-plugin; you can just use Vite directly.

## Migration Steps

### Install Vite
Although not in the migration documentation as of version 0.2.3, `autoprefixer` is also required. This will be unnecessary if it's added to package.json in future versions.
```
npm install --save-dev vite laravel-vite-plugin autoprefixer
```

### Create vite.config.js file
The plugin simply handles the various settings configured in this file.

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

### Change npm scripts
```json
"scripts": {
    "dev": "vite",
    "build": "vite build"
}
```

### Change require() to import
Refer to this:
https://github.com/laravel/laravel/pull/5895/files

No need to change `require()` in `tailwind.config.js`.

### If using Inertia or environment variables starting with MIX_
Refer to the migration documentation for changes.

### For SPAs, load CSS from JS
resources/js/app.js
```diff
  import './bootstrap';
+ import '../css/app.css';
```

### Change mix() to @vite()
```diff
- <link rel="stylesheet" href="{{ mix('css/app.css') }}">
- <script src="{{ mix('js/app.js') }}" defer></script>
+ @vite(['resources/css/app.css', 'resources/js/app.js'])
```

`@vite()` was added in Laravel 9.19.

### If using React or Vue
Refer to the migration documentation for changes.

### Remove Laravel Mix
```
npm remove laravel-mix
```
`webpack.mix.js` is also no longer needed.
```
rm webpack.mix.js
```

If any built JS/CSS files or other unnecessary files remain in the `public` directory, delete them.

### postcss.config.js is needed for Tailwind
```
npx tailwindcss init -p
```
`postcss-import` should not be necessary, but if it is, specify it like this:
```js
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
    'postcss-import': {},
  },
}
```
If you are using `@import` in `app.css`, `postcss-import` is necessary.
If you are using `@tailwind`, it is not necessary.

### Adding to .gitignore depends on your policy
Add if you build during deployment.
```
/public/build
```
Do not add if you include built JS/CSS files in the repository.

### Commands
For `watch` in Laravel Mix:
```
npm run dev
```

For `prod`:
```
npm run build
```

Built files are created in `public/build/`.

## Missing Features
Things that cannot be done with Vite + laravel-vite-plugin can be added with other Vite plugins (or Rollup plugins), so once more information is available, Mix will likely be completely replaceable.

For example, these could be used as alternatives to `mix.copy()`:

- https://github.com/sapphi-red/vite-plugin-static-copy
- https://github.com/mistjs/vite-plugin-copy-files

Vite's standard way is to copy from the `public` directory, but this is difficult to use with Laravel, so it is disabled by `laravel-vite-plugin`.

- https://vitejs.dev/guide/assets.html#the-public-directory
