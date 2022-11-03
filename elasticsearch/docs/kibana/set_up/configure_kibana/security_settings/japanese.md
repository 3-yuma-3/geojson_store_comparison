# Security settings in Kibana
Kibana のセキュリティ機能を使用するために、追加の設定をする必要はありません。これらはデフォルトで有効になっています。

高可用性の展開では、Kibana のすべてのインスタンスで同じセキュリティ設定を使用することを確認してください。また、暗号化キーや復号化キーなどの機密性の高いセキュリティ設定は、kibana.yml ファイルに平文で保管するのではなく、Kibana Keystore に安全に保管することを検討してください。

### Authentication security settings
kibana.yml の xpack.security.authc 名前空間で認証設定を行います。

例えば、以下のようになります。

```yml
xpack.security.authc:
    providers:
      basic.basic1: ---1
          order: 0 ---2
          ...

      saml.saml1: ---3
          order: 1
          ...

      saml.saml2: ---4
          order: 2
          ...

      pki.realm3:
          order: 3
```

1. 認証プロバイダーの種類（例：basic、token、saml、oidc、kerberos、pki）およびプロバイダー名を指定します。この設定は必須です。
2. 認証チェーンおよびログインセレクタのUIにおけるプロバイダの順序を指定します。この設定は必須です。
3. SAML認証プロバイダーの設定をsaml1名で指定します。
4. SAML認証プロバイダーの設定をsaml2名で指定します。

### Valid settings for all authentication providers
xpack.security.authc.providers 名前空間の有効な設定は、認証プロバイダの種類によって異なります。詳しくは、「認証」を参照してください。

- `xpack.security.authc.providers.<provider-type>.<provider-name>.enabled`
  - 認証プロバイダーを有効にするかどうかを決定します。デフォルトでは、Kibana は、そのプロパティのいずれかを設定するとすぐにプロバイダーを有効にします。
- `xpack.security.authc.providers.<provider-type>.<provider-name>.order`
  - 認証チェーンとログインセレクタUI上のプロバイダの順序。
- `xpack.security.authc.providers.<provider-type>.<provider-name>.description`
  - ログインセレクタのUIに表示されるプロバイダエントリのカスタム説明文です。
- `xpack.security.authc.providers.<provider-type>.<provider-name>.hint`
  - ログインセレクタのUIに表示されるプロバイダエントリのカスタムヒント。
- `xpack.security.authc.providers.<provider-type>.<provider-name>.icon`
  - ログインセレクタのUIに表示されるプロバイダエントリのカスタムアイコン。
- `xpack.security.authc.providers.<provider-type>.<provider-name>.showInSelector`
  - プロバイダがログインセレクタUIにエントリを持つべきかどうかを示すフラグ。falseに設定しても、認証チェーンからプロバイダが削除されることはありません。
  - 基本認証およびトークン認証のプロバイダには、この設定をfalseにすることはできません。
- `xpack.security.authc.providers.<provider-type>.<provider-name>.accessAgreement.message`
  - Markdown形式のアクセス合意書テキスト。詳細については、「アクセス権契約」を参照してください。
- `xpack.security.authc.providers.<provider-type>.<provider-name>.session.idleTimeout`
  - ユーザーセッションが一定時間使用されないと失効するようにします。この値を 0 に設定すると、非アクティブによるセッションの失効を防ぐことができます。デフォルトでは、この設定は xpack.security.session.idleTimeout と同じです。
  - [ms|s|m|h|d|w|M|Y] (e.g. 20m, 24h, 7d, 1w)の文字列を使用します。
- `xpack.security.authc.providers.<provider-type>.<provider-name>.session.lifespan`
  - ユーザーセッションが定義された期間の後に期限切れになることを保証します。この動作は、「絶対タイムアウト」とも呼ばれます。この値を0に設定すると、ユーザーセッションは無期限にアクティブになります。デフォルトでは、この設定はxpack.security.session.lifespanと等しくなっています。
  - [ms|s|m|h|d|w|M|Y] (e.g. 20m, 24h, 7d, 1w)の文字列を使用します。

### SAML authentication provider settings
すべてのプロバイダーで有効な設定に加え、以下の設定を指定することができます。

- `xpack.security.authc.providers.saml.<provider-name>.realm`
  - プロバイダが使用すべきElasticsearch内のSAMLレルム。
- xpack.security.authc.providers.saml.<provider-name>.useRelayStateDeepLink`
  - プロバイダが、Identity Provider によるログイン中に、RelayState パラメータを Kibana のディープリンクとして扱うかどうかを決定します。デフォルトでは、この設定は false に設定されています。RelayStateで指定されるリンクは、相対的な、URLエンコードされたKibanaのURLである必要があります。例えば、RelayState パラメータの /app/dashboards#/list のリンクは以下のようになります。RelayState=%2Fapp%2Fdashboards%23%2Flistのようになります。

### OpenID Connect authentication provider settings
すべてのプロバイダーで有効な設定に加え、以下の設定を指定することができます。

- `xpack.security.authc.providers.oidc.<provider-name>.realm`
  - プロバイダが使用すべきElasticsearch内のOpenID Connectレルム。


### Anonymous authentication provider settings
すべてのプロバイダーで有効な設定に加え、以下の設定を指定することができます。

匿名プロバイダーは、Kibana インスタンスごとに 1 つだけ設定できます。

- `xpack.security.authc.providers.anonymous.<provider-name>.credentials`
  - Elasticsearch への匿名リクエストを認証するために Kibana が内部で使用すべきクレデンシャル。

例えば、以下のようなものです。

```
xpack.security.authc.providers.anonymous.anonymous1:
  credentials:
    username: "anonymous_service_account"
    password: "anonymous_service_account_password"
```

詳しくは、「匿名認証」を参照してください。

### HTTP authentication settings
これらの設定を変更したいケースは、非常に限られています。詳細については、「HTTP認証」を参照してください。

- `xpack.security.authc.http.enabled`
  - HTTP認証を有効にするかどうかを指定します。デフォルトでは、この設定はtrueに設定されています。
- `xpack.security.authc.http.autoSchemesEnabled`
  - 有効な認証プロバイダで使用されるHTTP認証スキームをHTTP認証時に自動的にサポートするかどうかを決定します。デフォルトでは、この設定はtrueに設定されています。
- `xpack.security.authc.http.schemes[]`
  - Kibana HTTP認証がサポートすべきHTTP認証スキームのリスト。デフォルトでは、この設定は ['apikey', 'bearer'] に設定され、ApiKey および Bearer スキームを使用した HTTP 認証をサポートします。

### Login user interface settings
kibana.yml ファイルで以下の設定を行うことができます。

- `xpack.security.loginAssistanceMessage`
  - ログインUIにメッセージを追加します。メンテナンスウィンドウの情報や、企業のサインアップページへのリンクなどを表示するのに便利です。
- `xpack.security.loginHelp`
  - ログインUIでアクセス可能なメッセージに、ログイン処理に関する追加のヘルプ情報を追加します。
- `xpack.security.authc.selector.enabled`
  - ログインセレクタのUIを有効にするかどうかを決定します。デフォルトでは、複数の認証プロバイダーが構成されている場合、この設定はtrueに設定されます。

### Configure a default access agreement
kibana.yml ファイルで以下の設定を行うことができます。

- `xpack.security.accessAgreement.message`
  - この設定は、xpack.security.authc.providers.<プロバイダータイプ>.<プロバイダー名>.accessAgreement.message に値を指定しないすべてのプロバイダーに対するデフォルトアクセス契約として使用する Markdown 形式のアクセス契約テキストを指定するものです。詳しくは、「アクセス権」を参照してください。


### Session and cookie security settings
kibana.yml ファイルで以下の設定を行うことができます。

- `xpack.security.cookieName`
  - セッションに使用するCookieの名前を設定します。デフォルトは "sid "です。
- `xpack.security.encryptionKey`
  - セッション情報を暗号化するために使用する32文字以上の任意の文字列。このキーはKibanaのユーザーには公開しないでください。デフォルトでは、値がメモリ内で自動的に生成されます。そのデフォルトの動作を使用すると、Kibanaの再起動時にすべてのセッションが無効となります。また、この設定が Kibana のすべてのインスタンスで同じでない場合、Kibana の高可用性デプロイは予期しない動作をします。
- `xpack.security.secureCookies`
  - セッション Cookie のセキュアフラグを設定します。デフォルトはfalseです。server.ssl.enabled が true に設定されている場合、自動的に true に設定されます。SSL が Kibana の外部で設定されている場合 (たとえば、ロードバランサーやプロキシを経由してリクエストをルーティングしている場合)、この値を true に設定します。
- `xpack.security.sameSiteCookies`
  - セッション Cookie の SameSite 属性を設定します。これにより、クッキーがファーストパーティコンテキストまたはセイムサイトコンテキストに制限されるべきかを宣言することができます。有効な値は、Strict、Lax、Noneです。これはデフォルトでは設定されておらず、モダンブラウザではLaxとして扱われます。モダン ブラウザで iframe に埋め込まれた Kibana を使用する場合、None に設定する必要がある場合があります。この値を None に設定すると、xpack.security.secureCookies: true を設定して、Cookie を安全な接続で送信することが必要になります。
- `xpack.security.session.idleTimeout`
  - ユーザーセッションが一定期間使用されないと失効するようにします。xpack.security.session.lifespanとあわせて、この設定を強く推奨します。この設定は、プロバイダーごとに個別に指定することもできます。この値を0に設定すると、非アクティブによるセッションの失効が発生しません。デフォルトでは、この値は8時間です。
  - [ms|s|m|h|d|w|M|Y] (e.g. 20m, 24h, 7d, 1w)の文字列を使用します。
- `xpack.security.session.lifespan`
  - ユーザーセッションが定義された期間の後に失効することを保証します。この動作は、「絶対タイムアウト」とも呼ばれます。これを0に設定すると、ユーザーセッションは無期限にアクティブになります。この設定とxpack.security.session.idleTimeoutの両方が強く推奨されます。また、この設定はプロバイダごとに個別に指定することもできます。デフォルトでは、この値は30日です。
  - [ms|s|m|h|d|w|M|Y] (例: 20m, 24h, 7d, 1w)の文字列を使用します。
- `xpack.security.session.cleanupInterval`
  - Kibana がセッション インデックスから期限切れのセッションと無効なセッションを削除しようとする間隔を設定します。デフォルトでは、この値は 1 時間です。最小値は 10 秒です。
  - [ms|s|m|h|d|w|M|Y] (e.g. 20m, 24h, 7d, 1w) の文字列を使用します。

## Encrypted saved objects settings
これらの設定は、機密データを含む保存オブジェクトの暗号化を制御します。詳細については、「保存されたオブジェクトのセキュリティ」を参照してください。

- `xpack.encryptedSavedObjects.encryptionKey`
  - Elasticsearch に保存される前に、保存されたオブジェクトの機密プロパティを暗号化するために使用される、少なくとも 32 文字の任意の文字列です。設定しない場合、Kibana は起動時にランダムなキーを生成しますが、暗号化キーを明示的に設定するまで特定の機能は使用できません。
- `xpack.encryptedSavedObjects.keyRotation.decryptionOnlyKeys`
  - 以前に使用された暗号化キーのオプションリストです。xpack.encryptedSavedObjects.encryptionKey と同様に、これらは少なくとも 32 文字の長さである必要があります。Kibana はこれらのキーを暗号化に使用しませんが、一部の既存の保存されたオブジェクトを復号化するためにこれらのキーを必要とする場合があります。暗号化キーを変更したいが、以前に別のキーで暗号化された保存オブジェクトへのアクセスを失いたくない場合に、この設定を使用します。


### Audit logging settings
監査ログを有効にして、コンプライアンス、説明責任、およびセキュリティをサポートすることができます。有効にすると、Kibana は以下を記録します。

- 誰がアクションを実行したか
- どのようなアクションが実行されたか
- いつアクションが発生したか

詳細および監査イベントの参照については、「監査ログ」を参照してください。

- `xpack.security.audit.enabled`
  - 監査ログを有効にするにはtrueに設定します`。デフォルト: false

例えば、こんな感じです。

```yml
xpack.security.audit.enabled: true
xpack.security.audit.appender: ---1
  type: rolling-file
  fileName: ./logs/audit.log
  policy:
    type: time-interval
    interval: 24h ---2
  strategy:
    type: numeric
    max: 10 ---3
  layout:
    type: json
```

1. このアペンダーはデフォルトであり、appender.*設定オプションが指定されていない場合に使用されます。
2. 24時間ごとにログファイルをローテートします。
3. 古いログファイルを削除する前に、最大10個のログファイルを保持します。

- `xpack.security.audit.appender`
  - オプション。監査ログを書き込む場所とその書式を指定します。アペンダーを指定しない場合、デフォルトのアペンダーが使用されます（上記を参照）。
- `xpack.security.audit.appender.type`
  - 必須。監査ログが書き込まれる場所を指定します。許可された値は、console、file、またはrolling-fileです。
  - アペンダー固有の設定については、ファイルアペンダーとローリングファイルアペンダーを参照してください。
- `xpack.security.audit.appender.layout.type`
  - 必須。監査ログがどのようにフォーマットされるべきかを指定します。許可された値は、jsonまたはpatternです。
  - レイアウト固有の設定については、パターンレイアウトを参照してください。
  - Filebeatを使用してKibana監査ログをElasticsearchに取り込むことを可能にするために、json形式の使用を推奨します。

### File appender
ファイルアペンダはファイルへの書き込みを行うもので、以下のような設定が可能です。

- `xpack.security.audit.appender.fileName`
  - 必須 ログファイルの書き出し先のフルファイルパス。

### Rolling file appender
ローリングファイルアペンダーは、特定のポリシーがトリガーされたときに、ローリング戦略を使用してファイルに書き込み、それを回転させます。

- `xpack.security.audit.appender.fileName`
  - 必須。ログファイルが書き込まれるべき完全なファイルパス。
- `xpack.security.audit.appender.policy.type`
  - ロールオーバーが発生するタイミングを指定します。許可された値は、size-limitとtime-intervalです。デフォルトはtime-intervalです。
  - ポリシー固有の設定については、サイズ制限ポリシーと時間間隔ポリシーを参照してください。
- `xpack.security.audit.appender.strategy.type`
  - ロールオーバーがどのように発生するかを指定する。現在、数値のみが許可されています。デフォルト: numeric
  - ストラテジー固有の設定については、「数値ストラテジー」を参照してください。

### Size limit triggering policy
サイズ制限のトリガーとなるポリシーは、ファイルが特定のサイズに達したときにローテーションを行います。

- `xpack.security.audit.appender.policy.size`
  - ロールオーバーが実行される前に、ログファイルが到達すべき最大サイズ。デフォルト：100mb

### Time interval triggering policy
時間間隔トリガーポリシーは、与えられた時間間隔ごとにファイルを回転させる。

- `xpack.security.audit.appender.policy.interval`
  - ロールオーバーが発生する頻度。デフォルト：24時間
- `xpack.security.audit.appender.policy.modulate`
  - インターバルの境界で次のロールオーバーを発生させるために、インターバルを調整するかどうか。デフォルト: true

### Numeric rolling strategy
数値ロールオーバー戦略は、ロールオーバー時に与えられたパターンでログファイルをサフィックスし、ロールされたファイルの固定数を保持します。

- `xpack.security.audit.appender.strategy.pattern`
  - ロールオーバーするときにファイル名に追加するサフィックス。iを含む必要があります。デフォルト：-%i
- `xpack.security.audit.appender.strategy.max`
  - 保持するファイルの最大数。この数に達すると、最も古いファイルが削除されます。デフォルト: 7

### Pattern layout
パターンレイアウトは、実際のログメッセージからのデータで置き換えられる、特別なプレースホルダーを持つパターンを使用してフォーマットされた文字列を出力します。

- `xpack.security.audit.appender.layout.pattern`
  - オプション。ログ行がどのようにフォーマットされるべきかを指定します。デフォルト: [%date][%level][%logger]%meta %message
- `xpack.security.audit.appender.layout.highlight`
  - オプション。ログメッセージを色でハイライトすることを有効にするには、trueに設定します。


### Ignore filters
- `xpack.security.audit.ignore_filters[]`
  監査ログから除外するイベントを決定するフィルタのリストです。提供されたフィルタの少なくとも1つが一致する場合、イベントはフィルタリングされます。

例えば

```yml
xpack.security.audit.ignore_filters:
- actions: [http_request] ---1
- categories: [database]
  types: [creation, change, deletion] ---2
```

1. HTTPリクエストイベントのフィルタリング
2. データ書き込みイベントのフィルタリング


- `xpack.security.audit.ignore_filters[].actions[]`
  - 監査イベントのevent.actionフィールドと一致する値のリスト。利用可能なイベントの一覧は、「監査ログ」を参照してください。
- `xpack.security.audit.ignore_filters[].categories[]`
  - 監査イベントのevent.categoryフィールドと一致する値のリスト。許可される値については、ECS カテゴリフィールドを参照してください。
- `xpack.security.audit.ignore_filters[].types[]`
  - 監査イベントのevent.typeフィールドと一致する値のリスト。許可された値については、ECSタイプフィールドを参照してください。
- `xpack.security.audit.ignore_filters[].outcomes[]`
  - 監査イベントのevent.outcomeフィールドと一致する値のリスト。許可された値については、ECSの結果フィールドを参照してください。
