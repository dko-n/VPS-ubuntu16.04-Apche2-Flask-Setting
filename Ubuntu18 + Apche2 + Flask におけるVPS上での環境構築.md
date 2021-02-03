# 😛Ubuntu18上でのFlask環境の設定😛

## 1. 環境

今回は仮想環境は使用しません。
各環境のVersionは以下の通りとなっています。

* AWS LightSail Ubuntu18 
* Apache2.4
* Python3.6x
* Flask1.1.1

## 2.開発/実行に必要なパッケージのインストール

### 2-1. パッケージ追加
1. サーバのパッケージ情報を最新にする。`sudo apt-get update`コマンドをターミナル上で実行。
2. サーバのパッケージを最新に更新する。`sudo apt-get upgrade`コマンドをターミナル上で実行。
3. Pythonモジュールの構築に必須なファイル、ライブラリが含まれたPython3.6を追加する`sudo apt-get install python3-dev`コマンドをターミナル上で実行。
4. pip3をインストールする。`sudo apt-get install python3-pip`コマンドをターミナル上で実行。
5. Apache2インストールする。`sudo apt-get install apache2 apache2-dev`コマンドをターミナル上で実行。
6. firewallインストールする。`sudo apt-get install firewalld`コマンドをターミナル上で実行。
7. GCCをインストールする。`sudo apt-get install gcc`コマンドをターミナル上で実行。
8. XMLパーサをインストールする。`sudo apt-get install libexpat1-dev`コマンドをターミナル上で実行。

### 2-2. pipパッケージ追加
1. pip3自体をアップデートする。`sudo pip3 install --upgrade pip`コマンドをターミナル上で実行。
2. Flask追加`sudo pip3 install flask -t /usr/local/lib/python3.6/dist-packages`コマンドをターミナル上で実行。
3. mod_wsgi追加。`sudo pip3 install mod_wsgi -t /usr/local/lib/python3.6/dist-packages`コマンドをターミナル上で実行。
4. mod_wsgi_httpd追加。`sudo pip3 install mod_wsgi-httpd -t /usr/local/lib/python3.6/dist-packages`コマンドをターミナル上で実行。

* **WSGIが実行するPythonインタプリタが参照するパッケージのインストールディレクトリに追加すること。**  
例えば、`sudo pip3 install flask -t /usr/local/lib/python3.6/dist-packages`のようにインストール先を指定することができる。
* 基本的には「/usr/local/lib/python3.6/dist-packages」にinstallするのが吉。
* mod_wsgi-httpdはインストールに**時間がかかる**ので注意。
* PJにrequirements.txtファイル（PJ固有のpipライブラリ）が含まれている場合は、`sudo pip3 install -r requirements.txt -t /usr/local/lib/python3.6/dist-packages`コマンドをターミナル上で実行。

## 3. Applicationファイルと設定ファイルの作成

### 3-1. Flaskプロジェクトディレクトリの作成とapp.pyの作成
1. Apache2ホームディレクトリ（/var/www）直下ににflaskディレクトリを作成する。（/var/www/flask）
2. 作成したディレクトリのパーミッションを変更する。/var/wwwディレクトリ上で`sudo chmod 777 flask`コマンドをターミナル上で実行。
3. flaskディレクトリ直下に「app.py」を作成する。ファイルの中身は以下とする。
```

from flask import Flask
app = Flask(__name__)

@app.route("/")
def index():
    return "Flask Start!!!"

if __name__ == "__main__":
    app.debug = True
    app.run(host='0.0.0.0', port=80)

```

### 3-2. WSGIとの連携ファイルの作成
1. wsgiとのアダプターファイル「adapter.wsgi」をflaskディレクトリ直下に作成する。ファイルの中身は以下とする。
```

# coding: utf-8
import sys
sys.path.insert(0, "/var/www/flask")
from app import app as application

```

### 3-3.Apache2の設定ファイル（apache2.conf）への追記
1. ディレクトリ「/etc/apache2」直下のapache2.conf内に以下の内容を**追記**する。

```
<Directory /var/www/flask/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>

```

### 3-4. Apache2の設定ファイルの作成
1. ディレクトリ「/etc/apache2/sites-available」直下に「flask.conf」を作成。ファイルの中身は以下とする。

```
LoadModule wsgi_module /usr/local/lib/python3.6/dist-packages/mod_wsgi/server/mod_wsgi-py36.cpython-36m-x86_64-linux-gnu.so

<VirtualHost *:80>
    WSGIDaemonProcess flask user=www-data group=www-data threads=5
    WSGIScriptAlias / /var/www/flask/adapter.wsgi
    WSGIScriptReloading On

    <Directory “/var/www/flask“>
        WSGIProcessGroup flask
        WSGIApplicationGroup %{GLOBAL}
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

* 1行目：LoadModuleのあとに「mod_wsgi-py36.cpython-36m-x86_64-linux-gnu.so」ファイルの場所を記載する。（PythonのVerによってディレクトリの名称が変わるの要確認）
* 5行目：「/」と「wsgi設定ファイル」の場所を記載する。 ※ 本例では、「/var/www/flask/adapter.wsgi」
* 8行目：「Flaskのプロジェクトディレクトリを指定」。

### 3-5.作成したflask.confファイルをApache2に読み込ませる
1. /etc/apache2/sites-availableディレクトリ上で`sudo a2dissite 000-default.conf`コマンドをターミナル上で実行。（**デフォルトのconfファイルの無効化**）
2. /etc/apache2/sites-availableディレクトリ上で`sudo a2dissite flask.conf`コマンドをターミナル上で実行。
3. /etc/apache2/sites-availableディレクトリ上で`sudo a2ensite flask.conf`コマンドをターミナル上で実行。

## 4.WEBサーバの実行とFirewallの設定

### 4-1. Apache2の起動
1. `systemctl start apache2`コマンドをターミナル上で実行。

### 4-2. Firewallの設定
1. http通信の通過設定を行う。`firewall-cmd --add-service=http --zone=public --permanent`コマンドをターミナル上で実行。
2. https通信の通過設定を行う。`firewall-cmd --add-service=https --zone=public --permanent`コマンドをターミナル上で実行。
3. Firewallの再起動を行う。`systemctl restart firewalld`コマンドをターミナル上で実行。

### 4-3. 稼働のテスト
1. ブラウザで[http://xxx.xxx.xxx.xxx](http://xxx.xxx.xxx.xxx)（VPSのアドレス）にアクセスする。
2. Flask Start!!!と表示されればOK！。

## 参考文献
* [Flask公式](http://flask.pocoo.org/docs/1.0/)
* [Flask公式 -mod_wsgiについて-](http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/)
* [ネコでもわかる！さくらのVPS講座 ～第三回「Apacheをインストールしよう」](https://knowledge.sakura.ad.jp/8541/)
