# Install Elasticsearch with Docker
Elasticsearch は Docker イメージとしても提供されています。公開されているDockerイメージとタグの一覧は www.docker.elastic.co にあります。 ソースファイルはGithubにあります。

本パッケージには、無料とサブスクリプションの両方の機能が含まれています。30日間のトライアルを開始し、すべての機能をお試しください。

Elasticsearch 8.0から、セキュリティがデフォルトで有効になっています。セキュリティを有効にした場合、Elastic Stackのセキュリティ機能では、トランスポートネットワーキング層にTLS暗号化を必要とし、そうでない場合はクラスタの起動に失敗します。

# Install Docker Desktop or Docker Engine
お使いのOSに適したDockerアプリケーションをインストールしてください。

Dockerに少なくとも4GiBのメモリが割り当てられていることを確認してください。Docker Desktopでは、Preference (macOS) または Settings (Windows) の Advanced タブでリソースの使用状況を設定します。

# Pull the Elasticsearch Docker image
Elasticsearch for Dockerの入手は、Elastic Dockerレジストリに対してdocker pullコマンドを発行することで簡単に行うことができます。

```
docker pull docker.elastic.co/elasticsearch/elasticsearch:8.5.0
```

これでElasticsearchのDockerイメージが手に入ったので、シングルノードまたはマルチノードのクラスタを起動することができます。

# Start a single-node cluster with Docker
DockerコンテナでシングルノードのElasticsearchクラスタを起動する場合、セキュリティは自動的に有効化され、設定されます。Elasticsearchの初回起動時には、以下のセキュリティ設定が自動的に行われます。

1. トランスポート層とHTTP層に対して証明書と鍵が生成されます。
2. TLS（Transport Layer Security）の構成設定がelasticsearch.ymlに書き込まれる。
3. エラスティックユーザーのパスワードが生成されます。
4. Kibana 用の登録トークンが生成されます。


その後、Kibanaを起動し、30分間有効な登録トークンを入力します。このトークンは、Elasticsearchクラスタのセキュリティ設定を自動的に適用し、kibana_systemユーザーでElasticsearchを認証し、セキュリティ設定をkibana.ymlに書き込む。

以下のコマンドは、開発またはテスト用にシングルノードのElasticsearchクラスターを起動します。

1. ElasticsearchとKibanaのための新しいdockerネットワークを作成します。

```
docker network create elastic
```

2. DockerでElasticsearchを起動します。elasticユーザー用のパスワードが生成されてターミナルに出力され、さらにKibanaを登録するための登録トークンが出力されます。

```
docker run --name es01 --net elastic -p 9200:9200 -p 9300:9300 -it docker.elastic.co/elasticsearch/elasticsearch:8.5.0
```

パスワードとエンロールメントトークンを表示するために、ターミナルを少しスクロールバックする必要があるかもしれません。

3. 生成されたパスワードと登録トークンをコピーして、安全な場所に保存してください。これらの値は、Elasticsearchの初回起動時にのみ表示されます。

エラスティックユーザーやその他の組み込みユーザーのパスワードをリセットする必要がある場合は、elasticsearch-reset-passwordツールを実行します。このツールは、DockerコンテナのElasticsearch /binディレクトリで利用可能です。例えば、以下のようになります。

```
docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-reset-password
```

4. Dockerコンテナからhttp_ca.crtセキュリティ証明書をローカルマシンにコピーしてください。

```
docker cp es01:/usr/share/elasticsearch/config/certs/http_ca.crt .
```

5. 新しいターミナルを開き、Dockerコンテナからコピーしたhttp_ca.crtファイルを使って、認証通話を行い、Elasticsearchクラスタに接続できることを確認します。プロンプトが表示されたら、elastic userのパスワードを入力します。

```
curl --cacert http_ca.crt -u elastic https://localhost:9200
```

# Enroll additional nodes
Elasticsearchを初めて起動する際、インストールプロセスではデフォルトでシングルノードクラスターが構成されます。また、このプロセスでは登録トークンが生成され、ターミナルに出力されます。既存のクラスタにノードを参加させたい場合は、生成された登録トークンを使って新しいノードを起動します。

Generating enrollment tokens
登録トークンの有効期限は30分です。新しい登録トークンを生成する必要がある場合、既存のノードで elasticsearch-create-enrollment-token ツールを実行します。このツールは、DockerコンテナのElasticsearchのbinディレクトリで利用可能です。

例えば、既存のes01ノードで以下のコマンドを実行し、新しいElasticsearchノード用の登録トークンを生成します。

```
docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node
```

1. 最初のノードを起動したターミナルで、生成されたElasticsearchの新規ノード追加用 enrollment tokenをコピーします。
2. 新しいノードで、Elasticsearchを起動し、生成された登録用トークンを含めます。

```
docker run -e ENROLLMENT_TOKEN="<token>" --name es02 --net elastic -it docker.elastic.co/elasticsearch/elasticsearch:8.5.0
```

これで、Elasticsearchが既存のクラスターに参加するように設定されました。

## Setting JVM heap size
2番目のノードの起動時に、1番目のノードが動作しているコンテナが終了する問題が発生する場合は、JVMヒープ・サイズに明示的に値を設定してください。ヒープサイズを手動で設定するには、各ノードの起動時に ES_JAVA_OPTS 変数をインクルードして、-Xms および -Xmx の値を設定します。例えば、以下のコマンドは、ノードes02を起動し、JVMヒープ・サイズの最小値と最大値を1GBに設定します。

```
docker run -e ES_JAVA_OPTS="-Xms1g -Xmx1g" -e ENROLLMENT_TOKEN="<token>" --name es02 -p 9201:9200 --net elastic -it docker.elastic.co/elasticsearch/elasticsearch:8.5.0
```

## Next steps
これでElasticsearchのテスト環境は整いました。Elasticsearchを使った本格的な開発や本番運用を始める前に、DockerでElasticsearchを本番運用する際に適用すべき要件や推奨事項を確認しましょう。

## Security certificates and keys
Elasticsearchをインストールすると、Elasticsearchの設定ディレクトリに以下の証明書と鍵が生成され、KibanaインスタンスをセキュアなElasticsearchクラスタに接続したり、インターノード通信を暗号化するために使用されます。参考までにファイルを列挙しておきます。

`http_ca.crt`
このElasticsearchクラスタのHTTPレイヤーの証明書に署名するために使用されるCA証明書です。

`http.p12`
このノードのHTTPレイヤーのキーと証明書を含むキーストアです。

`transport.p12`
クラスタ内の全ノードのトランスポート層のキーと証明書が格納されているキーストアです。

http.p12とtransport.p12はパスワードで保護されたPKCS#12キーストアです。Elasticsearchは、これらのキーストアのパスワードを安全な設定として保存します。パスワードを取得して、キーストアの内容を調査または変更できるようにするには、bin/elasticsearch-keystoreツールを使用します。

http.p12 のパスワードを取得するには、次のコマンドを使用します。

```
bin/elasticsearch-keystore show xpack.security.http.ssl.keystore.secure_password
```

transport.p12 のパスワードを取得するには、次のコマンドを使用します。

```
bin/elasticsearch-keystore show xpack.security.transport.ssl.keystore.secure_password
```

# Start a multi-node cluster with Docker Compose
セキュリティを有効にしたDockerで複数ノードのElasticsearchクラスタとKibanaを稼働させるには、Docker Composeを使用することができます。

この構成は、複数のホストによる分散デプロイメントを構築する前に、開発用に使用できるセキュアなクラスタを開始する簡単な方法を提供します。

## Prerequisites
お使いのオペレーティングシステムに適したDockerアプリケーションをインストールします。

Linuxであれば、Docker Composeをインストールします。

Dockerに少なくとも4GBのメモリが割り当てられていることを確認します。Docker Desktop では、Preferences (macOS) または Settings (Windows) の Advanced タブでリソースの使用状況を設定します。

## Prepare the environment
新しい空のディレクトリに、以下の設定ファイルを作成します。これらのファイルは、GitHubのelasticsearchリポジトリからも入手可能です。

`.env`
.envファイルは、docker-compose.yml設定ファイルを実行するときに使用する環境変数を設定します。ELASTIC_PASSWORDおよびKIBANA_PASSWORD変数を使用して、elasticおよびkibana_systemユーザーに強力なパスワードを指定することを確認します。これらの変数は、docker-compose.ymlファイルから参照されます。

パスワードは英数字でなければならず、! や @ などの特殊文字を含むことはできません。docker-compose.ymlファイルに含まれるbashスクリプトは、英数字に対してのみ動作します。

```shell
# Password for the 'elastic' user (at least 6 characters)
ELASTIC_PASSWORD=

# Password for the 'kibana_system' user (at least 6 characters)
KIBANA_PASSWORD=

# Version of Elastic products
STACK_VERSION=8.5.0

# Set the cluster name
CLUSTER_NAME=docker-cluster

# Set to 'basic' or 'trial' to automatically start the 30-day trial
LICENSE=basic
#LICENSE=trial

# Port to expose Elasticsearch HTTP API to the host
ES_PORT=9200
#ES_PORT=127.0.0.1:9200

# Port to expose Kibana to the host
KIBANA_PORT=5601
#KIBANA_PORT=80

# Increase or decrease based on the available host memory (in bytes)
MEM_LIMIT=1073741824

# Project namespace (defaults to the current folder name if not set)
#COMPOSE_PROJECT_NAME=myproject
```

`docker-compose.yml`
この docker-compose.yml ファイルは、認証とネットワーク暗号化を有効にした 3 ノードの安全な Elasticsearch クラスターと、そこに安全に接続された Kibana インスタンスを作成します。

Exposing ports
この構成では、すべてのネットワーク・インターフェイスでポート9200が公開されます。Dockerがポートを処理する仕組み上、localhostにバインドされていないポートはElasticsearchクラスタにパブリックにアクセス可能な状態になり、ファイアウォール設定を無視する可能性があります。ポート9200を外部ホストに公開したくない場合は、.envファイルのES_PORTの値を127.0.0.1:9200などの値に設定してください。そうすると、Elasticsearchはホストマシンからしかアクセスできなくなります。

```yml
version: "2.2"

services:
  setup:
    image: docker.elastic.co/elasticsearch/elasticsearch:${STACK_VERSION}
    volumes:
      - certs:/usr/share/elasticsearch/config/certs
    user: "0"
    command: >
      bash -c '
        if [ x${ELASTIC_PASSWORD} == x ]; then
          echo "Set the ELASTIC_PASSWORD environment variable in the .env file";
          exit 1;
        elif [ x${KIBANA_PASSWORD} == x ]; then
          echo "Set the KIBANA_PASSWORD environment variable in the .env file";
          exit 1;
        fi;
        if [ ! -f config/certs/ca.zip ]; then
          echo "Creating CA";
          bin/elasticsearch-certutil ca --silent --pem -out config/certs/ca.zip;
          unzip config/certs/ca.zip -d config/certs;
        fi;
        if [ ! -f config/certs/certs.zip ]; then
          echo "Creating certs";
          echo -ne \
          "instances:\n"\
          "  - name: es01\n"\
          "    dns:\n"\
          "      - es01\n"\
          "      - localhost\n"\
          "    ip:\n"\
          "      - 127.0.0.1\n"\
          "  - name: es02\n"\
          "    dns:\n"\
          "      - es02\n"\
          "      - localhost\n"\
          "    ip:\n"\
          "      - 127.0.0.1\n"\
          "  - name: es03\n"\
          "    dns:\n"\
          "      - es03\n"\
          "      - localhost\n"\
          "    ip:\n"\
          "      - 127.0.0.1\n"\
          > config/certs/instances.yml;
          bin/elasticsearch-certutil cert --silent --pem -out config/certs/certs.zip --in config/certs/instances.yml --ca-cert config/certs/ca/ca.crt --ca-key config/certs/ca/ca.key;
          unzip config/certs/certs.zip -d config/certs;
        fi;
        echo "Setting file permissions"
        chown -R root:root config/certs;
        find . -type d -exec chmod 750 \{\} \;;
        find . -type f -exec chmod 640 \{\} \;;
        echo "Waiting for Elasticsearch availability";
        until curl -s --cacert config/certs/ca/ca.crt https://es01:9200 | grep -q "missing authentication credentials"; do sleep 30; done;
        echo "Setting kibana_system password";
        until curl -s -X POST --cacert config/certs/ca/ca.crt -u "elastic:${ELASTIC_PASSWORD}" -H "Content-Type: application/json" https://es01:9200/_security/user/kibana_system/_password -d "{\"password\":\"${KIBANA_PASSWORD}\"}" | grep -q "^{}"; do sleep 10; done;
        echo "All done!";
      '
    healthcheck:
      test: ["CMD-SHELL", "[ -f config/certs/es01/es01.crt ]"]
      interval: 1s
      timeout: 5s
      retries: 120

  es01:
    depends_on:
      setup:
        condition: service_healthy
    image: docker.elastic.co/elasticsearch/elasticsearch:${STACK_VERSION}
    volumes:
      - certs:/usr/share/elasticsearch/config/certs
      - esdata01:/usr/share/elasticsearch/data
    ports:
      - ${ES_PORT}:9200
    environment:
      - node.name=es01
      - cluster.name=${CLUSTER_NAME}
      - cluster.initial_master_nodes=es01,es02,es03
      - discovery.seed_hosts=es02,es03
      - ELASTIC_PASSWORD=${ELASTIC_PASSWORD}
      - bootstrap.memory_lock=true
      - xpack.security.enabled=true
      - xpack.security.http.ssl.enabled=true
      - xpack.security.http.ssl.key=certs/es01/es01.key
      - xpack.security.http.ssl.certificate=certs/es01/es01.crt
      - xpack.security.http.ssl.certificate_authorities=certs/ca/ca.crt
      - xpack.security.http.ssl.verification_mode=certificate
      - xpack.security.transport.ssl.enabled=true
      - xpack.security.transport.ssl.key=certs/es01/es01.key
      - xpack.security.transport.ssl.certificate=certs/es01/es01.crt
      - xpack.security.transport.ssl.certificate_authorities=certs/ca/ca.crt
      - xpack.security.transport.ssl.verification_mode=certificate
      - xpack.license.self_generated.type=${LICENSE}
    mem_limit: ${MEM_LIMIT}
    ulimits:
      memlock:
        soft: -1
        hard: -1
    healthcheck:
      test:
        [
          "CMD-SHELL",
          "curl -s --cacert config/certs/ca/ca.crt https://localhost:9200 | grep -q 'missing authentication credentials'",
        ]
      interval: 10s
      timeout: 10s
      retries: 120

  es02:
    depends_on:
      - es01
    image: docker.elastic.co/elasticsearch/elasticsearch:${STACK_VERSION}
    volumes:
      - certs:/usr/share/elasticsearch/config/certs
      - esdata02:/usr/share/elasticsearch/data
    environment:
      - node.name=es02
      - cluster.name=${CLUSTER_NAME}
      - cluster.initial_master_nodes=es01,es02,es03
      - discovery.seed_hosts=es01,es03
      - bootstrap.memory_lock=true
      - xpack.security.enabled=true
      - xpack.security.http.ssl.enabled=true
      - xpack.security.http.ssl.key=certs/es02/es02.key
      - xpack.security.http.ssl.certificate=certs/es02/es02.crt
      - xpack.security.http.ssl.certificate_authorities=certs/ca/ca.crt
      - xpack.security.http.ssl.verification_mode=certificate
      - xpack.security.transport.ssl.enabled=true
      - xpack.security.transport.ssl.key=certs/es02/es02.key
      - xpack.security.transport.ssl.certificate=certs/es02/es02.crt
      - xpack.security.transport.ssl.certificate_authorities=certs/ca/ca.crt
      - xpack.security.transport.ssl.verification_mode=certificate
      - xpack.license.self_generated.type=${LICENSE}
    mem_limit: ${MEM_LIMIT}
    ulimits:
      memlock:
        soft: -1
        hard: -1
    healthcheck:
      test:
        [
          "CMD-SHELL",
          "curl -s --cacert config/certs/ca/ca.crt https://localhost:9200 | grep -q 'missing authentication credentials'",
        ]
      interval: 10s
      timeout: 10s
      retries: 120

  es03:
    depends_on:
      - es02
    image: docker.elastic.co/elasticsearch/elasticsearch:${STACK_VERSION}
    volumes:
      - certs:/usr/share/elasticsearch/config/certs
      - esdata03:/usr/share/elasticsearch/data
    environment:
      - node.name=es03
      - cluster.name=${CLUSTER_NAME}
      - cluster.initial_master_nodes=es01,es02,es03
      - discovery.seed_hosts=es01,es02
      - bootstrap.memory_lock=true
      - xpack.security.enabled=true
      - xpack.security.http.ssl.enabled=true
      - xpack.security.http.ssl.key=certs/es03/es03.key
      - xpack.security.http.ssl.certificate=certs/es03/es03.crt
      - xpack.security.http.ssl.certificate_authorities=certs/ca/ca.crt
      - xpack.security.http.ssl.verification_mode=certificate
      - xpack.security.transport.ssl.enabled=true
      - xpack.security.transport.ssl.key=certs/es03/es03.key
      - xpack.security.transport.ssl.certificate=certs/es03/es03.crt
      - xpack.security.transport.ssl.certificate_authorities=certs/ca/ca.crt
      - xpack.security.transport.ssl.verification_mode=certificate
      - xpack.license.self_generated.type=${LICENSE}
    mem_limit: ${MEM_LIMIT}
    ulimits:
      memlock:
        soft: -1
        hard: -1
    healthcheck:
      test:
        [
          "CMD-SHELL",
          "curl -s --cacert config/certs/ca/ca.crt https://localhost:9200 | grep -q 'missing authentication credentials'",
        ]
      interval: 10s
      timeout: 10s
      retries: 120

  kibana:
    depends_on:
      es01:
        condition: service_healthy
      es02:
        condition: service_healthy
      es03:
        condition: service_healthy
    image: docker.elastic.co/kibana/kibana:${STACK_VERSION}
    volumes:
      - certs:/usr/share/kibana/config/certs
      - kibanadata:/usr/share/kibana/data
    ports:
      - ${KIBANA_PORT}:5601
    environment:
      - SERVERNAME=kibana
      - ELASTICSEARCH_HOSTS=https://es01:9200
      - ELASTICSEARCH_USERNAME=kibana_system
      - ELASTICSEARCH_PASSWORD=${KIBANA_PASSWORD}
      - ELASTICSEARCH_SSL_CERTIFICATEAUTHORITIES=config/certs/ca/ca.crt
    mem_limit: ${MEM_LIMIT}
    healthcheck:
      test:
        [
          "CMD-SHELL",
          "curl -s -I http://localhost:5601 | grep -q 'HTTP/1.1 302 Found'",
        ]
      interval: 10s
      timeout: 10s
      retries: 120

volumes:
  certs:
    driver: local
  esdata01:
    driver: local
  esdata02:
    driver: local
  esdata03:
    driver: local
  kibanadata:
    driver: local
```

## Start your cluster with sercurity enabled and configured
1. .envファイルを修正し、ELASTIC_PASSWORDとKIBANA_PASSWORDの両変数に強力なパスワードの値を入力します。

クラスタとさらにやり取りする場合は、ELASTIC_PASSWORD 値を使用する必要があります。KIBANA_PASSWORD 値は、Kibana を構成するときにのみ内部的に使用されます。

2. 3ノードのElasticsearchクラスタとKibanaインスタンスを作成し、起動します。

```
docker-compose up -d
```

3. デプロイが開始されたら、ブラウザを開いて http://localhost:5601 に移動し、Kibana にアクセスします。Kibana ではサンプルデータをロードしてクラスターと対話することができます。

## Stop and remove the deployment
クラスターを停止するには、docker-compose downを実行します。Dockerボリューム内のデータは保存され、docker-compose upでクラスタを再起動したときに読み込まれます。

```
docker-compose down
```

クラスター停止時にネットワーク、コンテナ、およびボリュームを削除するには、-vオプションを指定します。

```
docker-compose down -v
```

## Next steps
これでElasticsearchのテスト環境は整いました。Elasticsearchを使った本格的な開発や本番運用を始める前に、DockerでElasticsearchを本番運用する際に適用すべき要件や推奨事項を確認しましょう。

# Using the Docker images in production
DockerでElasticsearchを実運用する場合、以下の要件と推奨事項が適用されます。

## Set `vm.max_map_count` to at least `262144`
vm.max_map_count カーネル設定は、実稼働環境では最低でも 262144 に設定する必要があります。

vm.max_map_count をどのように設定するかは、お使いのプラットフォームによって異なります。

### Linux
vm.max_map_countの設定の現在の値を表示するには、以下を実行します。

```
grep vm.max_map_count /etc/sysctl.conf
vm.max_map_count=262144
```

ライブシステムで設定を適用するには、以下を実行します。

```
sysctl -w vm.max_map_count=262144
```

vm.max_map_count設定の値を永続的に変更するには、/etc/sysctl.confでその値を更新します。

### macOS with DOcker for Mac
vm.max_map_count 設定は、xhyve 仮想マシン内で設定する必要があります。

1. コマンド ラインから、実行します。

```
screen ~/Library/Containers/com.docker.docker/Data/vms/0/tty
```

2. Enterキーを押して、sysctlでvm.max_map_countを設定します。

```
sysctl -w vm.max_map_count=262144
```

3. 画面セッションを終了するには、Ctrl a d と入力します。

### Windows And macOS with DOcker Desktop
vm.max_map_count は docker-machine で設定する必要があります。

```
docker-machine ssh
sudo sysctl -w vm.max_map_count=262144
```

### Windows with Docker Desktop WSL 2 backend
vm.max_map_count の設定は、docker-desktop コンテナで設定する必要があります。

```
wsl -d docker-desktop
sysctl -w vm.max_map_count=262144
```

## Configuration files must be readable by the `elasticsearch` user
デフォルトでは、Elasticsearchはコンテナ内でユーザーelasticsearchとしてuid:gid 1000:0を使用して実行されます。

例外はOpenshiftで、任意に割り当てられたユーザーIDを使用してコンテナを実行します。Openshiftではgidが0に設定された永続ボリュームが提示されるが、これは何の調整もなく動作する。

ローカルのディレクトリやファイルをバインドマウントする場合、elasticsearchユーザーが読み取り可能である必要があります。さらに、このユーザはconfig、data、logディレクトリへの書き込み権限が必要です（Elasticsearchは鍵ストアを生成するため、configディレクトリへの書き込み権限が必要です）。良い方法は、ローカルディレクトリに対してgid 0のグループアクセス権を付与することです。

例えば、bind-mountでデータを格納するためのローカルディレクトリを用意する場合。

```
mkdir esdatadir
chmod g+rwx esdatadir
chgrp 0 esdatadir
```

また、カスタムUIDとカスタムGIDの両方を使用してElasticsearchコンテナを実行することも可能です。その際、ファイルパーミッションによってElasticsearchの実行が妨げられないようにする必要があります。以下の2つのオプションのいずれかを使用することができます。

- config、data、logsの各ディレクトリをバインドマウントする。プラグインをインストールするつもりで、カスタムDockerイメージを作成しない方が良い場合は、pluginsディレクトリもバインドマウントする必要があります。
- docker runに--group-add 0コマンドラインオプションを渡します。これにより、Elasticsearchを実行しているユーザが、コンテナ内のルート（GID 0）グループのメンバであることが保証されます。

## Increase ulimits for nofile and nproc
Elasticsearch コンテナで nofile と nproc の増加した ulimits が利用可能である必要があります。Docker デーモンの init システムが、これらを許容範囲内の値に設定していることを確認します。

Docker デーモンのデフォルトの ulimits を確認するには、以下を実行します。

```
docker run --rm docker.elastic.co/elasticsearch/elasticsearch:{version} /bin/bash -c 'ulimit -Hn && ulimit -Sn && ulimit -Hu && ulimit -Su'
```

必要に応じて、Daemon で調整するか、コンテナ単位で上書きしてください。例えば、docker runを使用する場合、設定します。

```
--ulimit nofile=65535:65535
```

## Disable swapping
パフォーマンスとノードの安定性のために、スワッピングを無効にする必要があります。これを行う方法については、スワッピングを無効にするを参照してください。

bootstrap.memory_lock: true のアプローチを選択した場合、Docker Daemon で memlock: true ulimit を定義するか、サンプルの compose ファイルに示すように、コンテナに明示的に設定する必要があります。docker runを使用する場合は、指定することができます。

```
-e "bootstrap.memory_lock=true" --ulimit memlock=-1:-1
```

## Randomize published ports
イメージは、TCP ポート 9200 と 9300 を公開しています。実運用クラスタでは、ホストごとに 1 つのコンテナを固定する場合を除き、--publish-all を使用して公開ポートをランダムにすることをお勧めします。

## Manually set the heap size
デフォルトでは、Elasticsearchは、ノードのロールとノードのコンテナで利用可能な総メモリに基づいて、自動的にJVMヒープをサイズします。ほとんどの実稼働環境では、このデフォルトのサイジングを推奨します。必要であれば、JVMヒープサイズを手動で設定することにより、デフォルトのサイジングをオーバーライドすることができます。

実稼働環境でヒープサイズを手動で設定するには、/usr/share/elasticsearch/config/jvm.options.d以下に、希望するヒープサイズ設定を含むJVMオプションファイルをバインドマウントしてください。

テスト用には、環境変数 ES_JAVA_OPTS を使用してヒープサイズを手動で設定することもできます。例えば、16GBを使用するには、-eを指定します。
ES_JAVA_OPTS="-Xms16g -Xmx16g "と指定して、docker runを実行します。ES_JAVA_OPTS変数は、他のすべてのJVMオプションより優先されます。ES_JAVA_OPTSを実稼働環境で使用することはお勧めしません。上記のdocker-compose.ymlファイルでは、ヒープサイズを512MBに設定しています。

## Pin deployments to a specific image version
ElasticsearchのDockerイメージの特定のバージョンにデプロイメントを固定します。例えば docker.elastic.co/elasticsearch/elasticsearch:8.5.0.

## Always bind data volumes
以下の理由から、/usr/share/elasticsearch/data にバインドされたボリュームを使用する必要があります。

1. コンテナが強制終了しても、Elasticsearchノードのデータは失われません。
2. ElasticsearchはI/Oに敏感であり、Dockerストレージドライバは高速I/Oには適していません。
3. 高度なDockerボリュームプラグインを使用することができます。

## Avoid using loop-lvm mode
devicemapper ストレージドライバを使用する場合、デフォルトの loop-lvm モードは使用しないでく ださい。direct-lvmを使用するようにdocker-engineを設定します。

## Centralize your logs
別のロギングドライバを使用して、ログを一元化することを検討してください。また、デフォルトの json-file ロギングドライバは、実稼働環境での使用には適していないことに注意してください。

# Configuring Elasticsearch with Docker
Dockerで実行する場合、Elasticsearchの設定ファイルは/usr/share/elasticsearch/config/から読み込まれます。

カスタム設定ファイルを使用する場合は、イメージ内の設定ファイルの上にファイルをバインドマウントします。

Elasticsearchの設定パラメータは、Docker環境変数を使って個別に設定することができます。コンポジットファイルのサンプルとシングルノードのサンプルはこの方法を採用しています。設定名をそのまま環境変数名として使用することができます。オーケストレーションプラットフォームが環境変数名のピリオドを禁止しているなどの理由でこれができない場合は、以下のように設定名を変換して別のスタイルを使用することができます。

1. 設定名を大文字に変更する
2. ES_SETTING_を先頭に付ける。
3. アンダースコア(_)を重複してエスケープする。
4. ピリオド(.)をすべてアンダースコア(_)に変換する

例えば、-e bootstrap.memory_lock=true は -e ES_SETTING_BOOTSTRAP_MEMORY__LOCK=true となります。

環境変数名に_FILEを付けると、ELASTIC_PASSWORDまたはKEYSTORE_PASSWORD環境変数の値を設定するためにファイルの内容を使用することができます。これは、パスワードなどの秘密を直接指定せずにElasticsearchに渡す場合に便利です。

例えば、Elasticsearchの起動パスワードをファイルから設定する場合、ファイルをマウントし、そのマウント先に環境変数ELASTIC_PASSWORD_FILEを設定することでバインドすることが可能です。パスワードファイルを/run/secrets/bootstrapPassword.txtにマウントする場合、指定します。

```
-e ELASTIC_PASSWORD_FILE=/run/secrets/bootstrapPassword.txt
```

イメージのデフォルトコマンドをオーバーライドして、Elasticsearchの設定パラメータをコマンドラインオプションとして渡すことができます。例えば

```
docker run <various parameters> bin/elasticsearch -Ecluster.name=mynewclustername
```

通常、実運用では設定ファイルをバインドマウントする方法が望ましいですが、設定を含むカスタムDockerイメージを作成することもできます。

## Mounting Elasticsearch configuration files
カスタム設定ファイルを作成し、Dockerイメージの対応するファイル上にバインドマウントします。例えば、custom_elasticsearch.yml を docker run で bind-mount する場合は、次のように指定します。

```
-v full_path_to/custom_elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
```

カスタム elasticsearch.yml ファイルをバインドマウントする場合、network.host: 0.0.0.0 の設定が含まれていることを確認してください。この設定は、そのポートが公開されている場合に、ノードがHTTPおよびトランスポートトラフィックに対して到達可能であることを保証します。Dockerイメージに組み込まれたelasticsearch.ymlファイルは、デフォルトでこの設定を含んでいます。

コンテナは、ユーザーelasticsearchとして、uid:gid 1000:0を使用してElasticsearchを実行します。バインドマウントされたホストのディレクトリやファイルはこのユーザがアクセスでき、データおよびログディレクトリはこのユーザが書き込み可能である必要があります。

## Create an encrypted Elasticsearch keystore
デフォルトでは、Elasticsearchはセキュアな設定のためにキーストアファイルを自動生成します。このファイルは難読化されていますが、暗号化されてはいません。

セキュアな設定をパスワードで暗号化し、コンテナ外で永続化させるには、代わりにdocker runコマンドを使用して手動でキーストアを作成します。このコマンドは次のようにしなければなりません。

- configディレクトリをバインドマウントする。このコマンドは、このディレクトリにelasticsearch.keystoreファイルを作成します。エラーを避けるため、elasticsearch.keystore ファイルを直接 bind-mount しないようにしてください。
- elasticsearch-keystoreツールをcreate -pオプションで使用します。keystoreのパスワードを入力するプロンプトが表示されます。

例えば、以下のようになります。

```
docker run -it --rm \
-v full_path_to/config:/usr/share/elasticsearch/config \
docker.elastic.co/elasticsearch/elasticsearch:8.5.0 \
bin/elasticsearch-keystore create -p
```

また、docker runコマンドを使用して、キーストアにセキュアな設定を追加または更新することもできます。設定値を入力するプロンプトが表示されます。鍵ストアが暗号化されている場合は、鍵ストアのパスワードを入力するプロンプトも表示されます。

```
docker run -it --rm \
-v full_path_to/config:/usr/share/elasticsearch/config \
docker.elastic.co/elasticsearch/elasticsearch:8.5.0 \
bin/elasticsearch-keystore \
add my.secure.setting \
my.other.secure.setting
```

すでにキーストアを作成しており、更新する必要がない場合は、elasticsearch.keystore ファイルを直接バインドマウントすることができます。環境変数KEYSTORE_PASSWORDを使用すると、起動時にコンテナにキーストアパスワードを提供することができます。例えば、docker runコマンドでは、以下のようなオプションがあります。

```
-v full_path_to/config/elasticsearch.keystore:/usr/share/elasticsearch/config/elasticsearch.keystore
-e KEYSTORE_PASSWORD=mypassword
```

## Using custom Docker images
環境によっては、設定を含むカスタムイメージを用意する方が合理的な場合もあります。これを実現するためのDockerfileは、以下のようなシンプルなものでしょう。

```
FROM docker.elastic.co/elasticsearch/elasticsearch:8.5.0
COPY --chown=elasticsearch:elasticsearch elasticsearch.yml /usr/share/elasticsearch/config/
```

でイメージを構築し、実行することができます。

```
docker build --tag=elasticsearch-custom .
docker run -ti -v /usr/share/elasticsearch/data elasticsearch-custom
```

一部のプラグインは、追加のセキュリティパーミッションを必要とします。以下のいずれかの方法で明示的に許可する必要があります。

- Docker イメージを実行するときに tty をアタッチし、プロンプトが表示されたときにパーミッションを許可する。
- プラグインインストールコマンドに --batch フラグを追加して、セキュリティパーミッションを検査し、（適切であれば）それを許可する。

詳しくは、プラグイン管理を参照してください。

## Troubleshoot Docker errors for Elasticsearch
ここでは、DockerでElasticsearchを実行する際によくあるエラーを解決する方法を紹介します。

## elasticsearch.keystore is a directory

```
Exception in thread "main" org.elasticsearch.bootstrap.BootstrapException: java.io.IOException: Is a directory: SimpleFSIndexInput(path="/usr/share/elasticsearch/config/elasticsearch.keystore") Likely root cause: java.io.IOException: Is a directory
```

keystore関連のdocker runコマンドが、存在しないelasticsearch.keystoreファイルを直接bind-mountしようとしました。存在しないファイルをマウントするために -v または --volume フラグを使用した場合、Docker は代わりに同じ名前のディレクトリを作成します。

このエラーを解決するには

1. configディレクトリ内のelasticsearch.keystoreディレクトリを削除してください。
2. vまたは--volumeフラグを更新して、keystoreファイルのパスではなく、configディレクトリのパスを指すようにします。例として、暗号化されたElasticsearchの鍵ストアを作成するを参照してください。
3. コマンドを再試行してください。

## elasticsearch.keystore: Device or resource busy

```
Exception in thread "main" java.nio.file.FileSystemException: /usr/share/elasticsearch/config/elasticsearch.keystore.tmp -> /usr/share/elasticsearch/config/elasticsearch.keystore: Device or resource busy
```

docker run コマンドが elasticsearch.keystore ファイルを直接バインドマウントしながら keystore を更新しようとしました。keystore を更新するために、コンテナは keystore.tmp などの config ディレクトリ内の他のファイルへのアクセスを必要とします。

このエラーを解決するには

1. vまたは--volumeフラグを更新して、keystoreファイルのパスではなく、configディレクトリのパスを指すようにします。例として、暗号化されたElasticsearchの鍵ストアを作成するを参照してください。
2. コマンドを再試行してください。
