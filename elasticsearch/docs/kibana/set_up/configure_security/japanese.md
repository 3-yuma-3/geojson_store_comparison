# Configure security in Kibana
Elasticsearchを初めて起動すると、クラスタ上でElastic Stackのセキュリティ機能が有効になり、TLSが自動的に構成されます。セキュリティ設定プロセスでは、ElasticユーザーのパスワードとKibanaの登録トークンが生成されます。セキュリティを有効にした状態でElastic Stackを起動し、設定プロセスの一環としてKibanaを登録します。

その後、エラスティックユーザーとしてKibanaにログインし、追加のロールとユーザーを作成することができます。

インデックス（Elasticsearchのインデックスなど）のデータを閲覧する権限がない場合、インデックス全体にアクセスできなくなり、Kibanaで表示されなくなります。


## Configure security settings
セッションが無効にならないように暗号化キーを設定します。オプションで追加のセキュリティ設定と認証を設定することができます。

1. kibana.yml 設定ファイルで xpack.security.encryptionKey プロパティを設定します。暗号化キーには、32文字以上の任意のテキスト文字列を使用できます。

```
xpack.security.encryptionKey: "something_at_least_32_characters"
```

詳しくは、「Kibanaのセキュリティ設定」を参照してください。

2. オプションです。Kibanaのセッションの有効期限を設定する。
3. オプション。クライアント証明書を使用して Elasticsearch に認証するように Kibana を設定します。
4. Kibana を再起動します。

## Create roles and users
Kibana ユーザーに対してロールを設定し、それらのユーザーがアクセスできるデータを制御します。

1. 組み込みの弾性スーパーユーザーを使用して Kibana に一時的にログインし、新しいユーザーの作成とロールの割り当てができるようにします。Kibana をローカルで実行している場合は、https://localhost:5601 にアクセスしてログイン ページを表示します。

組み込みのエラスティックユーザーのパスワードは、Elasticsearch のセキュリティ設定プロセスの一部として生成されます。エラスティックユーザーや他の組み込みユーザーのパスワードをリセットする必要がある場合は、elasticsearch-reset-password ツールを実行します。

2. Kibanaへのアクセス権を付与するために、ロールとユーザーを作成します。

Kibanaの権限を管理するには、メインメニューを開き、[スタック管理] > [ロール]をクリックします。組み込みの kibana_admin ロールは、管理者権限を持つ Kibana へのアクセスを許可します。また、Kibana への限定的なアクセスを許可する追加のロールを作成することもできます。

Basic認証でデフォルトのネイティブレルムを使用している場合、メインメニューを開き、Stack Management > Usersをクリックしてユーザーを作成しロールを割り当てるか、Elasticsearchのユーザー管理APIを使用します。例えば、以下は jacknich というユーザーを作成し、kibana_admin というロールを割り当てています。

```
POST /_security/user/jacknich
{
  "password" : "t0pS3cr3t",
  "roles" : [ "kibana_admin" ]
}
```

3. Kibana で作業するインデックスへのアクセス権をユーザーに付与します。

Kibana ユーザーには、必要な数だけ異なるロールを定義することができます。

たとえば、特定のデータビューで読み取り権限と view_index_metadata 権限を持つロールを作成します。詳細については、「ユーザーの権限設定」を参照してください。

4. Kibana からログアウトし、通常のユーザーとしてログインできることを確認します。ローカルで Kibana を実行している場合は、https://localhost:5601 にアクセスし、Kibana のユーザー ロールを割り当てたユーザーの資格情報を入力します。たとえば、ユーザー jacknich としてログインできます。

これは、Kibana の権限を割り当てられたユーザーである必要があります。Kibana サーバーの資格情報 (組み込みの kibana_system ユーザー) は、Kibana サーバーが内部的にのみ使用する必要があります。
