# 😛Ubuntu16.04におけるFlask開発環境の設定😛

## 1. 環境

今回は仮想環境は使用しません。
各環境のVersionは以下の通りとなっています。

* さくらVPS ubuntu16.04
* Apache2.4
* Python3.52
* Flask1.02

## 2.開発/実行に必要なパッケージのインストール

### 2-1. パッケージ追加
1. 「`sudo apt-get update`」実行。
2. 「`sudo apt-get upgrade`」実行。
3. Pythonモジュールの構築に必須なファイル、ライブラリが含まれたPython3.5追加「sudo apt-get install python3-dev」。
4. pip3のインストール「`sudo apt-get install python3-pip`」
5. Apache2インストール「`sudo apt-get install apache2 apache2-dev`」実行。
6. firewallインストール「`sudo apt-get install firewalld`」実行。
7. GCCインストール「`sudo apt-get install gcc`」実行。
8. XMLパーサインストール「`sudo apt-get install libexpat1-dev`」実行。

### 2-2. pipパッケージ追加
1. pip3のアップデート「`sudo pip3 install --upgrade pip`」実行。
2. Flask追加「`sudo pip3 install flask`」
3. 「`sudo pip3 install mod_wsgi`」実行。
4. 「`sudo pip3 install mod_wsgi-httpd`」実行。

※ mod_wsgi-httpdはインストールに時間がかかるので注意。

## 3. Applicationファイルと設定ファイルの作成

### 3-1. Flaskプロジェクトディレクトリの作成とapp.pyの作成
1. Apache2ホームディレクトリ（/var/www）直下ににflaskディレクトリを作成する。（/var/www/flask）
2. flaskディレクトリ直下に「app.py」を作成する。ファイルの中身は以下とする。
```

from flask import Flask
app = Flask(__name__)

@app.route("/")
def index():
    return "Hello, Worldだよ(*˘︶˘*).｡.:*♡"

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
LoadModule wsgi_module /usr/local/lib/python3.5/dist-packages/mod_wsgi/server/mod_wsgi-py35.cpython-35m-x86_64-linux-gnu.so

<VirtualHost *:80>
WSGIDaemonProcess snapmsg user=www-data group=www-data threads=5 python-home=/usr
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

* 1行目：LoadModuleのあとに「mod_wsgi-py35.cpython-35m-x86_64-linux-gnu.so」ファイルの場所を記載する。（PythonのVerによってディレクトリの名称が変わるの要確認）
* 6行目：「/」と「wsgi設定ファイル」の場所を記載する。 ※ 本例では、「/var/www/flask/adapter.wsgi」
* 9行目：「Flaskのプロジェクトディレクトリを指定」。

### 3-5.作成したflask.confファイルをApache2に読み込ませる
1. 「`sudo a2dissite flask.conf`」実行。
2. 「`sudo a2ensite flask.conf`」実行。

## 4.WEBサーバの実行とFirewallの設定

### 4-1. Apache2の起動
1. 「`systemctl start apache2`」実行。

### 4-2. Firewallの設定
1. http通信の通過設定「`firewall-cmd --add-service=http --zone=public --permanent`」実行。
2. https通信の通過設定「`firewall-cmd --add-service=https --zone=public --permanent`」実行。
3. Firewallの再起動「`systemctl restart firewalld`」実行。

### 4-3. 稼働のテスト
1. ブラウザで[http://xxx.xxx.xxx.xxx](http://xxx.xxx.xxx.xxx)（VPSのアドレス）にアクセスする。
2. Hello, Worldだよ(*˘︶˘*).｡.:*♡と表示されればOK！。

## 参考文献
* [Flask公式](http://flask.pocoo.org/docs/1.0/)
* [Flask公式 -mod_wsgiについて-](http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/)
* [ネコでもわかる！さくらのVPS講座 ～第三回「Apacheをインストールしよう」](https://knowledge.sakura.ad.jp/8541/)
