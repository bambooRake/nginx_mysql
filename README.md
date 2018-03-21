##目的
Codeigniterの導入テスト

実施者（私）のスキル
-業務でLaravelを使用している。
-業務でDockerを使用、自機にも構築経験あり。

##Dockerの設定内容
dockerの情報は散見しているので、githubで適当なリポジトリ探して来て、
pullした上で、設定ファイルの意味を舐めて行った方が早いと思う。
以下に、設定を晒す。

構成は、

```diff:構成は、
working_directory
└db
　└mysql_data
　└Dockerfile
└web
　└autorun.sh
　└Dockerfile
└docker-compose.yml
```

```dockerfile:db/Dokerfile
FROM mysql:5.6
```

nginxの導入は、yum -y install php71u-fpm-nginx を
追加すればほぼ終わり。設定ファイルに関しても、
apacheより馴染み易い気がしている。

```dockerfile:web/Dokerfile
FROM centos:centos6

RUN yum -y install https://dl.iuscommunity.org/pub/ius/archive/CentOS/6/x86_64/ius-release-1.0-14.ius.centos6.noarch.rpm \
    && yum -y update \
    && yum -y install php71u-cli \
    && yum -y install php71u-pecl-xdebug \
    && yum -y install php71u-json \
    && yum -y install php71u-xml \
    && yum -y install php71u-mbstring \
    && yum -y install php71u-pdo \
    && yum -y install php71u-fpm-nginx \
    && yum -y install php71u-mysqlnd \
    && yum -y install mysql56u
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
    && php -r "if (hash_file('SHA384', 'composer-setup.php') === '544e09ee996cdf60ece3804abc52599c22b1f40f4323403c44d44fdfdd586475ca9813a858088ffbc1f233e9b180f061') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" \
    && php composer-setup.php \
    && php -r "unlink('composer-setup.php');" \
    && mv composer.phar /usr/local/bin/composer \
    && echo 'xdebug.remote_port=9004' >> /etc/php.ini \
    && echo 'xdebug.remote_enable=1' >> /etc/php.ini \
    && echo 'xdebug.remote_autostart=1' >> /etc/php.ini \
    && echo 'xdebug.remote_log=/tmp/xdebug.log' >> /etc/php.ini \
    && echo 'xdebug.remote_host=docker.for.win.localhost' >> /etc/php.ini 
COPY ./autorun.sh /usr/etc
ENTRYPOINT /usr/etc/autorun.sh
```

apacheを使うときは、apacheだけをENTRYPOINTで起動させていたが、
nginxとphp-fpmを起動させないといけないので、複数起動用のスクリプトファイルを用意して、実行させる。
下記を参考にさせてもらった。ありがたや。
[Dockerで複数デーモンを起動する手法をまとめてみる](https://www.hilotech.jp/blog/it/290)

```sh:/web/autorun.sh
#!/bin/bash
nginx
php-fpm


while true
do
    sleep 10
done
```

```docker:docker-compose.yml
version: '2'
services:
  db:
    container_name: ndb
    build: ./db
    ports:
      - '3306:3306'
    environment:
      MYSQL_ROOT_PASSWORD: 'password'

    volumes:
      - ./db/mysql_data:/var/lib/mysql
  web:
    container_name: nweb
    build: ./web
    ports:
      - '8080:80'
    depends_on:
      - db
    tty: true
```

##ビルド・実行

```bash:ビルド・実行方法
docker-compose build
docker-compose up -d

シェルへの接続は↓
docker exec -ti nweb /bin/bash
```

##動作確認

```bash:ブラウザで
localhost:8080
```
に接続して、nginxの画面が表示されればサーバはOK。
phpに関しては、

```php:index.php
<?php phpinfo(); ?>
```
などを用意して確認。phpファイルは動作する状態になってるので
確認は飛ばしてもOK。

##Codeigniterの導入
[[初心者向け] Codeigniter3をComposerを使ってインストールして動かすまで](https://rdlabo.jp/codeigniter-302.php)
を参考にさせて頂く。

```bash:dockerのシェル上で作業を行う
↓でdocker上のシェルに接続
docker exec -ti nweb /bin/bash
```

```bash:/usr/share/nginx/html内にcomposer.jsonを作成
{
    "require": {
            "codeigniter/framework": "dev-master"
    }
}
```

```bash:以下コマンドを実行
composer install
```

```bash:フォルダ・ファイルをコピー
cp -r vendor/codeigniter/framework/application/ application
cp vendor/codeigniter/framework/index.php index.php
```

```bash:index.phpを下記に修正
$system_path = './vendor/codeigniter/framework/system';
$application_folder = './application';
```

```bash:./application/config/config.phpを下記に修正
$config['composer_autoload'] = realpath(APPPATH . "./vendor/autoload.php");
```

下記が表示されれば導入成功！
![localhost_8080_.png](https://qiita-image-store.s3.amazonaws.com/0/176756/5b96c5fc-b215-746e-7ade-33f6e84a47ae.png)

##あとがき
ここまでご覧いただき、ありがとうございます。
実際に作業した方はお疲れさまでした！

dockerを使うことで、工程の後戻りも気楽にできるのでドキュメントに起こすときには重宝しますね。
次回はCodeigniterのチュートリアルを試してみて、要点をまとめられればと思います。

