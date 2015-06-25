# Section 3 Ansibleによる自動化とテスト  
  
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
    my.cnf
    server.cnf
    www.conf
### Section 3-1-2  VagrantfileからAnsibleを呼び出す  

1.下記をVagrantfileに追記する。  

    config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"                                       
    end
2.何も考えずに  

    vagrant up  
3.無事にwordpressが立ち上がっていることが確認できたら終わり。   

