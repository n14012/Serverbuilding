
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
18.tar zxvf wordpress-4.2.2-ja.zip で、wordpress を展開。展開して出てきた wordpress ディレクトリを/usr/share/nginx/html　の下に移動。  
#### wordpress の設定
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
3.CentOS65 側で、vagrant /vagrant/ ディレクトリへ移動して、移動してきたphp-5.5.25.tar.gzを tar zxvf php-5.5.25.tar.gzを展開。展開してできたディレクトリに移動して、./configure --with-apxs2=/usr/local/apache2/bin/apxs --with-mysql コマンドを実行。  
6.sudo yum install libxml2-devel で足りなかったものをダウンロード。  
7.make コマンドでコンパイルして、sudo make install コマンドでインストールする。 
8.cp php-5.5.25/php.ini-development /usr/local/lib/php.ini コマンドでファイルをコピーする。  
9.http://php.net/mysql.default-socketを/　で検索して、その下にあるmysql.default_socket = の横に以下のように追記。  
   mysql.default_socket = /var/lib/mysql/mysql.sock  
#### mysql のインストール
1.sudo yum install mysql mysql-server コマンドでmysqlをインストール。  
2.mysqlを起動して、CREATE DATABASE Wordpressコマンドでデータベースを作成。  
3.GRANT ALL ON Wordpress.* TO n14012@localhost IDENTIFIED BY 'password';でユー>ザを作成し、パスワードを設定する。
#### wordpress の準備
1.2-2の手順と同様にインストールする。  
2.wordpress は /usr/share/nginx/html/　に置く。  

#### wordpress にインストール
192.168.56.129/wordpress/wp-admin/install.php　にアクセスして、インストール。
### 2-4 ベンチマークを取る
#### ab コマンドのインストール
sudo yum -y install apache2-util　コマンドでインストールする。
#### wordpress の高速化
1.[Wordpress.org](https://wordpress.org/plugins/wp-super-cache/)でwp-super-cacheプラグインをダウンロード。
2./usr/share/nginx/html/wordpress/wp-content/　のところに、mkdir uploadsコマンドで、ディレクトリを作り、chmod 777 uploads　コマンドで、サーバーに実行権限を与える。
3.sudo chown -R nginx:nginx /usr/share/nginx/html/　コマンドで、nginxにwordpressの権限を与える。
4.ダウンロードしたzipファイルを、wordpress のプラグインアップロード画面で読み込んで、インストール。
#### ab　コマンドの実行
プラグインを無効化にした状態で、 ab  -c 1000 -n 1000 http://192.168.56.129/wordpress コマンドで測定、その後有効化して測定して、結果を比較する。
#### プラグイン追加前 wordpress
Server Software:        nginx/1.0.15  
Server Hostname:        192.168.56.129  
Server Port:            80  
  
Document Path:          /wordpress  
Document Length:        185 bytes  
  
Concurrency Level:      1000  
Time taken for tests:   0.690 seconds  
Complete requests:      1000  
Failed requests:        0  
Non-2xx responses:      1000  
Total transferred:      387000 bytes  
HTML transferred:       185000 bytes  
Requests per second:    1448.87 [#/sec] (mean)  
Time per request:       690.191 [ms] (mean)  
Time per request:       0.690 [ms] (mean, across all concurrent requests)  
Transfer rate:          547.57 [Kbytes/sec] received  
  
Connection Times (ms)  
              min  mean[+/-sd] median   max  
Connect:        0   24  17.2     24      67  
Processing:     8  197 168.8    236     642  
Waiting:        8  196 169.6    236     641  
Total:         14  221 179.4    259     674  
  
Percentage of the requests served within a certain time (ms)  
  50%    259  
  66%    286  
  75%    303  
  80%    309  
  90%    346  
  95%    668  
  98%    672  
  99%    672  
 100%    674 (longest request)  
  
#### プラグイン追加後

Server Software:        nginx/1.0.15  
Server Hostname:        192.168.56.129  
Server Port:            80  
  
Document Path:          /wordpress  
Document Length:        185 bytes  
  
Concurrency Level:      1000  
Time taken for tests:   1.041 seconds  
Complete requests:      1000  
Failed requests:        0  
Non-2xx responses:      1000  
Total transferred:      387000 bytes  
HTML transferred:       185000 bytes  
Requests per second:    960.43 [#/sec] (mean)  
Time per request:       1041.195 [ms] (mean)  
Time per request:       1.041 [ms] (mean, across all concurrent requests)  
Transfer rate:          362.98 [Kbytes/sec] received  
  
Connection Times (ms)  
              min  mean[+/-sd] median   max  
Connect:        1   22   9.4     21      38  
Processing:    15  368 287.2    283     983  
Waiting:       15  366 288.0    281     982  
Total:         21  390 294.0    302    1021  
  
Percentage of the requests served within a certain time (ms)  
  50%    302  
  66%    501  
  75%    521  
  80%    525  
  90%    943  
  95%   1007  
  98%   1019  
  99%   1021  
 100%   1021 (longest request)  
   

ある程度の改善が見られた。  

## Section 3 Ansibleによる自動化とテスト  
  
### Section 3-0 Ansibleのインストール  
1.以下のコマンドを打ってAnsibleをインストール。  

    sudo apt-get install software-properties-common
    sudo apt-add-repository ppa:ansible/ansible
    sudo apt-get update
    sudo apt-get install ansible  
  
### Section 3-1 ansibleでwordpressを動かす(2)を行う  

1.ansibleをインストールしてplaybookが実行できるようになったので、playbookを書いていく。(playbook.yml参照)  

2.tempファイルを作り、その中に以下のファイルを事前に準備する。  
    default.con
    my.cn
    server.cn
    www.conf
### Section 3-1-2  VagrantfileからAnsibleを呼び出す  

1.下記をVagrantfileに
    config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"                                       
    end

### Section 5-1 bindのインストール


1.git clone http://github.com/cloneko/serverbuilding-2015/ コマンドで先生のファイルをclone  
2.vagrant plugin install vagrant-proxyconf
　vagrant plugin install vagrant-vbguest  コマンドでプラグインをインストールした後、何も考えずにvagrant up  

#### 5-2 bindの設定

