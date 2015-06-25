#Section 6 AWS

### 6-0 AWSコマンドラインインターフェースのインストール

1. 以下のコマンドで、awscliをダウンロード。
    sudo apt-get install python-pip  
    pip install awscli  

### 6-1 AWS EC2 + Ansible

1.  
    aws configure  

を実行して、先生にもらったaccessKeyとSecretaccessKeyを入力。  
  
2.ec2でインスタンスを作成。以下のコマンドを実行して、名前をつける。

    aws ec2 create-tags --resources i-e4dec417 --tags Key=Name,Value=n14012
  
3.ネットワークの設定を開いて、ゲートウェイを以下の値に編集。  

    172.16.40.10
  
4.3の設定で、外部にsshできるようになったので、以下のコマンドでsshを実行。
   
    ssh -i キーペアファイル名.pem ec2-user@パブリックIP
  
5.playbookファイルを開いて、以下の部分を書き換えてAnsibleがec2サーバーで動くように設定する。  

    mariadb →  mysql
    mariadb-server →  mysql-server
    MySQL-python →  MySQL-python27

    mariadb →  mysqld
  
    /home/nginx/latest.tar.gz →  /home/latest.tar.gz

6.以下のコマンドを実行して、playbookを動かしてansibleでwordpressをインストールして、パブリックIPをURLにぶち込んで確認する。  

    ansible-playbook -i hosts -u ec2 playbook.yml --private-key /home/n14012/Downloads/n14012.pem 
  

#### AMI(Amazon Machine Image)を作る  
  
1.前に作成したインスタンスのページを開き、右クリックでイメージを作成を選択。  
2.n14012_AMIという名前で作成し、保存。  
3.インスタンスを再度作成し、Amazon マシンイメージの選択で、マイAMIを選び、さっき作ったAMIを選択する。  
4.次の手順ボタンでステップ5：インスタンスのタグ付けまで飛ばす。  
5.値の入力欄に、好きな名前を入れる。(n14012)
6.次に進み、セキュリティグループの、ルールの追加でHTTPを選択して確認と作成を押す。  
7.作成したインスタンスのパブリックIPをコピーして、URLにぶち込んで、最初に作ったインスタンスのwordpressと同じページが表示されることを確認できればおk。  
### Section 6-2 AWS EC2(AMIMOTO)  

1.インスタンスを新たに作成する際にAWS Marketplace を選択し、検索欄にamimoto　と入力して検索する。  
2.PVM と HHVM が出てくるので、HHVM を選択。   
3.そのまま確認と作成を押して作成すると、エラーが出て作成できないので、セキュリティグループの編集から説明に書かれている日本語の説明文を消して、英語でわかりやすい説文に書き換える。  
4.すると無事に作成出来るので、パブリックIPをコピーして、下記のコマンドでターミナルからsshする。  

    ssh -i キーペア名.pem ec2-user@パブリックIP  

5.無事にsshでamimotoにログインが確認できたらおk。  
6.後はパブリックIPをURLにぶち込んでwordpressがインストールできたら終わり。  

### Section 6-3 Route53   

1.Route53のページを開き、Hosted Zone を開いて、Create Hosted Zone でDomain Name と Commentにわかりやすい名前で記入して Create する。  
2.5-1で作ったzoneファイルの中身をcatコマンドで開き、コピーする。  
3.Import Zone File を押して、コピーした中身を貼り付けてImport する。  
4.更新して書き換えられているのを確認して終わり。  

### Section 6-4 S3  

1.S3のページを開いてバケットを作成する。  
2.適当にうp用のhtmlファイルを作る。  
3.コマンドラインで以下のコマンドをうってうpる。  

    aws s3 cp 作ったhtmlファイル s3://バケット名/  
    
4.S3の自分のバケットのページで更新をかけて、うpったファイルが確認できればおk。  

### Section 6-5 CloudFront  

1.6-1で作ったAMIを選択し、インスタンスを起動。  
2.Cloud frontのページを開いて、Create Distribution!!!!!!!!!!  
3.delivery method は、webを選択する。  
4.さっきAMIで起動した、インスタンスのパブリックDNSをコピーして、orgin dmain name のところに貼り付けて、Create Distribution!!!!!!!!!!!!!  
5.Distribution一覧を開いて、statusがIn Progなんちゃらから、Deployedになるまで待つ。。。。ひたすら待つ。  
6.Deployedになったら、dmain name をコピーしてURLにぶっこんで、サイトが表示されるかを確認。  
7.abコマンドで速度を比べますが、プロキシでなんか負荷がかかるらしいので、sshしてそこで実行します。  

    ssh n14012@172.16.40.2  

8.コマンドで速度をはかりますます。  
    ab http://ec2-52-69-97-182.ap-northeast-1.compute.amazonaws.com/  

インスタンス ベンチ結果  

```
Server Software:        nginx/1.6.2
Server Hostname:        ec2-52-69-97-182.ap-northeast-1.compute.amazonaws.com
Server Port:            80

Document Path:          /
Document Length:        8902 bytes

Concurrency Level:      1
Time taken for tests:   0.213 seconds
Complete requests:      1
Failed requests:        0
Total transferred:      9109 bytes
HTML transferred:       8902 bytes
Requests per second:    4.70 [#/sec] (mean)
Time per request:       212.879 [ms] (mean)
Time per request:       212.879 [ms] (mean, across all concurrent requests)
Transfer rate:          41.79 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
              Connect:       34   34   0.0     34      34
              Processing:   179  179   0.0    179     179
              Waiting:      170  170   0.0    170     170
              Total:        213  213   0.0    213     213
```

    ab http://d1rv8hyj9d4tqt.cloudfront.net/  

cloud front ベンチ結果  

```
Server Software:        CloudFront
Server Hostname:        d1rv8hyj9d4tqt.cloudfront.net
Server Port:            80

Document Path:          /http://d1rv8hyj9d4tqt.cloudfront.net/
Document Length:        622 bytes

Concurrency Level:      1
Time taken for tests:   0.058 seconds
Complete requests:      1
Failed requests:        0
Non-2xx responses:      1
Total transferred:      944 bytes
HTML transferred:       622 bytes
Requests per second:    17.10 [#/sec] (mean)
Time per request:       58.478 [ms] (mean)
Time per request:       58.478 [ms] (mean, across all concurrent requests)
Transfer rate:          15.76 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
              Connect:       29   29   0.0     29      29
              Processing:    30   30   0.0     30      30
              Waiting:       29   29   0.0     29      29
              Total:         58   58   0.0     58      58
```
  

