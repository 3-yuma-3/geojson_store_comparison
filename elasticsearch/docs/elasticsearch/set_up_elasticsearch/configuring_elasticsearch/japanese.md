# Configuring Elasticsearch
Elasticsearchは優れたデフォルトで出荷されており、ほとんど設定を必要としません。ほとんどの設定は、Cluster update settings API を使って実行中のクラスタで変更することができます。

設定ファイルには、ノード固有の設定（node.name や paths など）、あるいはノードがクラスタに参加するために必要な設定（cluster.name や network.host など）を記述する必要があります。

## Config files location
Elasticsearchには3つの設定ファイルがあります。

- elasticsearch.yml Elasticsearchの設定を行います。
- jvm.options ElasticsearchのJVM設定を行います。
- log4j2.properties Elasticsearchのロギングを設定するためのファイルです。

これらのファイルは config ディレクトリに配置され、そのデフォルトの場所は、インストールがアーカイブディストリビューション（tar.gz または zip）かパッケージディストリビューション（Debian または RPM パッケージ）であるかどうかによって異なります。

アーカイブ・ディストリビューションの場合、config ディレクトリの場所はデフォルトで $ES_HOME/config になっています。コンフィグディレクトリの場所は、環境変数 ES_PATH_CONF を介して以下のように変更できます。

```
ES_PATH_CONF=/path/to/my/config ./bin/elasticsearch
```

また、コマンドラインやシェルプロファイルから ES_PATH_CONF 環境変数をエクスポートすることもできます。

パッケージ配布の場合、config ディレクトリの場所はデフォルトで /etc/elasticsearch になっています。設定ディレクトリの場所は ES_PATH_CONF 環境変数で変更することもできますが、シェルでこれを設定するだけでは十分でないことに注意してください。その代わり、この変数は /etc/default/elasticsearch (Debian パッケージ用) と /etc/sysconfig/elasticsearch (RPM パッケージ用)から取得されます。これらのファイルの ES_PATH_CONF=/etc/elasticsearch の項目を適宜編集して、設定ディレクトリの場所を変更する必要があります。

## Config file format
設定形式はYAMLです。ここでは、dataとlogsのディレクトリのパスを変更する例を示します。

```yml
path:
    data: /var/lib/elasticsearch
    logs: /var/log/elasticsearch
```

設定は以下のように平坦化することもできます。

```yml
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
```

YAMLでは、非スカラー値をシーケンスとしてフォーマットすることができます。

```yml
discovery.seed_hosts:
   - 192.168.1.10:9300
   - 192.168.1.11
   - seeds.mydomain.com
```

あまり一般的ではないが、スカラ値でない値を配列としてフォーマットすることも可能である。

```
discovery.seed_hosts: ["192.168.1.10:9300", "192.168.1.11", "seeds.mydomain.com"]
```

## Environment variable substitution
設定ファイル内で${...}の表記で参照される環境変数は、その環境変数の値に置き換えられます。例えば

```yml
node.name:    ${HOSTNAME}
network.host: ${ES_NETWORK_HOST}
```

環境変数の値は単純な文字列でなければなりません。Elasticsearchがリストとして解析する値を提供するには、カンマで区切られた文字列を使用してください。例えば、Elasticsearchは以下の文字列を環境変数${HOSTNAME}の値のリストとして分割します。

```
export HOSTNAME="host1,host2"
```

## Cluster and node setting types
クラスターとノードの設定は、その設定方法によって分類することができます。

### Dynamic
クラスタ更新設定APIを使用すると、実行中のクラスタで動的設定の構成と更新を行うことができます。また、elasticsearch.ymlを使用して、未起動またはシャットダウンしたノードでローカルに動的な設定を構成することもできます。

cluster update settings APIを使った更新は、クラスタの再起動に渡って適用されるpersistentと、クラスタの再起動後にリセットされるtransientがあります。また、APIを使用してNULL値を代入することで、transientまたはpersistentの設定をリセットすることができます。

複数の方法で同じ設定を行った場合、Elasticsearchは以下の優先順位で設定を適用します。

1. 一時的な設定
2. 永続的な設定
3. elasticsearch.ymlの設定
4. デフォルトの設定値

例えば、transient設定を適用することで、persistent設定やelasticsearch.ymlの設定を上書きすることができます。しかし、elasticsearch.ymlの設定を変更しても、定義されたtransientまたはpersistentの設定は上書きされません。

Elasticsearch Serviceを使用する場合、ユーザ設定機能を使用してすべてのクラスタ設定を行います。この方法では、クラスタを破壊する可能性のある安全でない設定を Elasticsearch Service が自動的に拒否することができます。

独自のハードウェアで Elasticsearch を実行する場合は、クラスタ更新設定 API を使用して動的なクラスタ設定を行います。静的なクラスタ設定とノード設定には elasticsearch.yml のみを使用してください。このAPIは再起動を必要とせず、設定の値がすべてのノードで同じであることを保証します。

一時的なクラスタ設定の使用は推奨されなくなりました。代わりにpersistent cluster settingsを使用してください。クラスタが不安定になると、transient設定が予期せずクリアされ、望ましくないクラスタ構成になる可能性があります。Transient settings migration ガイドを参照してください。

### Static
静的設定は、elasticsearch.ymlを使用して未起動またはシャットダウンのノードにのみ設定できます。

静的設定は、クラスタ内の関連するすべてのノードで設定する必要があります。
