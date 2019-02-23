# Ubuntu16.04におけるFlask開発環境の設定

## 1. 環境

今回は仮想環境は使用しません。
各環境のVerは以下の通りとなっています。


* さくらVPS Ubuntu 16.04
* Apache2.4
* Python3.52
* Flask1.02


## 2.開発/実行に必要なパッケージのインストール

### 2-1. パッケージ追加
1. 「sudo apt-get update」
2. 「sudo apt-get upgrade」
3. Pythonモジュールの構築に必須なファイル、ライブラリが含まれたPython3.6追加「sudo apt-get install python3-dev」。
4. pip3のインストール「sudo apt-get install python3-pip」
5. Apache2追加「sudo aot-get install apache2」実行。
5. Apache2追加「sudo aot-get install apache2-dev」実行。
5. 「sudo apt-get install firewalld」実行。
6. GCC追加「sudo apt-get install gcc」実行。
7. XMLパーサ追加「sudo apt-get install libexpat1-dev」実行。

### 2-2. pipパッケージ追加
1. 「pip3 install --upgrade pip」実行。
2. Flask追加「sudo pip3 install flask」
3. 「pip3 install mod_wsgi」実行。
3. 「pip3 install mod_wsgi-httpd」実行。

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
1. wsgiとのアダプターファイル「adapter.py」をflaskディレクトリ直下に作成する。ファイルの中身は以下とする。
```

# coding: utf-8
import sys
sys.path.insert(0, "/var/www/flask")

from app import app as application

```

### 3-3. Apache2の設定ファイルの作成
1. ディレクトリ「/etc/apache2/site-avaialble」直下に「flask.conf」を作成。ファイルの中身は以下とする。

```
LoadModule wsgi_module /usr/local/lib/python3.5/dist-packages/mod_wsgi/server/mod_wsgi-py35.cpython-35m-x86_64-linux-gnu.so

<VirtualHost *:80>
WSGIDaemonProcess snapmsg user=www-data group=www-data threads=5 python-home=/var/www/flask
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

* 1行目：LoadModuleのあとに「mod_wsgi-py35.cpython-35m-x86_64-linux-gnu.so」ファイルの場所を記載する。
* 5行目：「python-home=」の右側はFlaskのプロジェクトディレクトリを指定。
* 6行目：「/」と「wsgi設定ファイル」の場所を記載する。 ※ 本例では、「/var/www/flask/adapter.wsgi」
* 9行目：「Flaskのプロジェクトディレクトリを指定」。

#### .confファイルを読み込ませる
a2dissite "sites-enable内にある.confファイル名"
a2ensite "sites-available内にある.confファイル名"


### apache2.confに追記する

```
<Directory /var/www/flask/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>

```

## 4.WEBサーバの実行とFirewallの設定

### 4-1. Apache2の起動
1. 「systemctl start apache2」実行。

### 4-2. Firewallの設定
1. http通信の通過設定「firewall-cmd --add-service=http --zone=public --permanent」実行。
2. https通信の通過設定「firewall-cmd --add-service=https --zone=public --permanent」実行。
3. Firewallの再起動「systemctl restart firewalld」実行。

### 4-3. 稼働のテスト
1. ブラウザで「http://xxx.xxx.xxx.xxx（VPSのアドレス）」にアクセスする。「Hello, Worldだよ(*˘︶˘*).｡.:*♡」と表示されればOK！。

## 参考文献
* [Flask公式](http://flask.pocoo.org/docs/1.0/)
* [Flask公式 -mod_wsgiについて-](http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/)
* [ネコでもわかる！さくらのVPS講座 ～第三回「Apacheをインストールしよう」](https://knowledge.sakura.ad.jp/8541/)
