Laravel 0からの開発環境構築 Windows11版
----

何もインストールしてない新しいWindows PCでLaravelの開発ができるまでの環境を作る。

最終更新日：2022年12月  
環境構築は「いつ」の情報かが重要なので更新日から何年も後に読んでも役に立たない。

## 対象読者
プログラミング初心者は対象じゃない。Laravelは初心者が使うものじゃない。

すでにLaravelを使っていて大量のLaravelプロジェクトを抱えてる人が別の新しいPCでも開発する場合を想定。  
職場と自宅、WindowsとMacなど複数のPCで併用する前提。  
プロが対象なので有料のツールも遠慮なく含める。

## Laravelを使う前に必須な知識
- 一般以上のPCスキル
- Webに関する幅広い知識
- 素のPHP。最新バージョンまで。composerとPSR。
- git / GitHub
- フロントエンドの知識。最低限node.js/npmや「JavaScriptもビルドして使うようになった」という現代の常識は必須。html→PHPと進んできた初心者が躓くのはフロントの知識が足りてないのが主な原因。
- Linuxの知識。最低でも「`php -v`はターミナルで実行する」が分かる程度。

## Chrome
https://www.google.com/intl/ja_jp/chrome/

細かいプラグインは同期すれば勝手にインストールされるので書かない。以降のアプリも同様。

## GitHub Desktop
https://desktop.github.com/

## VS code
https://code.visualstudio.com/download

VS codeからWindows版gitのインストールを要求されるかもしれない。あまり使わないけど念のためインストール。  
https://git-scm.com/downloads

## PhpStorm
https://www.jetbrains.com/ja-jp/phpstorm/download/

普段の開発はPhpStormを使うけどちょっとした変更程度はVS codeを使うこともあるしGitHub上で直接変更することもある。

## データベースアプリ
ちょうどいいアプリがないのでデータベース(MySQL)の表示・簡易編集はPhpStormを使用。

sailで起動後、

- ホスト：localhost
- ポート：3306
- ユーザーとパスワード：.envで設定したもの。
- データベース：.envのDB_DATABASEで設定したもの。

で接続可能。

## Windows Subsystem for Linux
Microsoft Storeからインストール。

StoreからWSLをインストールしただけの段階では`wsl --version`でバージョンを表示できる。
```shell
wsl --version

WSL バージョン: 1.0.3.0
カーネル バージョン: 5.15.79.1
WSLg バージョン: 1.0.47
MSRDC バージョン: 
Direct3D バージョン: 
DXCore バージョン:
Windowsバージョン: 
```

Windows Terminal(PowerShell)で`wsl --install Ubuntu`を実行。  
ダウンロード後、Ubuntu用の新しいユーザー名とパスワードを決定。

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

以前と比べるとWSLのインストールもかなり簡単になっている。Laravelを使うならこのくらいはできて当然。

これ以降のコマンドはWSLのUbuntuで実行。  
Windows Terminalの設定で既存のプロファイルをUbuntuにしておく。

## WSLでのファイルの扱いを理解する
WSLを使っているとパーミッションのエラーに遭遇しやすい。「Windows側」なのか「WSLのUbuntu側」なのか常に意識してないと間違えやすい。

- Windows側からWSL側のファイルへのアクセス：`\\wsl$`もしくは`\\wsl.localhost\`
- WSL側からWindows側のファイル：`/mnt/c/`
- WSL内：通常のUbuntuと同じ`/` `/home/`

### GitHub Desktopでgit cloneしたらcomposer installでエラーが出る時の修正方法
**（WSLインストール直後に発生しやすいかも。一度再起動後に発生しなければ気にしなくていい）**

前提。  
WSLのUbuntuをインストール時に作った新しいユーザーをuserとするとWSL側のホームは`/home/user/`。  
GitHubからcloneする作業場所を`/home/user/GitHub/`に作ったとする。  

WSL側の`/home/user/GitHub/`はWindows側からは`\\wsl.localhost\Ubuntu\home\user\GitHub\`。  
Windows側のGitHub Desktopで`\\wsl.localhost\Ubuntu\home\user\GitHub\`にcloneするとファイルの所有者がrootになっていてuserからは書き込みできないのでcomposer installでエラーが出る。

確認。rootがあれば所有者がroot。userなら問題ないので以降は不要。
```shell
cd ~/GitHub/
ll
... root root ...
```

修正方法はWSL側で`/home/user/GitHub/`内の所有者を変更する。
```shell
cd
sudo chown user.user -R ./GitHub/
```

修正した後の次からはWSL側でcloneすれば最初からuserなので再発しない。
```shell
cd ~/GitHub/
git clone ...
```
とはいえ普段はGitHub Desktop使いたいので面倒。エラーが出た時の修正方法さえ覚えておけばGitHub Desktopからでも大丈夫。

## Docker Desktop
Dockerを使うためにはWSLが必須。  
https://www.docker.com/products/docker-desktop/

Settings -> Resources -> WSL Integrationで`Ubuntu`を有効化。ここを設定しないとUbuntu内でdockerコマンドなどが使えない。
```
Enable integration with additional distros:

Ubuntu
```

## WSLのUbuntu側にいろいろインストール
「開発環境は全部Docker内に閉じ込める」なんて現実的には机上の空論。そんな使い方は不便すぎるのでDocker外の手元でもphp,composer,npmくらいは使えるようにしておく。

この辺りはバージョンアップで変わるのでそのまま真似しない。

### PHP
artisanやcomposerのためなのでcliだけでもいい。  
SailのDockerfileを見て同じものをインストール。全部は不要なはずだけどcomposer install時に必要なこともあるので一応インストール。足りなければ後で追加。  
https://github.com/laravel/sail/tree/1.x/runtimes

```shell
sudo apt-get install curl zip unzip
```
```shell
LC_ALL=C.UTF-8 sudo add-apt-repository ppa:ondrej/php
```
```shell
sudo apt-get install php8.2-cli php8.2-dev \
       php8.2-pgsql php8.2-sqlite3 php8.2-gd \
       php8.2-curl \
       php8.2-imap php8.2-mysql php8.2-mbstring \
       php8.2-xml php8.2-zip php8.2-bcmath php8.2-soap \
       php8.2-intl php8.2-readline \
       php8.2-ldap \
       php8.2-msgpack php8.2-igbinary php8.2-redis php8.2-swoole \
       php8.2-memcached php8.2-xdebug
```
```shell
php -v
```

composer install時に`ext-***`が足りないみたいなエラーが出た時
```shell
sudo apt-get install php8.2-***
```

### composer
https://getcomposer.org/download/

```shell
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"

php -r "if (hash_file('sha384', 'composer-setup.php') === '55ce33d7678c5a611085589f1f3ddf8b3c52d662cd01d4ba75c0ee0459970c2200a51f492d557530c71c15d8dba01eae') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"

php composer-setup.php

php -r "unlink('composer-setup.php');"

sudo mv composer.phar /usr/local/bin/composer
```
```shell
composer
```

composerをインストールしてないとgit clone後の初回インストールでDockerを使う必要がある。
```
docker run --rm -u "$(id -u):$(id -g)" -v "$(pwd):/var/www/html" -w var/www/html composer/composer:latest install --ignore-platform-reqs
```
これよりもPhpStormでインストールできたほうが楽。

### node.js
https://github.com/nodesource/distributions/blob/master/README.md

```shell
curl -fsSL https://deb.nodesource.com/setup_19.x | sudo -E bash - &&\
sudo apt-get install -y nodejs
```
```shell
node -v
npm -v
```

## sailコマンドのエイリアス登録
```shell
echo "alias sail='[ -f sail ] && sh sail || sh vendor/bin/sail'" >> ~/.bash_aliases
```
これでsailだけで使えるようになる。
```shell
sail version
```
使える場所はcomposer install済のLaravelプロジェクト直下。

## 基本の使い方1 既存のLaravelプロジェクト
GitHubにLaravelプロジェクトがある状態から開始。

先に：↑のほうの所有者の問題がなければGitHub Desktopからcloneしてもいい。これが一番簡単。保存先は必ずWSL側にする。

以降はターミナルからcloneする場合の話。

WSL側
```shell
cd ~/GitHub/
git clone https://.../test.git
cd test
composer install
cp .env.example .env
php artisan key:generate
# 必要なら.envを編集

npm install
npm run build

sail up -d

# migrateなどDBに接続するコマンドは必ずsail内で実行
sail art migrate
# 他のmakeコマンドなどはsail外で使ってもいい。Dockerを使わないので若干早いしsail起動前でも使える。
php artisan make:controller TestController

sail down
```
clone以降はすぐにPhpStormで開いて続きはPhpStormのターミナルやcomposer install機能を使ってもいい。  
PhpStormでcomposerやnpmコマンドを使う場合、インタープリターの選択画面が出る。この時にWSL内のphpやnodeを使うように設定する。
https://pleiades.io/help/phpstorm/configuring-remote-interpreters.html

Windows側  
GitHub Desktopにcloneしたフォルダを追加。  
Add local repository -> Choose  
この時普通にフォルダを選択しようとしても最初は出て来ないのでまず`\\wsl$\Ubuntu\home`を表示してからcloneしたフォルダまで進める。  
Windows側からWSLのファイルを読んでいるので追加時に警告が出るけど問題ないので「add an exception for this directory」を選んで進めて追加する。  
PhpStormで開く時も同様。

PhpStormでコードを書いて、gitはGitHub Desktopを使う、コマンドはターミナル、といういつもの開発体制が整う。

composerやnpmのスクリプトはPhpStormから実行。  
composer.jsonのscriptsにsailのupとdownを書いておけばsailの起動・終了もPhpStormからさっとできる。
```json
        "sail:up": "./vendor/bin/sail up -d",
        "sail:down": "./vendor/bin/sail down",
```
もう一歩進めてsail up後にide-helper:modelsも実行。DBへの接続が必要なのでup直後に毎回実行が効率的。
```json
        "sail:up": [
            "./vendor/bin/sail up -d",
            "./vendor/bin/sail art ide-helper:models -N"
        ],
```

## 基本の使い方2 Laravelプロジェクトを新規作成
Laravelのドキュメント通りなので変わった所はない。

```shell
cd ~/GitHub/
curl -s "https://laravel.build/example-app" | bash
cd example-app
sail up -d
sail down
```

PhpStormやGitHub Desktopの話は↑と同じ。

## テスト
テストはWSLのphpとプロジェクトにインストールしたphpunitでPhpStormの機能で実行。カバレッジ付きで実行すればコードカバレッジの表示もできる。  
毎年11月頃はPHPの新バージョンで動作確認するために「WSLのPHPは8.1のままsailのPHPだけ8.2RCにしてテスト」みたいなことをするので`sail test`を使うこともある。

## ポイント
- Chrome, PhpStorm, GitHub Desktop, VS codeなどのアプリはWindows側で使う。
- その他のファイルの置き場所、ターミナルでのコマンドの実行などはすべてWSL側。ファイルはWindows側に置いてもいいけど実際に使うと速度に差があってWSL側がいいと分かる。
- WSLにインストールしたphp, composer, npmはすべてのLaravelプロジェクト共通で使う。installやupdateに使うだけなので常に最新バージョンでいい。プロジェクトごとの分離した環境はsailで作る。DBなどは分離。
- PhpStormでインタープリターの選択が出てきたらWSLを使うように設定する。ここを徹底しないと上手く動かない。新しい環境では最初だけ色々設定が必要だけど落ち着いたら意識せず使えるようになる。
- Windows環境ではWSLさえ使えれば困らない。WSLがなかった頃と比べると別世界。

## 余談：リモート開発
PhpStormを使うならここまでの使い方で十分だけどPhpStormにもVS codeにもリモート開発機能がある。VS codeをメインに使っていくならリモート開発を使ったほうがいい。

- https://pleiades.io/help/phpstorm/remote-development-starting-page.html
- https://learn.microsoft.com/ja-jp/windows/wsl/tutorials/wsl-vscode

## 余談：初心者がやりがちだけどやってはいけないこと
- XAMPP, phpMyAdminなどは「初心者が書いた間違った本やブログ」に騙された初心者がインストールしようとするけどLaravelユーザーが使うことはないので一切不要。
- Dockerコンテナ内で作業しない。sail登場前からDockerやHomestead(Vagrant)の内部に入ってコマンドを実行する使い方してる初心者が多い。これやると普段使ってるターミナルとは別の環境で作業することになるのでとても不便。考え方が逆。コンテナの外の使い慣れたターミナルからコンテナ内のコマンドを実行する使い方をすべき。Laravel公式のsailはこの辺り良く分かっていて`sail artisan ...`はコンテナの外で実行。コンテナ内で何か作業することは全くない。
