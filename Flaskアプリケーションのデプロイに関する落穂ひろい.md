# 😛Flaskアプリケーションのデプロイに関する落穂ひろい😛

## 1. 追加パッケージの使用について
#### [WTForms](https://wtforms.readthedocs.io/en/2.3.x/)
* CSRF対策がデフォルトで実装されているのでFlask-WTFormsと合わせてフォームの構築によく使用されるが、本番環境上で使用する際には、数点気を付ける点が存在する。
1. CSRFのチェックが通らない場合。
  * Flask-WTFormsを使用している場合、POSTでフォーム内容を送った後に「The CSRF session token is missing.」というメッセージが出続けた場合は、FlaskのconfigのSECRET_KEYを疑う。
  * ランダム生成されたSECRET_KEYを設定すると、**リクエスト間でSessionが持ちまわされないことがあることに注意。**
```
# flaskのアプリケーションで以下のような設定をWeb上でよく見かけるが、
# これはバッドプラクティス。固定の推測不可能な文字列を与えるのが正解。
app.config.update(
    SECRET_KEY=os.urandom(24)
)
```
