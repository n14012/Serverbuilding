# section5  
  
### Section 5-1 bindのインストール


1.git clone http://github.com/cloneko/serverbuilding-2015/ コマンドで先生のファイルをclone  
2.
    vagrant plugin install vagrant-proxyconf  
    vagrant plugin install vagrant-vbguest  

コマンドでプラグインをインストールした後、何も考えずにvagrant up  

#### 5-2 bindの設定  
  
1. zoneファイルを編集。先生の用意してたclonekoの部分を自分の学籍番号に書き換える。  
        IN SOA ns.n14012.com. ewe.n14012.com.  
             IN      NS      ns.n14012.com.  
             IN      MX      10      aspmx.l.google.com.  
             IN      NS      ns2.n14012.com.  
     www     IN      A       172.16.40.70  
     ns      IN      A       172.16.40.70  
     ns2     IN      A       172.16.40.71  

  
2. slave-named.confファイルを編集。以下の部分をzoneファイルに書いた学籍番号に書き換える。  

    zone "n14012.com" {
            type slave;
            masters { 192.168.33.14; };   
            file "slave/zone.cloneko.com";
    };
  
3. master-named.confファイルを編集。以下の部分を同じように書き換える。  

    zone "n14012.com" {
             type master;
             file "zone.cloneko.com";
    };
  
4. vagrant up で slave と master の２つの仮想マシンが立ち上がってるはずなので、両方にsshして、digコマンドでDNSに問い合わせる。  

   dig @192.168.56.14 zoneファイル名  

5. digの結果が返ってこればおｋ  

  結果(やっと返事をくれたから正しいとか正しくないとかもういいです)↓  

; <<>> DiG 9.9.4-RedHat-9.9.4-18.el7_1.1 <<>> @192.168.56.14 ns.n14012.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 57596
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 2
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;ns.n14012.com.     IN  A

;; ANSWER SECTION:
ns.n14012.com.    60  IN  A 172.16.40.70

;; AUTHORITY SECTION:
n14012.com.   60  IN  NS  ns2.n14012.com.
n14012.com.   60  IN  NS  ns.n14012.com.

;; ADDITIONAL SECTION:
ns2.n14012.com.   60  IN  A 172.16.40.71

;; Query time: 0 msec
;; SERVER: 192.168.56.14#53(192.168.56.14)
;; WHEN: Thu Jun 25 05:54:28 UTC 2015
;; MSG SIZE  rcvd: 106

