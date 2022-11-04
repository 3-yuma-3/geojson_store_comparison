# Ingest geospatial data into Elasticsearch with GDAL (2019/11/21)
KibanaのElastic Mapsはもう使いましたか？私はマルチレイヤーのサポートにとても興奮しています。ヒートマップ、Elastic Maps Serviceからのベクターレイヤー、そして個別のドキュメントまで、すべて同じインターフェイスで利用できるのです。データを分析し、可視化するための素晴らしい方法です。

しかし、Elasticsearchにない地理空間データについてはどうでしょうか？例えば、販売地域のシェープファイルと販売集計を重ね合わせたいとします。また、配送センターの位置を示すCSVファイルを持っていて、このデータをElasticsearchに取り込みたいが、FilebeatやLogstashの設定は静的なデータセットを取り込むには理想的でない、といったことも考えられます。そんなあなたにぴったりのソリューションがあります。GDALです。

GDAL (Geospatial Data Abstraction Library) には、Elasticsearchを含む75以上の異なる地理空間ファイルフォーマット間で地理空間データを変換できるコマンドラインツールが含まれています。GDALは、ソースからコンパイルするか、パッケージマネージャ経由でインストールすることができます。GDALは、MacのHomebrew OSGeo経由でインストールすることもできます（ex. brew tap osgeo/osgeo4mac && brew install osgeo-gdal）. なお、Elasticsearch 7.xにデータを取り込むには、GDAL v3.1以降が必要です。

## Connecting to Elasticsearch
GDALをインストールしたら、コマンドラインまたはターミナルウィンドウを開き、ogrinfoツールを使ってElasticsearchクラスタに接続してみてください。GDALにElasticsearchドライバを使用するように伝えるため、URLの前に "ES: "を付けています。

```bash
ogrinfo ES:http://localhost:9200
```

Elasticsearchクラスタでセキュリティを有効にしている場合（よかったですね）、IPまたはドメインの前にuser:pass@の形式でユーザ名とパスワードを追加してください。例えば

```bash
ogrinfo ES:https://elastic:mysecretpassword@example.com:9243
```

このコマンドは、Elasticsearchクラスタ内のインデックスとジオメトリタイプ（ある場合）のリストを表示します。

また、HOMEディレクトリにある.netrcファイルに資格情報を保存することができます。GDALはElasticsearchへの接続にcurlを使用するため、このファイルから自動的に認証情報を取得します。上記の例で言うと、私の.netrcファイルは以下のようになります。

```bash
machine example.com
login elastic
password mysecretpassword
```

そして、私のGDALコマンドは次のようになります。

```bash
ogrinfo ES:https://example.com:9243
```

## Ingesting a shapefile into Elasticsearch
Elasticsearchに接続できることが確認できたので、シェイプファイルの取り込みを試してみましょう。シェイプファイルとは、図形とプロパティを含むバイナリ地理空間ファイルフォーマットです。Elasticsearchにシェイプファイルを取り込むには、ogr2ogrを使うのが一番手っ取り早いです。

```bash
ogr2ogr ES:http://localhost:9200 my_shapefile.shp
```

しかし、GDALが設定したデフォルトは、私たちのデータにとって理想的とは言えないかもしれません。例えば、GDALは、全文検索インデックスのために、Elasticsearchですべてのテキストフィールドを「text」としてマップすることを想定しています。通常、私はテキストフィールドを「keyword」としてマッピングし、条件集計に使用できるようにしたい。そこで、コマンドラインに -lco NOT_ANALYZED_FIELDS={ALL} フラグを追加して、すべてのテキストフィールドを "keyword" にマップするようにします。ALL}の代わりにカンマで区切られたフィールドのリストを指定することもできますね。

また、Elasticsearchに取り込むために、独自のマッピングファイルを指定することもできる。幸いなことに、GDALがマッピングファイルを生成してくれるので、手作業で行う必要はない。ここでは、ogr2ogrツールを使って、マッピングを生成するためのパラメータを追加してみます。これはマッピングファイルを生成するだけで、Elasticsearchへのデータ取り込みは行いません。

```bash
ogr2ogr -lco INDEX_NAME=gdal-data -lco NOT_ANALYZED_FIELDS={ALL} -lco WRITE_MAPPING=./mapping.json ES:http://localhost:9200 my_shapefile.shp
```

テキストエディターでmapping.jsonファイルを編集して、いくつかの設定を微調整することができます。その後、カスタマイズしたマッピングへのパスを指定することで、シェイプファイルのインジェスト時にカスタマイズしたマッピングを使用することができます。

```bash
ogr2ogr -lco INDEX_NAME=gdal-data -lco OVERWRITE_INDEX=YES -lco MAPPING=./mapping.json ES:http://localhost:9200 my_shapefile.shp
```

## GDAL and mapping types
GDALにはMAPPING_NAMEパラメータがあり、マッピングタイプをサポートするElasticsearchの以前のバージョンで指定することができます。Elasticsearch 6 以前のバージョンを使用している場合、デフォルトのマッピングタイプは "FeatureCollection" で、各ドキュメントは "properties" というネストされたフィールドの下にその属性を持つことになります。現在、Kibanaはネストされたフィールドをサポートしていません。そのため、MAPPING_NAMEパラメータをネストされたフィールドを使用しない「doc」など他のものに変更することを強くお勧めします。Elasticsearch 7 以降ではマッピングタイプが削除されたため、MAPPING_NAME パラメータは影響を与えません。

```bash
ogr2ogr -lco MAPPING_NAME=doc -lco INDEX_NAME=gdal-data -lco MAPPING=./mapping.json ES:http://localhost:9200 my_shapefile.shp
```

## More resources
GDALは非常に強力なツールであり、使用するコマンドラインオプションを正確に見つけるには、ドライバのドキュメントを読み、多くのテストを行う必要があります。ogr2ogrのドキュメントを注意深く読むようにしてください。また、適切な入力と出力ドライバのドキュメントを確認し、それぞれ独自の設定オプションを持っています。

BostonGIS.comには、良いogr2ogrのチートシートがあり、参考になるかもしれません。また、GitHubにはいくつかの有用なGDALのチートシートがあります。また、地理空間フォーマットをElasticsearchに取り込むためのコマンドラインスクリプトのサンプルのリストも作成しました。もしよろしければ、Elasticsearch Serviceの無料トライアルで試してみてください。ついでにElastic Mapsもチェックしてみてください（まだ見ていない方は）。

もし何か問題があれば、Elastic Discussのフォーラムで質問するか、https://gis.stackexchange.com/。それでは、お楽しみください。
