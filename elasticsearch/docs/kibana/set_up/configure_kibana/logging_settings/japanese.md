# Logging settings in Kibana
Kibanaのログ機能を利用するために、追加で設定する必要はありません。ログ機能はデフォルトで有効になっており、stdoutにログを出力するpatternレイアウトを使用してinfoレベルでログを出力します。

ただし、Elasticsearchなどを使ってログをインジェストする場合は、ECS形式でログを出力するjsonレイアウトの利用をおすすめします。一般的に、人間が生のログを読む場合はパターンレイアウトを、機械がログを読む場合はjsonレイアウトを推奨します。

ログの設定は事前に定義されたスキーマに対して検証され、それに問題がある場合、Kibanaは詳細なエラーメッセージとともに起動に失敗します。

Kibanaはロギングサービスを設定するために、アペンダー、ロガー、ルートという3つのハイレベルなエンティティに依存しています。これらは kibana.yml の logging ネームスペースで設定することができます。

- アペンダーは、ログメッセージを表示する場所（標準出力またはコンソール）およびそのレイアウト（パターンまたはjson）を定義します。また、ログを保存するかどうか、保存する場合はどこに保存するか（ディスク上のファイル）を指定することも可能です。
- ロガーは、特定のコンテキストに適用する冗長度やアペンダーなど、ログの設定を定義します。各ログエントリコンテキストは、それを発行するサービスまたはプラグイン、およびそのサブパーツのいずれか（例えば、metrics.ops や elasticsearch.query）に関する情報を提供します。
- ルートは、Kibana のすべてのログ・エントリに適用されるロガーです。

次の表は、異なるロギング構成キーのクイックリファレンスとして機能します。これらはスタンドアロン設定ではなく、追加のロギング設定が必要な場合があることに注意してください。一般的な設定のユースケースについては、「Configure Logging in Kibana」ガイドと完全な例を参照してください。

- `logging.appenders[].<appender-name>`
  - 一意のアペンダー識別子です。

- `logging.appenders[].console:`
  - stdoutへのロギングレコードに使用するアペンダーです。デフォルトでは、[%date][%level][%logger] %messageパターンのレイアウトを使用します。jsonを使用するには、layout typeをjsonに設定します。

- `logging.appenders[].file:`
  - ログレコードをディスクに書き込むためのfileNameを指定できます。すべてのログレコードをファイルに書き込むには、root.appendersにファイルアペンダーを追加します。設定されている場合、logging.appenders.file.pathNameも指定する必要があります。

- `logging.appenders[].rolling-file:`
  - Log4jのRollingFileAppenderと同様に、このアペンダーは、設定されたポリシーがトリガーされたとき、ローリング戦略に従うなら、ファイルにログを取り、ローテーションする。現在、サポートされている2つのポリシーがあります：size-limitとtime-intervalです。

- `logging.appenders[].<appender-name>.type`
  - アペンダーのタイプは、ログメッセージが送信される場所を決定します。オプションは、console、file、rewrite、rolling-fileです。必須です。

- `logging.appenders[].<appender-name>.fileName`
  - fileとrolling-fileアペンダタイプのために、ログメッセージが書き込まれるファイルパスを決定します。fileに書き込むアペンダーのために必要です。

- `logging.appenders[].<appender-name>.policy.type`
  - ローリングファイルタイプのアペンダーのために、いつロールオーバーが起こるべきかのトリガーポリシーを指定します。

- `logging.appenders[].<appender-name>.policy.interval`
  - time-intervalタイプのローリングファイルアペンダーのログファイルを回転させるための時間間隔を指定します。デフォルトは24時間

- `logging.appenders[].<appender-name>.policy.size`
  - size-limit タイプのローリングファイルアペンダーのために、ポリシーがロールオーバーをトリガーするサイズリミットを指定します。デフォルトは100mbです。

- `logging.appenders[].<appender-name>.policy.interval`
  - 時間間隔タイプのローリングファイルアペンダーのために、ポリシーがロールオーバーをトリガーする時間間隔を指定します。

- `logging.appenders[].<appender-name>.policy.modulate`
  - 間隔の境界で次のロールオーバーを発生させるために間隔を調整するかどうかを指定します。ブール値。デフォルトはtrue。

- `logging.appenders[].<appender-name>.strategy.type`
  - ローリングファイルのストラテジーの種類を指定します。現在、数値のみがサポートされています。

- `logging.appenders[].<appender-name>.strategy.pattern`
  - ローリング時にファイルパスに追加するサフィックス。i を含める必要があります。

- `logging.appenders[].<appender-name>.strategy.max`
  - 保持するファイルの最大数。オプション。デフォルトは7で、最大は100です。

- `logging.appenders[].<appender-name>.layout.type`
  - ログメッセージがどのように表示されるかを決定します。オプションは、人間が読める出力を提供するpattern、またはECSに準拠した出力を提供するjsonです。必須です。

- `logging.appenders[].<appender-name>.layout.highlight`
  - ログメッセージを色でハイライトするためのオプションのブール値です。パターンレイアウトにのみ適用されます。デフォルトはfalseです。

- `logging.appenders[].<appender-name>.layout.pattern`
  - 実際のログメッセージからのデータで置き換えられるプレースホルダーのためのオプションの文字列パターン。パターンタイプのレイアウトにのみ適用されます。

- `logging.root.appenders[]`
  - ルートに適用する特定のアペンダーのリスト。デフォルトは、パターンレイアウトでコンソールになります。

- `logging.root.level`
  - 個々のロガー・レベルで特に構成されていない場合、フォールバックするすべてのログ・メッセージのデフォルトの冗長性を指定します。オプションは、all、fatal、error、warn、info、debug、trace、offです。allおよびoffレベルは、設定でのみ使用でき、すべてのログレコードをログに記録したり、ログを完全にまたは特定のロガーに対して無効にしたりできる、便利なショートカットです。デフォルトはinfoです。

- `logging.loggers[].<logger>.name:`
  - 特定のロガー・インスタンス。

- `logging.loggers[].<logger>.level`
  - <logger> コンテキストに対するログメッセージの冗長性を指定します。オプションで、ルート ロガー レベルまでのすべての祖先ロガーの冗長性を継承します。

- `logging.loggers[].<logger>.appenders`
  - 特定のロガーコンテキストに適用するアペンダーを配列として決定します。オプションで、指定しない場合、ルート ロガーのアペンダーにフォールバックします。
