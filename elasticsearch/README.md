# docker
## oss images
- [elasticsearch/elasticsearch-oss](https://www.docker.elastic.co/r/elasticsearch/elasticsearch-oss)
- [kibana/kibana-oss](https://www.docker.elastic.co/r/kibana/kibana-oss)

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
