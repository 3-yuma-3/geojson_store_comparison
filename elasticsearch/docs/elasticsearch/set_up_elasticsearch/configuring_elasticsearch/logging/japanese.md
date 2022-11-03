# Logging
Elasticsearchのアプリケーションログは、クラスタの監視や問題の診断に利用することができます。Elasticsearchをサービスとして実行する場合、ログのデフォルトの保存場所は、プラットフォームとインストール方法によって異なります。

Docker
Docker上では、ログメッセージはコンソールに送られ、設定されたDockerロギングドライバによって処理されます。ログにアクセスするには、docker logsを実行します。

Elasticsearchをコマンドラインから実行した場合、Elasticsearchは標準出力（stdout）にログを出力します。

## Logging configuration
Elasticでは、デフォルトで出荷されているLog4j 2のコンフィギュレーションを使用することを強く推奨します。

Elasticsearchは、ログ取得にLog4j 2を使用します。Log4j 2 は log4j2.properties ファイルを使用して設定することができます。Elasticsearch は、${sys:es.logs.base_path}, ${sys:es.logs.cluster_name}, ${sys:es.logs.node_name} という三つのプロパティを公開し、設定ファイル内で参照してログファイルの場所を決定することができるようになっています。プロパティ ${sys:es.logs.base_path} はログディレクトリ、 ${sys:es.logs.cluster_name} はクラスタ名（デフォルトではログファイル名のプレフィックスとして使用）、 ${sys:es.logs.node_name} はノード名（ノード名を明示的に設定した場合）に解決される。

例えば、ログディレクトリ（path.logs）が /var/log/elasticsearch で、クラスタ名が production の場合、 ${sys:es.logs.base_path} は /var/log/elasticsearch に、 ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}.log は /var/log/elasticsearch/production.log にリゾルブされることになります。

```log
######## Server JSON ############################
appender.rolling.type = RollingFile ---1
appender.rolling.name = rolling
appender.rolling.fileName = ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}_server.json ---2
appender.rolling.layout.type = ECSJsonLayout ---3
appender.rolling.layout.dataset = elasticsearch.server ---4
appender.rolling.filePattern = ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}-%d{yyyy-MM-dd}-%i.json.gz ---5
appender.rolling.policies.type = Policies
appender.rolling.policies.time.type = TimeBasedTriggeringPolicy ---6
appender.rolling.policies.time.interval = 1 ---7
appender.rolling.policies.time.modulate = true ---8
appender.rolling.policies.size.type = SizeBasedTriggeringPolicy ---9
appender.rolling.policies.size.size = 256MB ---10
appender.rolling.strategy.type = DefaultRolloverStrategy
appender.rolling.strategy.fileIndex = nomax
appender.rolling.strategy.action.type = Delete ---11
appender.rolling.strategy.action.basepath = ${sys:es.logs.base_path}
appender.rolling.strategy.action.condition.type = IfFileName ---12
appender.rolling.strategy.action.condition.glob = ${sys:es.logs.cluster_name}-* ---13
appender.rolling.strategy.action.condition.nested_condition.type = IfAccumulatedFileSize ---14
appender.rolling.strategy.action.condition.nested_condition.exceeds = 2GB ---15
################################################
```

1. RollingFile アペンダーの設定
2. ログは /var/log/elasticsearch/production_server.json に記録する。
3. JSONレイアウトを使用する。
4. datasetはECSJsonLayoutのevent.datasetフィールドに入力するフラグです。これは、ログを解析する際に、異なるタイプのログをより簡単に区別するために使用することができます。
5. ログを /var/log/elasticsearch/production-yyyy-MM-dd-i.json にロールする; ログはロールするたびに圧縮され、i が増加する
6. 時間ベースのロールポリシーを使用する
7. 日単位でログをロールする
8. 24時間ごとにロールするのではなく、1日単位でロールする。
9. サイズベースのロールポリシーを使用する
10. 256MBを超えたらログをロールバックする
11. ログをロールバックするときに削除アクションを使用する
12. ファイルパターンに一致するログのみを削除する
13. メインログのみを削除するパターン
14. 圧縮ログがたまりすぎた場合のみ削除する
15. 圧縮ログのサイズ条件は2GBとする

```log
######## Server -  old style pattern ###########
appender.rolling_old.type = RollingFile
appender.rolling_old.name = rolling_old
appender.rolling_old.fileName = ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}_server.log ---1
appender.rolling_old.layout.type = PatternLayout
appender.rolling_old.layout.pattern = [%d{ISO8601}][%-5p][%-25c{1.}] [%node_name]%marker %m%n
appender.rolling_old.filePattern = ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}-%d{yyyy-MM-dd}-%i.old_log.gz
```

1. 古いスタイルのパターンアペンダーの設定です。これらのログは *.log ファイルに保存され、アーカイブされた場合は *.log.gz ファイルに保存されます。これらは非推奨とされ、将来的に削除されることに注意してください。

Log4jの設定パースは、余計なホワイトスペースによって混乱します。このページでLog4j設定をコピー＆ペーストするか、一般的にLog4j設定を入力する場合、すべての先行および後続のホワイトスペースをトリミングすることを確認してください。

Appender.rolling.filePattern の .gz を .zip に置き換えることで、zip形式を使用してロールされたログを圧縮することができることに注意してください。.gz拡張子を削除した場合、ログは、ロールされる時に圧縮されません。

指定された期間、ログファイルを保持したい場合、削除アクションでロールオーバー戦略を使用することができます。

```log
appender.rolling.strategy.type = DefaultRolloverStrategy ---1
appender.rolling.strategy.action.type = Delete ---2
appender.rolling.strategy.action.basepath = ${sys:es.logs.base_path} ---3
appender.rolling.strategy.action.condition.type = IfFileName ---4
appender.rolling.strategy.action.condition.glob = ${sys:es.logs.cluster_name}-* ---5
appender.rolling.strategy.action.condition.nested_condition.type = IfLastModified ---6
appender.rolling.strategy.action.condition.nested_condition.age = 7D ---7
```

1. DefaultRolloverStrategyの設定
2. ロールオーバーを処理するためのDeleteアクションを設定する
3. Elasticsearchログのベースパス
4. ロールオーバーを処理する際に適用する条件
5. ベースパスが ${sys:es.logs.cluster_name}-* というグロブにマッチするファイルを削除すること；これはログファイルがロールオーバーされるグロブである；ロールアウトした Elasticsearch ログのみを削除し、デプレッションやスローログは削除しないために必要となる
6. グロブに一致するファイルに適用するネストされた条件
7. ログを7日間保持する

log4j2.propertiesという名前で、Elasticsearchの設定ディレクトリを先祖に持っていれば、複数の設定ファイルを読み込むことができます（その場合、マージされます）; これは、追加のロガーを公開するプラグインに便利です。loggerセクションには、javaパッケージとそれに対応するログレベルが含まれています。appender セクションには、ログの出力先が含まれています。ロギングをカスタマイズする方法とサポートされているアペンダの詳細な情報は、Log4jのドキュメントで見つけることができます。

## Configuring logging levels
Elasticsearchのソースコードに含まれる各Javaパッケージには、関連するロガーが存在します。例えば、org.elasticsearch.discovery パッケージには、discovery プロセスに関連するログのための logger.org.elasticsearch.discovery があります。

冗長なログを取得するためには、クラスタ更新設定APIを使用して、関連するロガーのログレベルを変更します。各ロガーは、Log4j 2 のビルトインログレベルを、最も冗長でないものから最も冗長なものへと受け取ります。OFF、FATAL、ERROR、WARN、INFO、DEBUG、およびTRACEです。デフォルトのログレベルはINFOです。より高い冗長性レベル（DEBUGとTRACE）で記録されるメッセージは、専門家の使用のみを意図しています。

```
PUT /_cluster/settings
{
  "persistent": {
    "logger.org.elasticsearch.discovery": "DEBUG"
  }
}
```

その他、ログレベルを変更する方法は以下の通りです。

1. `elasticsearch.yml`:

```
logger.org.elasticsearch.discovery: DEBUG
```

単一ノードで問題をデバッグする場合に最適です。

2. `log4j2.properties`:

```
logger.discovery.name = org.elasticsearch.discovery
logger.discovery.level = debug
```

これは、他の理由ですでにLog4j 2の設定を変更する必要がある場合に最も適切です。例えば、特定のロガーのためのログを別のファイルに送りたいかもしれません。しかし、これらのユースケースはまれです。

## Deprecation logging
また、Elasticsearchはdeprecationログをlogディレクトリに書き込みます。これらのログには、非推奨のElasticsearchの機能を使用した際のメッセージが記録されています。Elasticsearchを新しいメジャーバージョンにアップグレードする前に、非推奨ログを使用してアプリケーションを更新することができます。

デフォルトでは、Elasticsearchは非推奨ログを1GBでロールバックし、圧縮しています。デフォルトの設定では、ロールされた4つのログとアクティブなログの最大5つのログファイルが保存されます。

ElasticsearchはCRITICALレベルのdeprecationログメッセージを出力します。このメッセージは、使用されている非推奨機能が次のメジャーバージョンで削除されることを示すものです。WARNレベルのDeprecationログメッセージは、それほど重要ではない機能が使用されていたことを示し、次のメジャーバージョンでは削除されないが、将来的に削除される可能性があることを示しています。

非推奨のログメッセージを書かないようにするには、log4j2.properties で logger.deprecation.level を OFF に設定します。

```
logger.deprecation.level = OFF
```

また、ロギングレベルを動的に変更することもできます。

```
PUT /_cluster/settings
{
  "persistent": {
    "logger.org.elasticsearch.deprecation": "OFF"
  }
}
```

ロギングレベルの設定」を参照してください。

HTTP ヘッダーとして X-Opaque-Id が使用されている場合、非推奨の機能をトリガーしているものを特定することができます。非推奨JSONログのX-Opaque-IDフィールドには、ユーザーIDが含まれます。

```json
{
  "type": "deprecation",
  "timestamp": "2019-08-30T12:07:07,126+02:00",
  "level": "WARN",
  "component": "o.e.d.r.a.a.i.RestCreateIndexAction",
  "cluster.name": "distribution_run",
  "node.name": "node-0",
  "message": "[types removal] Using include_type_name in create index requests is deprecated. The parameter will be removed in the next major version.",
  "x-opaque-id": "MY_USER_ID",
  "cluster.uuid": "Aq-c-PAeQiK3tfBYtig9Bw",
  "node.id": "D7fUYfnfTLa2D7y-xw6tZg"
}
```

Deprecationログは.logs-deprecation.elasticsearch-default data stream cluster.deprecation_indexing.enabled の設定を true にするとインデックス化できるようになります。

## Deprecation logs throttling
非推奨ログは、非推奨機能のキーと x-opaque-id に基づいて重複排除されるため、ある機能が繰り返し使用されても、非推奨ログに過剰な負荷をかけることはありません。これは、インデックス化された非推奨ログとログファイルに出力されるログの両方に適用されます。cluster.deprecation_indexing.x_opaque_id_used.enabled を false に変更することで、スロットリングにおける x-opaque-id の使用を無効にすることができます RateLimitingFilter を参照してください。

## JSON log format
Elasticsearchのログを簡単にパースするために、ログはJSON形式で出力されるようになりました。これは、Log4Jのレイアウトプロパティ appender.rolling.layout.type = ECSJsonLayoutによって設定されます。このレイアウトは、解析時にログストリームを区別するために使用されるデータセット属性の設定が必要です。

```log
appender.rolling.layout.type = ECSJsonLayout
appender.rolling.layout.dataset = elasticsearch.server
```

各行は、ECSJsonLayoutで設定されたプロパティを持つ1つのJSONドキュメントを含む。詳しくは、このクラスのjavadocを参照してください。ただし、JSON文書に例外が含まれる場合は、複数行に渡って表示されます。最初の行には通常のプロパティが含まれ、それ以降の行にはJSON配列としてフォーマットされたスタックトレースが含まれます。

あなた自身のカスタムレイアウトを使用することもできます。そのためには、appender.rolling.layout.typeの行を別のレイアウトに置き換えてください。以下のサンプルを参照してください。

```log
appender.rolling.type = RollingFile
appender.rolling.name = rolling
appender.rolling.fileName = ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}_server.log
appender.rolling.layout.type = PatternLayout
appender.rolling.layout.pattern = [%d{ISO8601}][%-5p][%-25c{1.}] [%node_name]%marker %.-10000m%n
appender.rolling.filePattern = ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}-%d{yyyy-MM-dd}-%i.log.gz
```
