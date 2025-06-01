# Laravel Development Environment Setup on a New PC - Windows 11 Edition

---

This guide explains how to set up a Laravel development environment on a new Windows PC with nothing installed. I wrote this while actually working on a new PC, so it's the best approach as of now.

Last updated: January 2025
The "when" of environment setup information is crucial, so reading this years after the update date won't be helpful.

## Update History
- January 2025: Updated to PHP 8.4.
- April 2024: Updated node.js installation method. Updated to PHP 8.3. It might be getting old as time has passed since the first edition.
- October 2023: Changed node.js installation method to use Installation Scripts.
- February 2023: Changed database application.

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

GitHub Desktop alone is sometimes not enough, so install SourceTree as well.

## VS Code
https://code.visualstudio.com/download

VS Code might ask you to install the Windows version of git. I don't use it much, but install it just in case.
https://git-scm.com/downloads

## PhpStorm
https://www.jetbrains.com/ja-jp/phpstorm/download/

I usually use PhpStorm for development, but I sometimes use VS Code for minor changes or make changes directly on GitHub.

## TablePlus
Database application
https://tableplus.com/

For connection settings, creating one common setting for Sail is sufficient.

- Host: 127.0.0.1
- Port: 3306
- User and Password: Those set in .env. `sail` and `password`
- Database: Empty

Connection is possible after starting Sail. By not specifying a database, it can be used commonly for many Laravel projects. You will need to select the database to display after connecting.

## Windows Subsystem for Linux
Install from the Microsoft Store.

At the stage where you've only installed WSL from the Store, you can display the version with `wsl --version`.
```shell
wsl --version

WSL version: 1.0.3.0
Kernel version: 5.15.79.1
WSLg version: 1.0.47
MSRDC version:
Direct3D version:
DXCore version:
Windows version:
```

In Windows Terminal (PowerShell), run `wsl --install Ubuntu`.
After downloading, decide on a new username and password for Ubuntu.

```shell
Installing, this may take a few minutes...
Please create a default UNIX user account. The username does not need to match your Windows username.
For more information visit: https://aka.ms/wslusers
Enter new UNIX username:
New password:
Retype new password:
passwd: password updated successfully
Installation successful!
```

Compared to before, installing WSL has become considerably easier. If you use Laravel, you should be able to do this much as a matter of course.

Subsequent commands are executed in WSL's Ubuntu.
In Windows Terminal settings, set the existing profile to Ubuntu.

## Understand File Handling in WSL
When using WSL, you are likely to encounter permission errors. It's easy to make mistakes if you're not constantly aware of whether you're on the "Windows side" or the "WSL Ubuntu side."

- Accessing WSL files from the Windows side: `\\wsl$` or `\\wsl.localhost\`
- Accessing Windows files from the WSL side: `/mnt/c/` `/mnt/d/`
- Within WSL: Same as normal Ubuntu `/` `/home/`

### How to fix errors during composer install after git cloning with GitHub Desktop
**(This might occur immediately after installing WSL. If it doesn't happen after a restart, you don't need to worry about it.)**

Prerequisite:
Assuming the new user created when installing WSL Ubuntu is `user`, the WSL home directory is `/home/user/`.
Assume you created a working directory for cloning from GitHub at `/home/user/GitHub/`.

WSL's `/home/user/GitHub/` is `\\wsl.localhost\Ubuntu\home\user\GitHub\` from the Windows side.
If you clone to `\\wsl.localhost\Ubuntu\home\user\GitHub\` using GitHub Desktop on the Windows side, the file owner will be `root`, and `user` won't have write permission, causing errors during `composer install`.

Confirmation. If `root` is present, the owner is `root`. If it's `user`, there's no problem, and the following is unnecessary.
```shell
cd ~/GitHub/
ll
... root root ...
```

To fix this, change the owner of the files within `/home/user/GitHub/` on the WSL side.
```shell
cd
sudo chown user.user -R ./GitHub/
```

After fixing, if you clone from the WSL side next time, the owner will be `user` from the beginning, so it won't happen again.
```shell
cd ~/GitHub/
git clone ...
```
However, I usually want to use GitHub Desktop, so it's a hassle. As long as you remember how to fix the error when it occurs, it's okay to use GitHub Desktop.

## Docker Desktop
WSL is essential for using Docker.
https://www.docker.com/products/docker-desktop/

Enable `Ubuntu` in Settings -> Resources -> WSL Integration. If this is not set, docker commands etc. cannot be used within Ubuntu.
```
Enable integration with additional distros:

Ubuntu
```

## Install various things on the WSL Ubuntu side
"Confine the entire development environment within Docker" is, realistically, an armchair theory. Such usage is too inconvenient, so keep php, composer, and npm readily available outside of Docker as well.

This area changes with version upgrades, so don't just copy it as is.

### PHP
Since it's for artisan and composer, cli alone is fine.
Look at Sail's Dockerfile and install the same things. Not all of them should be necessary, but they are sometimes needed during `composer install`, so install them just in case. If something is missing, add it later.
https://github.com/laravel/sail/tree/1.x/runtimes

(Details omitted so as not to have to update with every new version)

```shell
php -v
```

When an error like `ext-***` is missing during `composer install`
```shell
sudo apt-get install php8.4-***
```

### Use Xdebug in on-demand mode with PhpStorm
https://pleiades.io/help/phpstorm/configuring-xdebug.html#on_demand_mode
Writing tests is common in Laravel, and "step execution" is hardly ever used. So, it's fine to disable Xdebug normally and enable it only when running tests with coverage. This is the fastest way to use it.

- Check the location of php.ini with `php --ini`. For PHP 8.4, it's `/etc/php/8.4/cli/conf.d/20-xdebug.ini`. Change it to `;zend_extension=xdebug.so` to disable it.
- Specify the path to `xdebug.so` in PhpStorm's interpreter settings under "Debugger extension". This changes with PHP version updates. For example, for PHP 8.4, it's `/usr/lib/php/20240924/xdebug.so`.

### composer
Always copy and paste from here.
https://getcomposer.org/download/

```shell
# Run the installation script copied from the download page.

# Move composer.phar to make it usable just by typing composer.
sudo mv composer.phar /usr/local/bin/composer
```
```shell
composer -V
```

If composer is not installed, you need to use Docker for the initial installation after git clone.
```
docker run --rm -u "$(id -u):$(id -g)" -v "$(pwd):/var/www/html" -w var/www/html composer/composer:latest install --ignore-platform-reqs
```
It's easier if you can install it with PhpStorm.

If you plan to install `laravel/installer`. Recently, `laravel.build` is often used, so it's not strictly necessary.
```shell
composer global require laravel/installer

laravel
```
You also need to set the PATH in .bashrc to use composer commands installed globally.
```shell
export PATH=~/.composer/vendor/bin:$PATH
```

### node.js
The installation method from nodesource itself changes occasionally, so always check here.
https://github.com/nodesource/distributions

This is for installing node.js 21, so be sure to check the link above and install the latest version.
```shell
curl -fsSL https://deb.nodesource.com/setup_21.x | sudo -E bash - &&\
sudo apt-get install -y nodejs
```
```shell
node -v
npm -v
```

There are various methods other than installing directly into WSL, but for Laravel use, just using npm is sufficient.

### Upgrade process
```shell
sudo apt update
sudo apt upgrade

composer selfupdate
```

## Register alias for sail command
```shell
echo "alias sail='[ -f sail ] && sh sail || sh vendor/bin/sail'" >> ~/.bash_aliases
```
This allows you to use `sail` by itself.
```shell
sail version
```
It can be used directly in a Laravel project where composer install has been completed.

## Basic Usage 1: Existing Laravel Project
Start with a Laravel project already on GitHub.

First: If there is no owner issue mentioned above, you can clone from GitHub Desktop. This is the easiest. Be sure to save to the WSL side.

The following is for cloning from the terminal.

WSL side
```shell
cd ~/GitHub/
git clone https://.../test.git
cd test
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
After cloning, you can immediately open it in PhpStorm and continue using PhpStorm's terminal or composer install function.
When using composer or npm commands in PhpStorm, the interpreter selection screen will appear. At this time, configure it to use php and node within WSL.
https://pleiades.io/help/phpstorm/configuring-remote-interpreters.html

Windows side
Add the cloned folder to GitHub Desktop.
Add local repository -> Choose
At this time, if you try to select the folder normally, it won't appear at first, so first display `\\wsl$\Ubuntu\home` and then navigate to the cloned folder.
Since you are reading WSL files from the Windows side, a warning will appear when adding, but there is no problem, so select "add an exception for this directory" and proceed to add.
The same applies when opening in PhpStorm.

This sets up the usual development system: writing code in PhpStorm, using GitHub Desktop for git, and the terminal for commands.

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
cd ~/GitHub/
curl -s "https://laravel.build/example-app" | bash
cd example-app
sail up -d
sail down
```

The discussion about PhpStorm and GitHub Desktop is the same as above.

## Tests
Run tests using WSL's php and phpunit installed in the project with PhpStorm's features. Running with coverage also displays code coverage.
Around November each year, to check compatibility with new PHP versions, I sometimes do things like "keep WSL's PHP at 8.1 and set sail's PHP to 8.2RC for testing," so I also use `sail test`.

## Points
- Use apps like Chrome, PhpStorm, GitHub Desktop, and VS Code on the Windows side.
- All other file locations, execution of commands in the terminal, etc., are on the WSL side. You can place files on the Windows side, but you'll find that there's a speed difference in actual use, and the WSL side is better.
- Use php, composer, and npm installed in WSL commonly for all Laravel projects. Since they are only used for install and update, the latest version is always fine. Create isolated environments for each project with sail. DBs, etc., are separated.
- When the interpreter selection appears in PhpStorm, set it to use WSL. If this is not done thoroughly, it will not work properly. In a new environment, various settings are required only at the beginning, but once settled, you can use it without thinking about it.
- In a Windows environment, as long as you can use WSL, you won't have any problems. It's a different world compared to when WSL didn't exist.

## Aside: Remote Development
If you use PhpStorm, the usage described so far is sufficient, but both PhpStorm and VS Code have remote development features. If you mainly use VS Code, it's better to use remote development.

- https://pleiades.io/help/phpstorm/remote-development-starting-page.html
- https://learn.microsoft.com/ja-jp/windows/wsl/tutorials/wsl-vscode

## Aside: Things beginners tend to do but shouldn't
- XAMPP, phpMyAdmin, etc., are things that beginners deceived by "wrong books and blogs written by beginners" try to install, but Laravel users never use them, so they are completely unnecessary.
- Do not work inside Docker containers. Many beginners, even before sail appeared, have been entering Docker or Homestead (Vagrant) internals to execute commands. This is very inconvenient as you end up working in an environment different from your usual terminal. The thinking is reversed. You should use your familiar terminal outside the container to execute commands inside the container. Laravel's official sail understands this well, and `sail artisan ...` is executed outside the container. There is no need to do any work inside the container at all.
