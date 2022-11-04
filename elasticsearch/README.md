# docker
## oss images
- [elasticsearch/elasticsearch-oss](https://www.docker.elastic.co/r/elasticsearch/elasticsearch-oss)
- [kibana/kibana-oss](https://www.docker.elastic.co/r/kibana/kibana-oss)
- 7系までしかないので古い

## install with docker
- [Install Elasticsearch with Docker](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)
  - [Start a multi-node cluster with Docker Composee](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)
- [Install Kibana with Docker](https://www.elastic.co/guide/en/kibana/current/docker.html)

## docker compose up に失敗する

```log
elasticsearch-es01-1    | ERROR: [1] bootstrap checks failed. You must address the points described in the following [1] lines before starting Elasticsearch.
elasticsearch-es01-1 exited with code 78
container for service "es01" is unhealthy
```

- [Set vm.max_map_count to at least 262144](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#_set_vm_max_map_count_to_at_least_262144)
  - 一時的な対処であれば下記コマンドを打てばいい
    - `$ sysctl -w vm.max_map_count=262144`
- wslでは `/etc/sysctl.conf` に `vm.max_map_count=262144` を書き込むだけでは不十分らしい
  - [Windows で Docker で Kibana を入れる](https://www.toyfish.blog/entry/2022/05/04/040025)

## elastic search への curlが失敗する
- index 一覧を取得するrequest, response
  ```bash
  $ curl -XGET 'localhost:9200/_cat/indices?v'
  curl: (52) Empty reply from server
  ```

- 下記ドキュメントのpathをdocker-compose.ymlのpathに読み替えて `.crt` ファイルをlocalにcopyする
  - `$ sudo docker cp single-node-es01-1:/usr/share/elasticsearch/config/certs/ca/ca.crt ./http_ca.crt`
  - [Install Elasticsearch with Docker > Start a single-node cluster with Docker > 4.](docs/elasticsearch/set_up_elasticsearch/install_elasticsearch_with_docker/japanese.md)
    > Copy the http_ca.crt security certificate from your Docker container to your local machine.
    ```bash
    docker cp es01:/usr/share/elasticsearch/config/certs/http_ca.crt .
    ```

- 下記のようにcurlを叩けるようになる
  ```bash
  $ sudo curl --cacert http_ca.crt -u elastic https://localhost:9200/_cat/indices?v
  Enter host password for user 'elastic':
  health status index            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
  yellow open   a001010120160205 jep2izLdQ4-Q0XyAw-Uitg   1   1         46            0        1mb            1mb
  ```


## 動作確認
- [国土数値情報ダウンロードサービス > 農業地域データ > 北海道 > 平成27年](https://nlftp.mlit.go.jp/ksj/gml/datalist/KsjTmplt-A12.html)
- [GeoJSONのオープンデータをKibanaのマップに表示する](https://qiita.com/mkyz08/items/9dbae101dbaeaec087d1)
  - この記事同様、文字化けした
