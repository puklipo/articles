Laravel入門 いにしえの掲示板を作る パート1
----

20世紀末のインターネットではPerl CGIの掲示板が個人ページ内でのコミュニケーションの主流だった。レンタルサーバーにbbs.cgiを設置した人も多いだろう。（実際には自分で設置せずレンタルが大半）

当時くらいの掲示板を作りながらLaravelの入門レベルの説明をしていく。掲示板作りは入門にちょうどいい。最近の入門はTwitterもどきが多いけど考えれば分かるようにやってることは掲示板でもブログでもTwitterでも似たようなもの。

懐かしいと思った人でも思い浮かべる掲示板はいろいろ違うだろう。よく使われてた掲示板も今から考えれば初心者による拙い作り。実は当時でも上級者によるよくできた掲示板も存在したけど設置する側が初心者で扱える人が少なかった。今回作る掲示板は後者の仕様を思い出して作る。

## 仕様
- ユーザー登録不要で誰でも書き込める掲示板。
- 親投稿に対してコメントも書ける。
- 入力項目はタイトル・本文・名前・メール・アイコン選択・削除編集用パスワード。本文以外は省略可能。昔とは違うのでメールは表示しない。コメントはタイトルなし。
- html入力は一切不可。URL自動リンクも今回はなし。
- 表示順は新しい投稿順かつ新しいコメントが付いた順。
- 投稿したことがある人が掲示板を見に来たら「最近の訪問者」に表示。足跡機能。
- 管理者だけはユーザー登録が必要。管理者以外の登録も禁止はしない。

## 技術的な仕様
- Laravel9.x-10.x, PHP8.1-8.2。
- スターターキットはBreeze Blade stack
- データベースは使う。
- JavaScriptは使わない。Breezeで使われてるalpine.jsだけは仕方ないので残す。
- CSSはTailwind
- 表示順はupdated_at順。
- 認証・認可機能の説明用に管理者だけ登録。

## 対象読者
この環境構築ができている人。

- [Laravel 開発環境構築 Windows版](https://github.com/pop-culture-studio/articles/blob/main/laravel-installation-windows.md)
- [Laravel 開発環境構築 Mac版](https://github.com/pop-culture-studio/articles/blob/main/laravel-installation-mac.md)

Laravelの標準的な使い方を知りたい人。

「Laravel入門」であって「プログラミング入門」ではないのでプログラミング初心者は完全に対象外。Laravelの前にPHPや他の言語で基礎を覚えるのが先に必須。

## 新規プロジェクト作成
最初に
- WindowsならWSL側のホームに作業用のディレクトリ(`Sites`等なんでもいい)を作る。
- Macなら元からある`Sites`を使う。

この部分は自由なので別にどこでもいい。説明不足な記事だといきなりホームにLaravelプロジェクトを作ってしまうことがあるので作業場所を最初に決めておく。今後のプロジェクトも全部ここに保存。

ターミナルで作業場所に移動してプロジェクト作成コマンドを実行。  

```shell
cd ~/Sites/
curl -s "https://laravel.build/laravel-bbs" | bash
```
Dockerイメージのビルドで数分かかることもあるのでしばらく待つ。`Thank you!`の表示が出れば完了。
```shell
Thank you! We hope you build something incredible. Dive in with: cd laravel-bbs && ./vendor/bin/sail up
```

新しく作られたプロジェクトのディレクトリに移動。これ以降の作業場所は`~/Sites/laravel-bbs/`
```shell
cd laravel-bbs/
```
「Laravelが正常にインストールできていること」と「ローカルのPHPが正常に動くこと」の確認にLaravelのバージョン表示。
```shell
php artisan --version
```
laravel.buildで作成した場合は全部準備万端なのでそのままsail起動可能。
```shell
sail up -d
```
ブラウザで http://localhost/ が表示できれば新規プロジェクト作成は完了。

終了はstopかdown
```shell
sail down
```

「全部sail(Docker)から使う」なんて不便なことはしないのでローカルのPHPやnpmとsailは併用していく。

### ヒント1
作成方法はlaravel.build以外にも`composer create-project laravel/laravel example-app`や`laravel/installer`を使う方法がある。それぞれ微妙に違うけど基本的にはlaravel.buildを使えばいい。

### ヒント2
laravel.buildで実際に何をやっているかはここで確認。  
https://github.com/laravel/sail-server/blob/master/resources/scripts/php.sh

### ヒント3
「Dockerを知らなくてもSailは使える」なんてことはないのでDockerについても学習する。

## インストール直後のLaravelプロジェクトを確認
ここからはPhpStormでプロジェクトを開いて進める。

