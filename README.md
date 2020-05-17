# LaravelでAirbnb風アプリケーションを作る!

本リポジトリは、Laravelを使った勉強用のAirbnb風のアプリケーションのリポジトリです。

## 1. 環境セットアップ

環境のセットアップ方法について記載します。

## 1.1. Cloud9環境

### PHPのバージョン確認

```sh
$ php -v
```

### MySqlのバージョン確認

```sh
$ mysql --version
```

### パッケージのアップデート

```sh
sudo yum -y update
```

### PHP 7.3 インストール 

```sh
$ sudo yum -y install php73 php73-cli php73-common php73-devel php73-mysqlnd php73-pdo php73-xml php73-gd php73-intl php73-mbstring php73-mcrypt php73-zip
```

### PHPバージョンの切り替え 

```sh
$ sudo alternatives --set php /usr/bin/php-7.3
```

### MySQL停止

```sh
$ sudo service mysqld stop
```

### MySQLアンインストール

```sh
$ sudo yum -y erase mysql-config mysql55-server mysql55-libs mysql55
```

### MySQLインストール

```sh
$ sudo yum -y install mysql57-server mysql57
```

### MySQL起動

```sh
$ sudo service mysqld start
```

### MySQL起動(サーバー起動時)

```sh
sudo chkconfig mysqld on
```

### MySQLの初期設定

```sh
$ mysql_secure_installation
```

Yを入力
0を入力

rootパスワード設定
　**********
控えておくこと

あとはY(5回)


### MySqlログイン

```sh
$ mysql -u root -p
```

### データベース作成

```sql
CREATE DATABASE airbnb CHARACTER SET utf8mb4;
```

### データベース確認

```sql
show databases;
```
### データベース退去

```sql
exit
```
### gitリポジトリ先ファイルバックアップ

一時フォルダ作成

```sh
$ mkdir ../tmp
```

ディレクトリコピー

```sh
$ cp -r .c9/ ../tmp/.c9
```

削除

```sh
$ rm -rf .c9/
$ rm README.md
```

### クローン：gitリポジトリをダウンロード

airbnbstyle配下の資産をダウンロードする

```sh
$ git clone https://github.com/tomo1833/airbnbstyle/ .
```

### gitリポジトリ先ファイル戻し

```sh
$ cp -r ../tmp/.c9 .
```

### composer インストール

```sh
curl -sS https://getcomposer.org/installer | php
```
### composer ファイル移動

```sh
$ sudo mv composer.phar /usr/local/bin/composer
```

### composer アップデート

airbnbstyle配下で実施する。

```sh
$ cd airbnbstyle
```
### composer アップデート

```sh
$ composer update
```
version 確認

```sh
composer -V
```

### 環境ファイル更新

```sh
$ cp .env.example ./.env
```

### 環境ファイル更新

vi コマンドで書き換える

```sh
$ vi .env
```

```sh
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=airbnb
DB_USERNAME=root
DB_PASSWORD=password
```
DB_PASSWORDは、MySQLの初期設定で設定したパスワードを使う


### マイグレーション

マイグレーションをします。

```sh
$ php artisan migrate
```

### シーダー

シーダーを使ってデータを登録します。

```sh
$ php artisan db:seed --class=RoomsTableSeeder
```

### appキー作成

```sh
$ php artisan key:generate
```

### サーバーを起動する

```sh
$ php artisan serve --port=8080
```

### 画面確認
 
「Preview」の「Preview Running Application」を実行します。


## 1.2. windows環境（docker）

Dockerを使って環境を構築します。
DockerとDocker Composeのインストール方法は省略します。

### 設定ファイル作成

### docker-compose.yml

```yml
version: '3'
services:
    web:
        image: nginx:1.15.6
        ports:
            - "8000:80"
        depends_on:
            - app
        volumes:
            - ./default.conf:/etc/nginx/conf.d/default.conf
            - ./aribstyle/:/var/www/html
    app:
        build: .
        depends_on:
            - mysql
        volumes:
          - ./aribstyle/:/var/www/html

    mysql:
        image: mysql:5.7
        environment:
            MYSQL_DATABASE: airbnb
            MYSQL_USER: airbnb
            MYSQL_PASSWORD: password
            MYSQL_ROOT_PASSWORD: password
        ports:
            - "3306:3306"
        volumes:
            - ./data:/var/lib/mysql

```

### Dockerfile

```dockerfile
FROM php:7.2-fpm

# install composer
RUN cd /usr/bin && curl -s http://getcomposer.org/installer | php && ln -s /usr/bin/composer.phar /usr/bin/composer
RUN apt-get update \
    && apt-get install -y \
    git \
    zip \
    unzip \
    vim

RUN apt-get update \
    && apt-get install -y libpq-dev \
    && docker-php-ext-install pdo_mysql pdo_pgsql

WORKDIR /var/www/html
```

### default.conf

default.confを作成する。

```sh
server {
    listen 80;

    root  /var/www/html/airbnbstyle/public;
    index index.php index.html;

    access_log /var/log/nginx/access.log;
    error_log  /var/log/nginx/error.log;
    
    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }


    location ~ \.php$ {
          fastcgi_split_path_info ^(.+\.php)(/.+)$;
          fastcgi_pass   app:9000;
          fastcgi_index  index.php;
          include        fastcgi_params;
          fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
          fastcgi_param  PATH_INFO $fastcgi_path_info;
      }
}
```


### クローン：gitリポジトリをダウンロード

```sh
$ git clone https://github.com/tomo1833/airbnbstyle.git
```


## dockerコンテナの作成と起動

```sh
$ docker-compose up -d
```

docker-compose upは作成と起動を一緒に実施します。

-dはオプションで デーモン（バックグラウンドで動作する）です。


### composer更新

```sh
$ composer update
```

### .env修正

```sh
$ cp .env.example ./.env
```

```sh
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=airbnb
DB_USERNAME=root
DB_PASSWORD=password
```

### マイグレーション

マイグレーションをします。

```sh
$ php artisan migrate
```

### シーダー

```sh
$ php artisan db:seed --class=RoomsTableSeeder

```

### appキー作成
```sh
$ php artisan key:generate
```


## ブラウザ起動

http://localhost:8000 でアクセスして画面が表示されることを確認します。


## 1.3. Mac環境（docker）

Dockerを使って環境を構築します。
DockerとDocker Composeのインストール方法は省略します。

### 設定ファイル作成

### docker-compose.yml

```yml
version: '3'
services:
    web:
        image: nginx:1.15.6
        ports:
            - "8000:80"
        depends_on:
            - app
        volumes:
            - ./default.conf:/etc/nginx/conf.d/default.conf
            - ./aribstyle/:/var/www/html
    app:
        build: .
        depends_on:
            - mysql
        volumes:
          - ./aribstyle/:/var/www/html

    mysql:
        image: mysql:5.7
        environment:
            MYSQL_DATABASE: airbnb
            MYSQL_USER: airbnb
            MYSQL_PASSWORD: password
            MYSQL_ROOT_PASSWORD: password
        ports:
            - "3306:3306"
        volumes:
            - ./data:/var/lib/mysql

```

## 1.4. 対象バージョン

|アプリ|バージョン|
|:--|:--|
|PHP|7.X|
|MySQL|5.X|


## 2. ER図


![ER図](https://user-images.githubusercontent.com/61913268/81294929-8cd79480-90aa-11ea-9742-f917c26a7e46.png)

本アプリケーションで使用するデータベースのER図です。


## 3. 機能の概要

本アプリケーションは以下の画面があります。

* トップ画面
* 検索結果画面
* ルーム画面
* ログイン画面(Laravelが提供する機能)
* 会員（ユーザ）登録画面(Laravelが提供する機能)

### トップ画面

最初に表示される画面です。
ロケーション（場所）、チェックイン、チェックアウト（滞在期間）、人数で検索することができます。

![TOP画面](https://user-images.githubusercontent.com/61913268/81470422-a81adf00-9225-11ea-9480-7353622d7d02.gif)


### 検索結果画面

検索結果の画面です。
トップ画面で指定した条件に合致するルームを表示します。
また、Google mapを表示します。

![検索画面](https://user-images.githubusercontent.com/61913268/81470525-22e3fa00-9226-11ea-90b4-cad11e51f328.gif)

### ルーム画面

検索結果画面で指定したルームの画面です。
ルームの確認、予約状況の確認、予約の登録、口コミの登録が出来ます。

![ルーム画面](https://user-images.githubusercontent.com/61913268/81470575-62124b00-9226-11ea-8cbd-a72d57a4c535.gif)


### ログイン画面

会員のログイン画面です。
会員のログインが出来ます。
本アプリでは、Laravelが提供する機能を使ってログイン画面を表示します。

### 会員登録画面

会員の登録画面です。
会員の登録が出来ます。
本アプリでは、Laravelが提供する機能を使って会員の登録画面を表示します。


## 4. ライブラリに関して

本アプリで利用したライブラリを以下に記載します。

|ライブラリ|バージョン|備考|
|:--|:--|:--|
|composer|1.10.5|php用ライブラリ管理ツール|
|Google map api|X||
|bootstrap|4|cssフレームワークを採用しています。cdnを利用しています。|
|JQuery|X|cdnを利用しています。|

