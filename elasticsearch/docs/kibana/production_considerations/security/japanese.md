# Security production considerations
本番環境での Kibana インストールを保護するために、これらの優先順位の高いトピックを考慮し、許可されたユーザーのみが Kibana にアクセスできるようにします。Kibana のセキュリティ制御の詳細については、「セキュリティを設定する」を参照してください。

## Enable SSL/TLS
ブラウザと Kibana サーバー間のトラフィックを第三者が閲覧または改ざんできないようにするため、SSL/TLS 暗号化を使用する必要があります。KibanaのHTTPクライアント通信を暗号化するを参照してください。

encrypt-kibana-http

## Use Elastic Stack security features
Elastic Stackのセキュリティ機能を使用すると、Kibanaを通じてユーザーがアクセスできるElasticsearchのデータを制御することができます。

セキュリティ機能が有効な場合、Kibanaユーザーはログインする必要があります。彼らはKibanaの特権を付与するロールと、Kibanaで作業するインデックスへのアクセス権を持っている必要があります。

ユーザーが、閲覧権限のないインデックス内のデータにアクセスする Kibana ダッシュボードをロードした場合、インデックスが存在しないことを示すエラーが表示されます。

Kibana へのアクセス権付与の詳細については、「Kibana へのアクセス権付与」を参照してください。

## Use secure HTTP headers
Kibanaサーバーは、HTTPヘッダーを使用して追加のセキュリティ制御を有効にするようにブラウザに指示することができます。

1. HTTP Strict-Transport-Securityを有効にします。

strictTransportSecurity を使用して、ブラウザが SSL/TLS 暗号化で Kibana にのみアクセスしようとするようにします。これは、マニピュレーター・イン・ザ・ミドル攻撃を防止するためのものです。kibana.yml で有効期限を 1 年として設定する。

```
server.securityResponseHeaders.strictTransportSecurity: "max-age=31536000"
```

このヘッダーは、ドメイン全体の暗号化されていない接続をブロックします。同じドメインで、異なるポートやパスを使用して複数のウェブアプリケーションをホストしている場合、それらすべてに影響が及びます。

2. 埋め込みを無効にする

disableEmbedding を使用して、Kibana を他の Web サイトに埋め込むことができないようにします。kibana.yml で設定する場合。

```
server.securityResponseHeaders.disableEmbedding: true
```

## Require a Content Security Policy
Kibana はコンテンツ セキュリティ ポリシー (CSP) を使用して、安全でないスクリプトをブラウザで許可しないようにしますが、古いブラウザではこのポリシーが黙って無視されます。お客様の組織で、弊社のサポートするブラウザーの非常に古いバージョンをサポートする必要がない場合は、CSP に対して Kibana の厳格モードを有効にすることをお勧めします。これにより、CSP 保護の初歩的なセットさえも実施しないブラウザーの Kibana へのアクセスがブロックされます。

これを行うには、kibana.yml で csp.strict を true に設定します。

```
csp.strict: true
```
