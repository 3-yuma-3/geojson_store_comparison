# Configure Kibana
Kibana サーバーは起動時に kibana.yml ファイルからプロパティを読み取ります。このファイルの場所は、Kibana をインストールした方法によって異なります。例えば、Kibana をアーカイブディストリビューション (.tar.gz または .zip) からインストールした場合、デフォルトでは $KIBANA_HOME/config に格納されます。パッケージディストリビューション（DebianやRPM）の場合、デフォルトでは/etc/kibanaになります。設定ディレクトリは、環境変数KBN_PATH_CONFで変更することができます。

```
KBN_PATH_CONF=/home/kibana/config ./bin/kibana
```

デフォルトのホストおよびポート設定は、Kibana が localhost:5601 で実行されるように構成されています。この動作を変更し、リモートユーザーの接続を許可するには、kibana.yml ファイルを更新する必要があります。また、SSLを有効にし、その他のさまざまなオプションを設定することもできます。最後に、環境変数は ${MY_ENV_VAR} 構文を使用して設定に注入することができます。


【以下、TODO】
