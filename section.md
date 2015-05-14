
# Section0
### VirtualBox のインストール
1.[VirtualBox公式サイト](https://www.virtualbox.org/wiki/Linux_Downloads)に移動してダウンロード
2.dpkg コマンドでインストール
3.何かわけの分からないエラー文が出て、そこにlibsdl1.2debianがまだインストールされていませんと書いてあったのでとりあえずインストール

### Vagrant のインストール

1.[Vagrant公式サイト](http://www.vagrantup.com/downloads)からLINUXの64bitをダウンロード
2.vagrantをdpkgコマンドでインストール

	
# Section1
### CentOSのインストール
1.CentOS公式サイトからCentOSをダウンロード。
2.virtualboxを起動して、新規のところからメモリサイズ1GB、ストレージを8GBに設定してCentOSをインストール。インストール中に管理者ユーザーを作成。
3.ネットワークアダプタ２を設定したつもりが設定できていなかったことに気づいてホストオンリーアダプターで設定。

### ネットワークアダプター1/2へのIPアドレスの設定とssh接続の確認

1.とりあえずCentOSを起動し、cd　/etc/sysconfig/network-scriptに移動。ifcfg-enp0s3ファイルと、RedHatマニュアルを見ながら色々書き換えた。結局ONBOOT=yesにしただけ
2.書き換えたけどうまくいかなかったのでいささんに助けを求めてifcfg-enp0s8我あることを知り、0s3の中身をそのままcpして0s8を作成。
3.ip a コマンドを使ってIPアドレスを確認してsshで接続確認。

#### インストール後の設定
1.cd /etc/にいって、sudo vi yum.confを編集。下行にproxy=http://172.16.40.1:8888を追加。
2.sudo vi profile を実行してprofileの最終行に
	PROXY='172.16.40.1:8888'
	export http_proxy=$PROXY
	export HTTP_PROXY=$PROXY
	export HTTPS_PROXY=$PROXY
	export https_proxy=$PROXY を追加する。

#### アップデート

sudo yum update を実行。

### Wordpressを動かす

1.sudo yum list httpd でhttpdを確認して、sudo yum install httpd.x86_64を実行しApacheをインストール。
2.yum -y install http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm コマンドで、公式リポジトリファイルをインストール。
3.yum -y install mysql
　yum -y install mysql-devel
　yum -y install mysql-server
　yum -y install mysql-utilitiesコマンドを実行してmysqlをインストール。
4.yum -y install php-mysql php php-gd php-mbstringコマンドを実行して、phpをインストール。

5.mysqlを起動して、CREATE DATABASE Wordpressコマンドでデータベースを作成。
6.続いて、CREATE USER n14012@192.168.56.101 IDENTIFIED by 'hoge'コマンドでユーザーを作成。
7.wget http://ja.wordpress.org/wordpress-3.5.1-ja.tar.gzコマンドでWordpressをダウンロード。
8.tar zxvf wordpress-3.5.1-ja.tar.gz で解凍。
9.mv wordpress /var/www/html コマンドでファイルを移動。
10.cp wp-config-sample.php wp-config.phpコマンドで名前を変えてコピー。
11.vi wp-config.php の中のデータベース名、ユーザー名、パスワードを設定したものに変更。:wqで保存。
12.GRANT ALL ON Wordpress.* TO n14012@localhost IDENTIFIED BY 'password';で設定する。
13.vi /etc/sysconfig/selinux のSELINIXをdisabledに変更。
14.systemctl start httpdコマンドでapacheを起動。
15.systemctl stop firewalldコマンドでファイアーウォールを停止。
16.http://192.168.56.101/wordpress/wp-admin/install.phpにアクセスして確認。
