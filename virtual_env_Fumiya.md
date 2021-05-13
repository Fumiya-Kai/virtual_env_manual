# 環境構築手順書

## 使用環境
|項目|環境|
|:----:|:----:|
|仮想環境|Vagrant|
|OS|CentOS|
|DB|MySQL5.7|
|Webサーバ|Nginx|
|PHP|PHP7.3|
|フレームワーク|Laravel6.0|


## 環境構築手順  
1. ディレクトリ作成  
2. Vagrantfileの編集
3. 必要なもののインストール    
	1. プラグインのインストール  
	2. 開発ツールを一括インストール  
	3. phpのインストール  
	4. composerのインストール  
	5. mysqlのインストール  
	6. laravelのインストール  
	7. nginxのインストール  
4. 設定  
	1. nginxの設定  
	2. php-fpmの設定  
5. 作動  

## 1.ディレクトリ作成  
mkdirコマンドで作業ディレクトリを作成する。今回はvagrant_dirという名前で作成。  
```
mkdir vagrant_dir  
```
作成したディレクトリ（この例ではvagrant_dir）内で以下のコマンドを実行  
```
vagrant init centos/7  
```
成功すると以下の文章が出る  
>A \`Vagrantfile\` has been placed in this directory. You are now  
>ready to \`vagrant up\` your first virtual environment! Please read  
>the comments in the Vagrantfile as well as documentation on  
>\`vagrantup.com\` for more information on using Vagrant.  
***  
`vagrant init box名`  
vagrantを初期化するコマンド。  
これによりVagrantfileというファイルが作業ディレクトリ内に作られる。  
box名を指定することで、Vagrantfileにbox名が設定された状態で出力される。  
***  
  
## 2.Vagarantfileの編集  
次にVagrantfileの編集をして、ポート番号やipアドレスの設定や共有フォルダの同期の設定を行う。  
```
config.vm.network "forwarded_port", guest: 80, host: 8080  
の#をはずす
config.vm.network "private_network", ip: "192.168.33.10"  
の#をはずし、192.168.33.10を192.168.33.19に変更  
config.vm.synced_folder "../data", "/vagrant_data"  
↓以下に変更
config.vm.synced_folder "./", "/vagrant", type:"virtualbox"  
```
***  
`config.vm.network "forwarded_port", guest: 80, host: 8080`  
これはホストOSへの通信をゲストOSに転送するときのポート番号の設定  
`config.vm.network "private_network", ip: "192.168.33.10"`  
これは構築するサーバーのプライベートネットワーク上のipアドレスの設定。今回の場合、192.168.33.19  
`config.vm.synced_folder "./", "/vagrant", type:"virtualbox"`  
ホストOSとゲストOSで同期するフォルダの設定。今回の場合、ホストOS側の作業ディレクトリとゲストOS側の/vagrantを同期。第3引数以降はオプションで、typeは同期フォルダタイプ。
これにより、virtualboxの機能でフォルダを同期できる。  
***  
## 3.必要なものをインストール  
次に必要なものをインストールする。  
### 1.プラグインのインストール  
以下のコマンドでプラグインをインストールできる。（今回必要なのはvagrant-vbguest、vagrant-vbguestはホストOSとゲストOSでGuestAdditionsのバージョンを合わせ、様々な機能がホストとゲストで統合的に扱えるようになる）  
vagrant plugin listでインストール完了か確認できる。  
``` 
vagrant plugin install vagrant-vbguest  
vagrant plugin list  
``` 
vagrantを起動する。  
```  
vagrant up  
``` 
vagrant ssh-configの情報に基づいてvagrantにログイン。  
```  
vagrant ssh  
```  
  
### 2.gitなどの開発に必要なツールを一括でインストール  
以下のコマンドで様々なツールを一括でインストールする。
```  
sudo yum -y groupinstall "development tools"  
```  
***  
`groupinstall`は複数のパッケージを一括でインストールする。  
`sudo`はrootユーザーの権限を借りるコマンド。許可がなくて実行できない時はこのコマンドで実行すると実行できる。  
`yum`はcentOSのパッケージ管理ツール -yをつけるとコマンド実行時の質問に自動でyesと答える。  
***  
  
### 3.phpをインストール  
以下のコマンドを順に実行してphpをインストールする。  
```  
sudo yum -y install epel-release wget  
sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm  
sudo rpm -Uvh remi-release-7.rpm  
sudo yum -y install --enablerepo=remi-php73 php php-pdo php-mysqlnd php-mbstring php-xml php-fpm php-common php-devel php-mysql unzip  
php -v  
```  
phpのバージョンが確認できたらインストール完了  
***  
`wget`は指定したURLのファイルをダウンロードする。
`rpm`は`yum`と同じくcentOSのパッケージ管理ツールだが、依存関係の解決、自動更新などの機能はrpmにはない。  
4つ目のコマンドで、php7.3と拡張モジュールをインストールしている。  
***  
  
### 4.composerのインストール  
以下のコマンドを順に実行しcomposerをインストールする。  
```  
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"  
php composer-setup.php  
php -r "unlink('composer-setup.php');"  
どのディレクトリにいてもcomposerコマンドを使用できるようfileを移動  
sudo mv composer.phar /usr/local/bin/composer  
composer -v 　
```  
composerのバージョンが確認できたらインストール完了  
***  
composerはphpのパッケージ管理ツール  
***  
  
### 5.mysqlのインストールとlaravelアプリのデータベース作成  
以下のコマンドを順に実行しmysqlをインストールする。  
```  
sudo wget https://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm  
sudo rpm -Uvh mysql57-community-release-el7-7.noarch.rpm  
sudo yum install -y mysql-community-server  
mysql --version  
```  
mysqlのバージョンが確認できたらインストール完了  

次にmysqlに入り、laravelアプリのデータベースを作成する。
```  
sudo cat /var/log/mysqld.log | grep 'temporary password' #パスワードを確認、以下の出力ならhogehoge

出力:2017-01-01T00:00:00.000000Z 1 [Note] A temporary password is generated for root@localhost: hogehoge


mysql -u root -p                              #mysqlにログイン
set password = "新しいパスワード"             #パスワードの設定
create database laravel_app;                  #データベースの作成　今回はlaravel_appという名前で作成
```  
  
### 6.laravelのインストールとログイン機能の実装  
ホストOSでvagrantの作業ディレクトリに移動し、以下を実行（今回はlaravel_appがプロジェクト名）  
```  
composer create-project laravel/laravel --prefer-dist laravel_app 6.0  
```  
以下でログイン機能も実装できる。
```  
composer require laravel/ui "^1.0" --dev
php artisan ui vue --auth
```  
laravel_app内の.envファイルを編集してデータベースの設定を以下のようにする。
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_app #自分のlaravelアプリ用データベース名
DB_USERNAME=root        #変更
DB_PASSWORD=password    #自分のmysqlパスワード
```  
次に、マイグレーションを実行する。  
```  
php artisan migrate
```
これで必要なデータベースが作成され、laravelとmysqlが連携できるようになり、ログイン機能も実装された。  
  
### 7.nginxのインストール  
ゲストOSに戻り、以下のファイルを作成  
```  
sudo vi /etc/yum.repos.d/nginx.repo  
```  
以下の内容を書き込む  
```  
[nginx]  
name=nginx repo  
baseurl=https://nginx.org/packages/mainline/centos/\$releasever/\$basearch/  
gpgcheck=0  
enabled=1  
```  
その後、インストール  
```  
sudo yum install -y nginx  
```  
  
## 3.設定  
### laravelを動かすための設定  
```  
sudo vi /etc/nginx/conf.d/default.conf   
```  
- server_nameをVagrantfileに設定したipに変更  
- rootとindexを追記  
- location / {}の中身を変更  
- location ~\.php${}の変更
```  
server {
    listen       80;
    server_name  192.168.33.19; #Vagrantfileのipに設定したアドレスに設定。
    root /vagrant/laravel_app/public; #追記
    index  index.html index.htm index.php; #追記

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
       #root   /usr/share/nginx/html; #コメントアウト
       #index  index.html index.htm; #コメントアウト
       try_files $uri $uri/ /index.php$is_args$args; #追記
    }

    ###省略###

    location ~ \.php$ {
    #    root           html;
       fastcgi_pass   127.0.0.1:9000; #www.confのlistenと同じにする
       fastcgi_index  index.php;
       fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name;  # $fastcgi_script_name以前を /$document_root/に変更
       include        fastcgi_params;
    }  
```  
  
### php-fpmの設定  
```  
sudo vi /etc/php-fpm.d/www.conf
```  
以下の変更。  
vagrantの部分は、ls -laで確認したディレクトリやファイルの権限者と同じにする  
```  
user = apache
# ↓ 以下に編集
user = vagrant

group = apache
# ↓ 以下に編集
group = vagrant
```  
  
## 4. 作動  
http://192.168.33.19 にアクセスするとlaravelのwelcome画面にアクセスできる。ログインを実行できれば環境構築完了  

## 気づいたこと、学んだこと  
設定ファイルの意味がわかればエラーの対処もしやすいのではないかと思いました。  
もともとphp-fpmの設定でuser=nginx、group=nginxとしていましたが、セッションが変わるたびに権限のエラーが起こり、該当のファイルの権限を調べるとvagrantとなっており
nginxに変更ができなかったので設定ファイルの方を変更したらうまくいったので今回はvagrantとしてあります。設定ファイルのuser,groupの部分の意味が分かったから気づいたことなので、
他の箇所もわかっていれば様々なエラーに対応しやすいと思いました。  
## 参考サイト  
[giztech](https://giztech.gizumo-inc.work/lesson/18)  
[nginxで、connect() failed (111: Connection refused) while connecting to upstreamってエラーにハマった。 - Qiita](https://qiita.com/mindlessdoll/items/9dd035a53e4491a8ef9b)  
[Laravel4、app/storageのパーミッショントラブル](https://kore1server.com/261)  
[Vagrantの使い方 - Qiita](https://qiita.com/IK12_info/items/28ac74bfa92a39e886ef)  
[【Vagrant】vagrantを導入しよう - Qiita](https://qiita.com/ohuron/items/057b74a42b182b200ae6)  
[Vagrantfileの基本的な書き方｜FKeisuke｜note](https://note.com/fkeisuke/n/n9259600151c1)  
