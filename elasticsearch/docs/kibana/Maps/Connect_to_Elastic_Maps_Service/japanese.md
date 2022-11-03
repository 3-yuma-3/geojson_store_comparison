# Connect to Elastic Maps Service
Elastic Maps Service (EMS) は、行政境界のタイル レイヤーとベクトル図形をホストするサービスです。Kibana のアウトオブボックス設定を使用している場合、Maps はすでに EMS を使用するように設定されています。

EMS のリクエストは、以下のドメインに対して行われます。

- tiles.maps.elastic.co
- vector.maps.elastic.co。

Maps はブラウザから EMS に直接リクエストを行います。

## Disable Elastic Maps Service
Kibana サーバーまたはブラウザーがプライベート ネットワーク上またはファイアウォールの背後にある場合、EMS 接続の問題が発生する場合があります。このような場合、EMS 接続を無効にして、不要な EMS リクエストを回避することができます。

EMS を無効にするには、kibana.yml ファイルを変更します。

1. map.includeElasticMapsService を false に設定し、EMS 接続をオフにします。
2. map.tilemap.url に、タイルサーバーの URL を設定します。これにより、Mapsのデフォルトタイルレイヤーが設定されます。

## Host Elastic Maps Service locally
KibanaサーバやブラウザクライアントからElastic Maps Serviceに接続できず、クラスタが適切なライセンスレベルを持っている場合、独自のインフラでサービスをホストすることを選ぶことができます。

Elastic Maps Serverは、Elastic Maps Serviceのセルフマネージドバージョンで、EMSベースマップとEMSバウンダリの両方を提供するDockerイメージとして提供されます。イメージには、ズームレベル8までのベースマップがバンドルされています。Elasticsearchクラスタに接続してライセンスを確認した後、より詳細なベースマップ・データベースをダウンロードして設定するオプションがあります。

Elastic Maps Serverは、Vega、座標、地域地図の可視化で必要となるラスタータイルを提供しません。

Elastic Maps Serverのイメージは、docker pullを使用してElastic Dockerレジストリからダウンロードすることができます。

```
docker pull docker.elastic.co/elastic-maps-service/elastic-maps-server-ubi8:8.5.0
```

Elastic Maps Serverを起動し、デフォルトの8080番ポートを公開します。

```
docker run --rm --init --publish 8080:8080 \
  docker.elastic.co/elastic-maps-service/elastic-maps-server-ubi8:8.5.0
```

Elastic Maps Serverが起動したら、localhost:8080のウェブページの指示に従って、設定ファイルを定義し、オプションでより詳細なベースマップ・データベースをダウンロードします。

### Configuration
Elastic Maps Serverは、起動時に検証されるYAML形式のコンフィギュレーションファイルからプロパティを読み取ります。このファイルの場所はEMS_PATH_CONF環境変数によって提供され、デフォルトは/usr/src/app/server/config/elastic-maps-server.ymlとなります。

#### General settings
【TODO】

### Data
Elastic Maps Serverは、地球全体のベクターレイヤー境界線とベクタータイルベースマップをホストしています。境界には、世界の国、世界の行政区域、特定の国の地域が含まれます。ベースマップはズームレベル8までがDockerイメージにバンドルされています。これらのベースマップは、国レベルの地図やダッシュボードには十分です。より詳細な地図を表示するには、フロントページの指示に従って、適切なベースマップ・データベースをダウンロードし、設定してください。ズームレベル14の最も詳細なベースマップは、ストリートレベルの地図に適していますが、~90GBのディスク容量が必要です。

利用可能なベースマップと境界線は、https://maps.elastic.co と同等の自己管理型ウェブページの /maps エンドポイントから探索することができます。

### Kibana configuration
Elastic Maps Server が稼働している状態で、kibana.yml ファイルにサービスのルートを指す map.emsUrl 設定キーを追加します。この設定は、Kibana が Elastic Maps Server から EMS ベースマップと境界線を要求するように指し示します。通常、これは Elastic Maps Server のホストとポートへの URL になります。例えば、map.emsUrl: https://my-ems-server:8080。

### Status check
Elastic Maps Serverは定期的にステータスチェックを行い、3つの形式で公開されます。

- Elastic Maps Serverのルートでは、ウェブページに様々なサービスのステータスが表示されます。
- Elastic Maps ServerのステータスをJSONで表したものが、/statusエンドポイントにあります。
- Docker HEALTHCHECK命令はデフォルトで実行され、/statusエンドポイントに相当するプロセスを実行しながら、サービスの状態を通知します。

Elastic Maps Serverは、ライセンスバリデーションが満たされていない場合、どのようなデータ要求にも応答することはありません。

### Logging
ログはECSのJSON形式で生成され、標準出力と/var/log/elastic-maps-server/elastic-maps-server.logに出力されます。サーバは自動的にログをローテートしませんが、logrotate ツールがイメージにインストールされています。そのファイルへの出力を無効にしたい場合は、/dev/nullをデフォルトのログパスにマウントしてください。
