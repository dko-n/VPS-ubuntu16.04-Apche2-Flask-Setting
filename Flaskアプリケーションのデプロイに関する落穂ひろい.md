# 😛Flaskアプリケーションのデプロイに関する落穂ひろい😛

## 1. 追加パッケージの使用について
#### [WTForms](https://wtforms.readthedocs.io/en/2.3.x/)
* CSRF対策がデフォルトで実装されているのでFlask-WTFormsと合わせてフォームの構築によく使用されるが、本番環境上で使用する際には、数点気を付ける点が存在する。
##### 1. CSRFのチェックが通らない場合。
  * Flask-WTFormsを使用している場合、POSTでフォーム内容を送った後に「The CSRF session token is missing.」というメッセージが出続けた場合は、FlaskのconfigのSECRET_KEYを疑う。
  * ランダム生成されたSECRET_KEYを設定すると、**リクエスト間でSessionが持ちまわされないことがあることに注意。**
```
# flaskのアプリケーションで以下のような設定をWeb上でよく見かけるが、
# これはバッドプラクティス。固定の推測不可能な文字列を与えるのが正解。
app.config.update(
    SECRET_KEY=os.urandom(24)
)
```

## 2. Configの設定について
#### [Flask Config](https://flask.palletsprojects.com/en/1.1.x/config/#configuration-basics)
* 本番環境上にアプリをデプロイする場合、FlaskアプリのConfig設定には注意が必要なので以下に記載する。
##### 1. DEBUG
 * デバッグモードの有効/無効設定。本番環境上ではもちろん**False**。
 
##### 2. TESTING
 * テストモードの有効/無効設定。本番環境上ではもちろん**False**。

##### 3. SECRET_KEY
 * Session Cookieの署名用に使用する。リクエスト間でSessionが持ちまわされないことがあるので、本番環境上では**固定の推測不可能な文字列を与える**。
 * os.urandom(24)などのランダム文字列生成はバッドプラクティス。

##### 4. SESSION_COOKIE_SECURE
 * Trueの場合は、HTTPSアクセスの場合でのみレスポンスにCookieを送信する。
 * これがTrueになっていて、本番環境がHTTPアクセスである場合は、Session Cookieが利用できないので注意すること。

##### 5. PERMANENT_SESSION_LIFETIME
 * Session Cookieの有効期限を設定する。
 * datetime.timedeltaかint側で秒数を設定する。
 
##### 6. SESSION_REFRESH_EACH_REQUEST
 * すべてのレスポンスにCookie情報を送信するか設定する。（Trueの場合送信する）
 * Sessionの期限切れを防ぐために利用される。
