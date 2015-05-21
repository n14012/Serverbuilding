
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

### 2-1 Vagrant を使用したCentOS 6.5のイメージを登録  

1.usbストレージからコピーしたCentOS65を次のコマンドでvagrantに登録。  
    vagrant box add CentOS65 centos65-x86_64-20140116.box --force  
#### vagrantの初期設定
1.作業用ディレクトリvagrant/ を作成し、初期設定をする。  
2.vagrant init コマンドで、Vagrantfileを作成。  
3.vi Vagrantfile コマンドでVagrantfileを開き、config.vm.box = "base"のbaseと書かれている部分を "CentOS65" に書き換える。  

#### 仮想マシンの起動
vagrant up コマンドで起動。  
#### 仮想マシンへ接続
vagrant ssh コマンドで、起動した仮想マシンへ接続。  
#### ホストオンリーアダプターの設定
vi Vagrantfile コマンドで開き、Vagrant.configure(2) do |config| と、config.vm.box = "CentOS65"の下に、config.vm.network :private_network, ip:"192.168.56.129"と追加。
#### Vagrantfileの反映
vagrant reload で再起動して設定を反映。  
### 2-2 Wordpressを動かす(2)
1.centos7と同じように、yum コマンドがproxyを通るように、/etc/yum.conf の下行に、proxy=http://172.16.40.1:8888を追加。
#### yum のproxy設定
2.sudo vi profile を実行してprofileの最終行に  
        PROXY='172.16.40.1:8888'  
        export http_proxy=$PROXY  
        export HTTP_PROXY=$PROXY  
        export HTTPS_PROXY=$PROXY  
        export https_proxy=$PROXY を追加する。
3.sudo yum update を実行。
#### nginx のインストール  
4.[Nginx公式サイト](http://nginx.org/en/linux_packages.html#stable)から、nginx-release-centos-6-0.el6.ngx.noarch.rpm をダウンロード。  
5.sudo rpm -i nginx-release-centos-6-0.el6.ngx.noarch.rpm で展開。  
6.sudo yum install nginx コマンドでnginxをインストール。  
#### MariaDB のインストール
7.cat << EOS | sudo tee /etc/yum.repos.d/MariaDB.repo > /dev/null とかいうどっかから拾ってきたコマンドで頑張って設定。しかしエラーを吐かれたので結局 sudo vi /etc/yum.repos.d/MariaDB.repo を自力で下記のように編集。  
    [mariadb]  
    name = MariaDB  
    baseurl = http://yum.mariadb.org/5.5/centos6-amd64  
    gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB  
    gpgcheck=1  
    EOS  
8.yum search maridb コマンドを実行。よくわからない。  
9.sudo yum -y install MariaDB-server MariaDB-client コマンドでmariadb をインストール。  
10.mysqlを起動して、CREATE DATABASE Wordpressコマンドでデータベースを作成。  
11.GRANT ALL ON Wordpress.* TO n14012@localhost IDENTIFIED BY 'password';でユーザを作成し、パスワードを設定する。  
#### php-fpm のインストール
12.sudo yum install php-fpm でphp-fpmをインストール。  
13.sudo vi php-fpm.d/www.conf の中のuser = apache と group = apache　のapache の部分を nginx に書き換える。  
14./etc/init.d/nginx start コマンドで起動。 sudo chkconfig nginx on で自動起動設定をする。  
#### wordpress の準備
15.wget がインストールしてなかったので、sudo yum install wget でインストール。  
16.wget が proxyを通るように、sudo vi /etc/wgetrc の中のコメントアウトされているプロキシ設定のコメントアウトを外し、172.16.40.1:8888を書き込む。  
17.wget https://ja.wordpress.org/wordpress-4.0-ja.tar.gz コマンドで、wordpress　をダウンロード。  
18.unzip wordpress-4.2.2-ja.zip で、wordpress を展開。展開して出てきた wordpress ディレクトリを/usr/share/nginx/html　の下に移動。  
#### worfpress の設定
19.wordpress ディレクトリの中の wp-config.sample.php を cp wp-config.sample.php wp-config.php コマンドで、名前を変えてコピー。  
20.sudo vi wp-config.php コマンドで下記の設定を書き換える。  
    define('DB_NAME', 'wordpress');  
    define('DB_USER', 'n14012');  
    define('DB_PASSWORD', 'hoge');  
#### nginx の設定
21.sudo vi /etc/nginx/conf.d/default.conf コマンドで、default.conf内のroot の部分を html; から /usr/share/nginx/html; に書き換え、/scriptの部分を$document_rootに書き換える。  
#### 起動
22.service iptables stop コマンドでファイアーウォールを停止。  
23.service nginx restart でnginxを再起動。  
24.service php-fpm start でphp-fpmを起動。  
25.service mysql start でMariaDBを起動。  
#### wordpress 確認
26. 192.168.56.129/wordpress/wp-admin/install.php にアクセス。必要事項を記入して登録し、wordpress をインストール。index.phpのページを表示できるか確認。  
### 2-3 wordpressを動かす(3)
#### apache のインストール
1.wget http://www.apache.org/dist/apr/apr-1.5.2.tar.gz コマンドで必要なファイルをダウンロードして準備する。  
2.tar zxvf httpd-2.2.29.tar.gz コマンドで展開。展開したディレクトリに移動し、./configure コマンドを実行。  
3.make コマンドでコンパイルして、 make install コマンドでインストール。  
4.wget http://www.apache.org/dist/apr/apr-util-1.5.4.tar.gz コマンドで同じように必要なファイルをダウンロードする。  
5.tar zxvf apr-util-1.5.4.tar.gz コマンドで展開。2.と同じ手順で進める。ただし、./configure コマンドに追加で以下のオプションを入れる。  
    ./configure --with-apr=/usr/local/apr  
6.wget http://www.apache.org/dist/httpd/httpd-2.2.29.tar.gz コマンドでいよいよapacheのインストールです。  
7.こっちも前のと同じ手順で進める。  
#### php-5.5.25のインストール
1.[PHP公式サイト](http://php.net/get/php-5.5.25.tar.gz/from/a/mirror) からphp-5.5.25.tar.gz をダウンロード。  
2.cd Downloads/php-5.5.25.tar.gz vagrant_2-3 コマンドで、Downloads ディレクトリから、vagrantの作業ディレクトリへphp-5.5.25を移動。  
3.CentOS65 側で、vagrant /vagrant/ ディレクトリへ移動して、移動してきたphp-5.5.25.tar.gzを tar zxvf php-5.5.25.tar.gzを展開。展開してできたディレクトリに移動して、./configure --apxs2=/usr/local/apache2/bin/apxs --with-mysql コマンドを実行。  
6.sudo yum install libxml2-devel で足りなかったものをダウンロード。  
7.make コマンドでコンパイルして、sudo make install コマンドでインストールする。  
