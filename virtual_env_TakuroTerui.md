# 環境構築手順書
## バージョン一覧
- PHP ・・・**7.3**
- Nginx ・・・**1.19**
- MySQL ・・・**5.7**
- Laravel ・・・**6.0**
- OS ・・・**CentOS7**
<br />
<br />
## 環境構築手順  
### Vargrant  s
 - Vagrant boxdダウンロード  
```
vagrant box add centos/7
```
 - Vagrantの作業ディレクトリ準備  
Vagrantの作業用ディレクトリを作成し、作成したフォルダの中で以下のコマンドを実行します 。  
```
vagrant init centos/7
```  
 - Vagrantfileの編集  
```
# 変更点①
config.vm.network "forwarded_port", guest: 80, host: 8080

# 変更点②
config.vm.network "private_network", ip: "192.168.33.10" 

# 変更点③
config.vm.synced_folder "../data", "/vagrant_data"
# ↓ 以下に編集
config.vm.synced_folder "./", "/vagrant", type:"virtualbox"
```
 - Vagrant プラグインのインストール  
```
vagrant plugin install vagrant-vbguest
```
 - ゲストOSの起動・ログイン  
```
vagrant up
vagrant ssh
```
### パッケージのインストール  
```
sudo yum -y groupinstall "development tools"
```
### PHPのインストール
```
sudo yum -y install epel-release wget
sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
sudo rpm -Uvh remi-release-7.rpm
sudo yum -y install --enablerepo=remi-php73 php php-pdo php-mysqlnd php-mbstring php-xml php-fpm php-common php-devel php-mysql unzip
php -v
```
### composerのインストール
```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"

sudo mv composer.phar /usr/local/bin/composer
```
### laravelのインストール
```
composer create-project laravel/laravel --prefer-dist laravel_app 6.0
```
### データベースのインストール
```
sudo wget https://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
sudo rpm -Uvh mysql57-community-release-el7-7.noarch.rpm
sudo yum install -y mysql-community-server
mysql --version
sudo systemctl start mysqld
mysql -u root -p  
```
### データベースの作成
```
mysql > create database laravel_app;
```
### Nginxのインストール
viエディタを使用して以下のファイルを作成  
```
sudo vi /etc/yum.repos.d/nginx.repo
```
下記内容を書き込み
```
[nginx]
name=nginx repo
baseurl=https://nginx.org/packages/mainline/centos/\$releasever/\$basearch/
gpgcheck=0
enabled=1
```
書き終えたら保存して、以下のコマンドを実行しNginxのインストールを実行、起動
```
sudo yum install -y nginx
sudo systemctl start nginx
```
### Nginx設定ファイルの編集  
`default.conf`ファイルを編集
```
sudo vi /etc/nginx/conf.d/default.conf
```
```
server {
  listen       80;
  server_name  192.168.33.19; # Vagranfile記載のIPアドレス
  root /vagrant/laravel_app/public; # 追記
  index  index.html index.htm index.php; # 追記

  #charset koi8-r;
  #access_log  /var/log/nginx/host.access.log  main;

  location / {
      #root   /usr/share/nginx/html; # コメントアウト
      #index  index.html index.htm;  # コメントアウト
      try_files $uri $uri/ /index.php$is_args$args;  # 追記
  }

  # 省略

  # 該当箇所のコメントを解除し、必要な箇所には変更を加える
  # 下記は root を除いたlocation { } までのコメントが解除されていることを確認してください。

  location ~ \.php$ {
  #    root           html;
      fastcgi_pass   127.0.0.1:9000;
      fastcgi_index  index.php;
      fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name; 
      include        fastcgi_params;
  }
```
` php-fpm`ファイルを編集
```
;24行目近辺
user = apache
# ↓ 以下に編集
user = vagrant

group = apache
# ↓ 以下に編集
group = vagrant
```
Nginxは再起動
```
sudo systemctl restart nginx
sudo systemctl start php-fpm
```
下記コマンドを実行して nginx というユーザーでもログファイルへの書き込みができる権限を付与
```
$ cd /vagrant/laravel_app
$ sudo chmod -R 777 storage
```
### laravelの認証機能実装
laravel/uiライブラリをインストール
```
$ composer require laravel/ui 1.*
```
artisanコマンドを実行
```
$ php artisan ui vue --auth
```
node.js、npmをインストール
```
sudo yum install nodejs
sudo yum install npm
```
npmパッケージをインストール
```
$ npm install
$ npm run dev
```

## 環境構築の所感  
環境構築を行うにあたり感じたことを以下にまとめた。  
 - 簡易なアプリでもインストール、ダウンロードするものがたくさんある。  
 - ダウンロード、インストールするディレクトリによって予期せぬ動作が発生する可能性がある。
 - 設定ファイルの一行でも間違えてしまうと正常に動作しなくなるため、  
   慎重に編集する必要がある。
 - 使用しているライブラリ等のバージョンによって使用するコマンドが異なるため、  
   よく調べて実行する必要がある。
 - プログラミングにかかわらず、PCに関する基礎知識が不足しているため、  
   別途学習していきたい。
 - 環境構築の方法は多種多様であるため、幅広く知識を身につけていきたい。
## 参考サイト
### マークダウン関連
- [マークダウン確認方法](https://qiita.com/zoekaeru/items/084948b5c6479af1158e)

- [マークダウン記法](https://qiita.com/tbpgr/items/989c6badefff69377da7)

### 環境構築関連  
 - [laravelインストール方法](https://qiita.com/yukibe/items/5ee27163b603d7f68250)
 - [laravel6.0〜認証機能実装方法](https://qiita.com/ucan-lab/items/bd0d6f6449602072cb87)