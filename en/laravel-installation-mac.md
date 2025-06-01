# Laravel Development Environment Setup for Mac (Apple Silicon)

---

Unlike the [Windows version](./laravel-installation-windows.md), I haven't worked on a completely new PC, but the setup method for Mac hasn't changed, so it shouldn't be an issue.

Last updated: February 2023
Environment setup information's "when" is crucial, so reading this years after the update date won't be helpful.

## Update History
- February 2023: Rewrote for Apple Silicon as I actually set up the environment on a new Mac. I used Migration Assistant, so it wasn't a completely fresh install, but since it was a change to Apple Silicon, various reinstallations were necessary.

## Target Audience
This is not for programming beginners. Laravel is not for beginners.

This assumes someone who already uses Laravel and has many Laravel projects is setting up development on a different new PC.
It assumes use on multiple PCs, such as at work and home, or Windows and Mac.
Since professionals are the target, paid tools are included without hesitation.

## Essential Knowledge Before Using Laravel
A framework is "something to align the lower limit of knowledge." The official Laravel documentation proceeds with the assumption that you know this much.

- General PC skills and above
- Broad knowledge of the Web
- Plain PHP. Up to the latest version. composer and PSR.
- git / GitHub
- Front-end knowledge. At a minimum, modern common sense like "JavaScript is now built and used" with node.js/npm is essential. The main reason beginners who have progressed from html to PHP stumble is a lack of front-end knowledge.
- Linux knowledge. At the very least, understanding that "`php -v` is executed in the terminal."

## Chrome
https://www.google.com/intl/ja_jp/chrome/

Detailed plugins will be installed automatically when synced, so I won't write them down. The same applies to subsequent apps.

## GitHub Desktop
https://desktop.github.com/

## SourceTree
https://www.sourcetreeapp.com/

GitHub Desktop alone is sometimes not enough, so install SourceTree as well. On Mac, SourceTree is the main one.

## VS Code
https://code.visualstudio.com/download

## PhpStorm
https://www.jetbrains.com/ja-jp/phpstorm/download/

I usually use PhpStorm for development, but I sometimes use VS Code for minor changes or make changes directly on GitHub.

## Sequel Ace
https://sequel-ace.com/

For connection settings, creating one common setting for Sail is sufficient.

- TCP/IP
- Host: `localhost` or `127.0.0.1`
- Port: 3306
- User and Password: Those set in .env. Default sail and password are fine.
- Database: Empty

Connection is possible after starting Sail. By not specifying a database, it can be used commonly for many Laravel projects. You will need to select the database to display after connecting.

If you want to share with Windows, TablePlus is also fine.
https://tableplus.com/

## Docker Desktop
https://www.docker.com/products/docker-desktop/

## iTerm2 or other terminal app
https://iterm2.com/

The standard terminal is also fine.

Although not written here, customize zsh to your liking.

## Xcode Command Line Tools
It might seem unrelated to Laravel, but it's sometimes needed by Homebrew or can affect node.js/npm, so install it.
```shell
xcode-select --install
```

Installing Xcode from the Mac App Store shouldn't be necessary, but if problems arise later, install it.

## Homebrew
https://brew.sh/index_ja

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
```shell
brew -v
```

Guidance to add to `.zprofile` will appear during installation, but ignore this if you use PhpStorm.
```shell
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> /Users/***/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```
Add it to `.zshrc` instead of `.zprofile`. The cause is unclear, but the installation destination may change, causing PHP/composer execution from PhpStorm to fail. Adding it to `.zshrc` solves the problem.
```shell
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> /Users/***/.zshrc
eval "$(/opt/homebrew/bin/brew shellenv)"
```

On Apple Silicon, Homebrew's installation destination has changed to `/opt/homebrew/`. If you migrate from an old model using Migration Assistant, the old Homebrew will remain in `/usr/local/`, so reinstall with the new Homebrew.

```shell
cd ~/

# Create Brewfile with old brew
/usr/local/bin/brew bundle dump

# Reinstall with new brew
brew bundle
```

Commands installed with the old Homebrew can be deleted.

Since the installation destination has changed, all PATHs set in various places will also change. Things that were previously recognized as `/usr/local/bin/php` as a matter of course will change, so you need to forget everything and relearn.

## Install various things with Homebrew
"Confine the entire development environment within Docker" is, realistically, an armchair theory. Such usage is too inconvenient, so keep php, composer, and npm readily available outside of Docker as well.

It's easy because you just install the latest version of everything with brew.

### PHP
The brew version automatically installs extensions, so you'll rarely have problems within the scope of Laravel usage.
```shell
brew install php
```
```shell
php -v
```

### Xdebug
Install with pecl after PHP.

```shell
pecl install xdebug
```

To use Xdebug in on-demand mode with PhpStorm:
https://pleiades.io/help/phpstorm/configuring-xdebug.html#on_demand_mode
Writing tests is common in Laravel, and "step execution" is hardly ever used. So, it's fine to disable Xdebug normally and enable it only when running tests with coverage. This is the fastest way to use it.

- After installing with pecl, Xdebug is automatically enabled, so edit php.ini to disable it.
  - Check the location of php.ini with `php --ini`. For PHP 8.2, it's `/opt/homebrew/etc/php/8.2/php.ini`. Delete the line `zend_extension="xdebug.so"` or disable it with `;zend_extension="xdebug.so"`.
- Specify the path to `xdebug.so` in PhpStorm's interpreter settings under "Debugger extension". This changes with PHP version updates, so check the display after installing with pecl. For example, for PHP 8.2, it's `/opt/homebrew/Cellar/php/8.2.0/pecl/20220829/xdebug.so`.
- Do not copy and paste these paths as they change depending on the environment and version.

### composer
```shell
brew install composer
```
```shell
composer
```

If you plan to install `laravel/installer`. Recently, `laravel.build` is often used, so it's not strictly necessary.
```shell
composer global require laravel/installer

laravel
```

### node.js
```shell
brew install node
```
```shell
node -v
npm -v
```

### Upgrade process
```shell
brew update
brew upgrade
```

## Register alias for sail command
```shell
echo "alias sail='[ -f sail ] && sh sail || sh vendor/bin/sail'" >> ~/.zshrc
```
This allows you to use `sail` by itself.
```shell
sail version
```
It can be used directly in a Laravel project where composer install has been completed.

## Basic Usage 1: Existing Laravel Project
Start with a Laravel project already on GitHub.

On Mac, there are no owner issues like with WSL, so cloning from SourceTree or GitHub Desktop is fine. You can save it anywhere, like `~/Sites/` or `~/Documents/`.
Opening it with PhpStorm should prompt you to install composer and npm dependencies. You can execute them using PhpStorm's features, or in "PhpStorm's terminal" or "iTerm2, etc."

```shell
composer install
cp .env.example .env
php artisan key:generate
# Edit .env if necessary

npm install
npm run build

sail up -d

# Always execute commands that connect to the DB, like migrate, within sail
sail art migrate
# Other make commands can be used outside of sail. It's slightly faster as it doesn't use Docker and can be used even before sail starts.
php artisan make:controller TestController

sail down
```

This sets up the usual development system: writing code in PhpStorm, using SourceTree/GitHub Desktop for git, and the terminal for commands.

Run composer and npm scripts from PhpStorm.
If you write sail up and down in composer.json's scripts, you can quickly start and stop sail from PhpStorm.
```json
        "sail:up": "./vendor/bin/sail up -d",
        "sail:down": "./vendor/bin/sail down",
```
Taking it a step further, run ide-helper:models after sail up. Since a DB connection is required, running it immediately after up each time is efficient.
```json
        "sail:up": [
            "./vendor/bin/sail up -d",
            "./vendor/bin/sail art ide-helper:models -N"
        ],
```

## Basic Usage 2: Create a New Laravel Project
There are no changes from the official Laravel documentation.

```shell
cd ~/Sites/
curl -s "https://laravel.build/example-app" | bash
cd example-app
sail up -d
sail down
```

## Tests
Run tests using brew's php and phpunit installed in the project with PhpStorm's features. Running with coverage also displays code coverage.
Around November each year, to check compatibility with new PHP versions, I sometimes do things like "keep brew's PHP at 8.1 and set sail's PHP to 8.2RC for testing," so I also use `sail test`.

## Points
- Use php, composer, and npm installed with brew commonly for all Laravel projects. Since they are only used for install and update, the latest version is always fine. Create isolated environments for each project with sail. DBs, etc., are separated.
- Since the apps and usage are the same, there's no awkwardness even when using Mac and Windows together.
- Although WSL integrates well, it forcefully puts Linux inside Windows, so there are various things to be careful about. However, Mac, which has been UNIX-based for a long time, has no such issues.

## Aside: Things beginners tend to do but shouldn't
- MAMP, phpMyAdmin, etc., are things that beginners deceived by "wrong books and blogs written by beginners" try to install, but Laravel users never use them, so they are completely unnecessary.
- Do not work inside Docker containers. Many beginners, even before sail appeared, have been entering Docker or Homestead (Vagrant) internals to execute commands. This is very inconvenient as you end up working in an environment different from your usual terminal. The thinking is reversed. You should use your familiar terminal outside the container to execute commands inside the container. Laravel's official sail understands this well, and `sail artisan ...` is executed outside the container. There is no need to do any work inside the container at all.
