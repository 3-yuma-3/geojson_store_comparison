# Install Kibana with Docker
Kibana用のDockerイメージは、Elastic Dockerレジストリから入手可能です。ベースイメージはubuntu:20.04です。

公開されているすべてのDockerイメージとタグの一覧は、www.docker.elastic.co。 ソースコードはGitHubにあります。

これらのイメージには、無料とサブスクリプションの両方の機能が含まれています。30日間のトライアルを開始すると、すべての機能を試すことができます。


## Run Kibana on Docker for development
1. 開発・テスト用のElasticsearchコンテナを起動します。

```
docker network create elastic
docker pull docker.elastic.co/elasticsearch/elasticsearch:8.5.0
docker run --name es-node01 --net elastic -p 9200:9200 -p 9300:9300 -t docker.elastic.co/elasticsearch/elasticsearch:8.5.0
```

Elasticsearchの初回起動時には、以下のセキュリティ設定が自動的に行われます。

- トランスポート層とHTTP層に対して証明書と鍵が生成される。
- TLS（Transport Layer Security）の構成設定がelasticsearch.ymlに書き込まれる。
- エラスティックユーザーのパスワードが生成されます。
- Kibana 用の登録トークンが生成されます。

パスワードとエンロールメントトークンを表示するには、ターミナルを少しスクロールバックする必要があるかもしれません。

2. 生成されたパスワードと登録トークンをコピーして、安全な場所に保存してください。これらの値は、Elasticsearchの初回起動時のみ表示されます。これらを使って、KibanaをElasticsearchクラスタに登録し、ログインします。
3. 新しいターミナル・セッションで、Kibana を起動し、Elasticsearch コンテナに接続します。

```
docker pull docker.elastic.co/kibana/kibana:8.5.0
docker run --name kib-01 --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:8.5.0
```

Kibanaを起動すると、端末に固有のリンクが出力されます。

4. Kibanaにアクセスするには、ターミナルで生成されたリンクをクリックします。
  a. ブラウザで、Elasticsearchの起動時にコピーしたenrollment tokenを貼り付け、ボタンをクリックしてKibanaインスタンスとElasticsearchを接続します。
  b. Elasticsearchの起動時に生成されたパスワードでelasticユーザーとしてKibanaにログインします。

## Generate passwords and enrollment tokens
elasticユーザーやその他の組み込みユーザーのパスワードをリセットする必要がある場合は、elasticsearch-reset-passwordツールを実行します。このツールは、DockerコンテナのElasticsearchのbinディレクトリで利用可能です。

例えば、エラスティックユーザーのパスワードをリセットする場合。

```
docker exec -it es-node01 /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
```

Kibana または Elasticsearch ノード用に新しい登録トークンを生成する必要がある場合、elasticsearch-create-enrollment-token ツールを実行してください。このツールは、DockerコンテナのElasticsearchのbinディレクトリで利用可能です。

例えば、Kibana 用の新しい登録トークンを生成する場合。

```
docker exec -it es-node01 /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
```

## Remove Docker containers
コンテナとそのネットワークを削除するには、以下を実行します。

```
docker network rm elastic
docker rm es-node01
docker rm kib-01
```

## Configure Kibana on Docker
Dockerイメージでは、Kibanaを設定する方法がいくつか用意されています。従来は「Kibanaの設定」で説明したkibana.ymlファイルを用意する方法が一般的でしたが、環境変数で設定を定義することも可能です。

### Bind-mounted configuration
Docker上でKibanaを設定する方法として、kibana.ymlをbind-mountingで提供する方法がある。docker-composeでは、このようにbind-mountを指定することができます。

```yml
version: '2'
services:
  kibana:
    image: docker.elastic.co/kibana/kibana:8.5.0
    volumes:
      - ./kibana.yml:/usr/share/kibana/config/kibana.yml
```

## Persist the Kibana keystore
デフォルトでは、Kibana は起動時にセキュア設定のための keystore ファイルを自動生成します。セキュアな設定を永続化するには、kibana-keystore ユーティリティを使用して、キーストアの親ディレクトリをコンテナにバインドマウントします。たとえば、次のようになります。

```
docker run -it --rm -v full_path_to/config:/usr/share/kibana/config -v full_path_to/data:/usr/share/kibana/data docker.elastic.co/kibana/kibana:8.5.0 bin/kibana-keystore create
docker run -it --rm -v full_path_to/config:/usr/share/kibana/config -v full_path_to/data:/usr/share/kibana/data docker.elastic.co/kibana/kibana:8.5.0 bin/kibana-keystore add test_keystore_setting
```

### Environment variable configuration
Docker環境では、Kibanaを環境変数で設定することができる。コンテナの起動時にヘルパープロセスが環境をチェックし、Kibanaのコマンドライン引数にマッピング可能な変数を探します。

コンテナオーケストレーションシステムとの互換性を保つため、これらの環境変数はすべて大文字で記述し、アンダースコアを単語の区切り文字として使用します。ヘルパーは、これらの名前を有効な Kibana 設定名に変換します。

環境変数に含めるすべての情報は、機密情報を含め、psコマンドで見ることができます。

翻訳例をいくつかご紹介します。

**Table 1. Example Docker Environment Variables**

| Environment Variable | Kibana Setting |
|-|-|
| SERVER_NAME | server.name |
| SERVER_BASEPATH | server.basePath |
| ELASTICSEARCH_HOSTS | elasticsearch.hosts |

一般に、Configure Kibana に記載されている設定はすべてこの手法で設定することができます。

配列オプションを提供することは、厄介なことがあります。次の例は、ELASTICSEARCH_HOSTSに配列を指定する構文です。

これらの変数は、このようにdocker-composeで設定することができます。

```yml
version: '2'
services:
  kibana:
    image: docker.elastic.co/kibana/kibana:8.5.0
    environment:
      SERVER_NAME: kibana.example.org
      ELASTICSEARCH_HOSTS: '["http://es01:9200","http://es02:9200","http://es03:9200"]'
```

環境変数はCLI引数に変換されるため、kibana.ymlで設定した内容よりも優先されます。

### Docker defaults
以下の設定は、Docker イメージを使用する場合、デフォルト値が異なります。

|||
|-|-|
| server.host | "0.0.0.0" |
| server.shutdownTimeout | "5s" |
| elasticsearch.hosts | http://elasticsearch:9200 |
| monitoring.ui.container.elasticsearch.enabled | true |

これらの設定は、デフォルトの kibana.yml で定義されています。これらは、カスタムの kibana.yml または環境変数で上書きすることができます。

kibana.yml をカスタムバージョンに置き換える場合、デフォルトを保持したい場合は、必ずカスタムファイルにコピーしてください。そうしないと、新しいファイルによって「マスク」されてしまいます。
