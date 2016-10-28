# AnsibleからAzureを操作する

[AnsibleからAzureを操作してみた](http://techblog.clara.jp/2016/07/ansible_azure_arm01/)の写経。

* Vagrant上のUbuntu16.04に環境を構築します
* セットアップ手順をAnsibleでコード化
    * パスワードは自動生成されます
* Azureの無料試用版でも動作します
* 　WindowsではなくCentosを構築します

## 環境構築

Vagrant自体の構築は省略しますが使用したバージョンは以下です。

    % vagrant --version
    Vagrant 1.8.1

以下のコマンドでvagrantを起動します。

    $ vagrant up -d

起動したらsshでログインします。

    $ vagrant ssh

Azureにログインします。

ブラウザで「[https://aka.ms/devicelogin](https://aka.ms/devicelogin)」を開いてコードを入力します。

    $ azure login
    info:    Executing command login
    |info:    To sign in, use a web browser to open the page https://aka.ms/devicelogin. Enter the code {{ ここのコードを入力 }} to authenticate.

ansibleを実行してAzureへの接続をセットアップをします。

    $ cd ansible/azure-setup
    $ ansible-playbook site.yml

接続確認でSUCCESSなら成功！

    $ ansible localhost -m azure_rm_resourcegroup_facts
     [WARNING]: provided hosts list is empty, only localhost is available

    localhost | SUCCESS => {
        "ansible_facts": {
            "azure_resourcegroups": [
                {
                    "location": "southcentralus",
                    "name": "muutest",
                    "properties": {}
                },
                {
                    "location": "japanwest",
                    "name": "strage-test",
                    "properties": {}
                }
            ]
        },
        "changed": false
    }

登録したアプリはAzureポータルの Azure Active Directory > アプリ登録 で確認/削除が可能です。

## 仮想マシン作成

作業用のディレクトリに移動します。

    $ cd ../vm

tmpフォルダを作成してSSHの鍵を作成します。

「Enter file in which to save the key」の所だけ指定します。

    $ mkdir tmp
    $ ssh-keygen -t rsa
    Generating public/private rsa key pair.
    Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa): tmp/id_rsa
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:
    Your identification has been saved in tmp/id_rsa.
    Your public key has been saved in tmp/id_rsa.pub.
    The key fingerprint is:

以下でCentosの仮想マシンが作成されます。

    $ ansible-playbook site.yml

終了時にIPアドレスが表示されます。

    TASK [作成した仮想マシンのパブリックIPを表示] ****************************************************
    ok: [localhost] => {
        "msg": "Public IP Address: xxx.xxx.xxx.xxx"
    }

以下で作成した仮想マシンにsshログインが可能です。

    ssh MyAdminName@{{ 作成した仮想マシンのパブリックIP }} -i tmp/id_rsa

## 参考サイト

* [AnsibleからAzureを操作してみた その1 ~準備編~](http://techblog.clara.jp/2016/07/ansible_azure_arm01/)
* [AnsibleからAzureを操作してみた その2 ~仮想マシン作成編~](http://techblog.clara.jp/2016/07/ansible_azure_arm02/)
* [AnsibleからAzureを操作してみた その3 ~Dynamic Inventory & パッケージ・アップデート編~](http://techblog.clara.jp/2016/08/ansible_azure_arm03/)
