# Secure settings
一部の設定は機密性が高く、その値を保護するためにファイルシステムのパーミッションに依存することは十分ではありません。このようなユースケースのために、Kibanaは鍵ストアと、鍵ストア内の設定を管理するためのkibana-keystoreツールを提供します。

ここでのコマンドはすべて、Kibanaを実行するユーザーとして実行する必要があります。

## Create the keystore
kibana.keystoreを作成するには、createコマンドを使用します。

```
bin/kibana-keystore create
```

環境変数KBN_PATH_CONFで定義されたconfigディレクトリにkibana.keystoreというファイルが作成されます。

## List settings in the keystore
keystoreの設定の一覧は、listコマンドで見ることができます。

```
bin/kibana-keystore list
```

## Add string settings
Elasticsearchの認証情報など、機密性の高い文字列の設定はaddコマンドで追加することができます。

```
bin/kibana-keystore add the.setting.name.to.set
```

キーストアに追加されると、これらの設定はKibanaのこのインスタンスを起動したときに自動的に適用されます。たとえば、次のように実行すると

```
bin/kibana-keystore add elasticsearch.username
```

を実行すると、elasticsearch.username の値を入力するようプロンプトが表示されます。(入力はアスタリスクで表示されます。)

ツールは設定値の入力を促します。標準入力で値を渡すには、--stdinフラグを使用します。

```
cat /file/containing/setting/value | bin/kibana-keystore add the.setting.name.to.set --stdin
```

## Remove settings
キーストアから設定を削除するには、removeコマンドを使用します。

```
bin/kibana-keystore remove the.setting.name.to.remove
```
