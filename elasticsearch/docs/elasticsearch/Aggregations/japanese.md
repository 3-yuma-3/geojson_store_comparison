# Aggregations
集計は、データを指標、統計、その他の分析として要約するものです。集計は、次のような質問に答えるのに役立ちます。

- ウェブサイトの平均ロードタイムは？
- 取引量に基づくと、最も価値のある顧客は誰か？
- ネットワーク上で大きなファイルと見なされるものは何か？
- 各製品カテゴリに含まれる製品の数は？

Elasticsearchはアグリゲーションを3つのカテゴリーに分類しています。

- メトリックアグリゲーション：フィールドの値から合計や平均などのメトリックを計算します。
- バケットアグリゲーション：フィールド値、範囲、その他の基準に基づいて、ドキュメントをバケット（ビンとも呼ばれる）にグループ化します。
- パイプライン集計：文書やフィールドの代わりに、他の集計から入力を得る。

## Run an aggregation
検索APIのaggsパラメータを指定することで、検索の一部として集計を実行することができます。次の検索では、my-fieldに対して用語集計を実行しています。

```bash
GET /my-index-000001/_search
{
  "aggs": {
    "my-agg-name": {
      "terms": {
        "field": "my-field"
      }
    }
  }
}
```

集計結果はレスポンスの集計オブジェクトの中にあります。

```bash
{
  "took": 78,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 5,
      "relation": "eq"
    },
    "max_score": 1.0,
    "hits": [...]
  },
  "aggregations": {
    "my-agg-name": { --1
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": []
    }
  }
}
```

1. my-agg-nameの集計結果です。

## Change an aggregation’s scope
クエリパラメータは、集計を実行する文書を制限するために使用します。

```bash
GET /my-index-000001/_search
{
  "query": {
    "range": {
      "@timestamp": {
        "gte": "now-1d/d",
        "lt": "now/d"
      }
    }
  },
  "aggs": {
    "my-agg-name": {
      "terms": {
        "field": "my-field"
      }
    }
  }
}
```

## Return only aggregation results
デフォルトでは、集約を含む検索では、検索ヒットと集約結果の両方が返されます。集計結果のみを返すには、sizeを0に設定します。

```bash
GET /my-index-000001/_search
{
  "size": 0,
  "aggs": {
    "my-agg-name": {
      "terms": {
        "field": "my-field"
      }
    }
  }
}
```

## Run multiple aggregations
同じリクエストで複数のアグリゲーションを指定することができます。

```bash
GET /my-index-000001/_search
{
  "aggs": {
    "my-first-agg-name": {
      "terms": {
        "field": "my-field"
      }
    },
    "my-second-agg-name": {
      "avg": {
        "field": "my-other-field"
      }
    }
  }
}
```

## Run sub-aggregations
バケット集約は、バケットまたはメトリックの下位集約をサポートする。例えば、avg副集約を持つ項集約は、文書の各バケットの平均値を計算する。サブ集合のネストには、レベルや深さの制限はない。

```bash
GET /my-index-000001/_search
{
  "aggs": {
    "my-agg-name": {
      "terms": {
        "field": "my-field"
      },
      "aggs": {
        "my-sub-agg-name": {
          "avg": {
            "field": "my-other-field"
          }
        }
      }
    }
  }
}
```

このレスポンスは、親集約の下に副集約の結果を入れ子にしています。

```json
{
  ...
  "aggregations": {
    "my-agg-name": { --1
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "foo",
          "doc_count": 5,
          "my-sub-agg-name": { --2
            "value": 75.0
          }
        }
      ]
    }
  }
}
```

1. 親集約であるmy-agg-nameの結果。
2. my-agg-nameのサブアグリゲーションであるmy-sub-agg-nameの結果です。

## Add custom metadata
metaオブジェクトを使用して、カスタムメタデータをアグリゲーションに関連付けます。

```bash
GET /my-index-000001/_search
{
  "aggs": {
    "my-agg-name": {
      "terms": {
        "field": "my-field"
      },
      "meta": {
        "my-metadata-field": "foo"
      }
    }
  }
}
```

レスポンスには、メタオブジェクトがそのまま返されます。

```json
{
  ...
  "aggregations": {
    "my-agg-name": {
      "meta": {
        "my-metadata-field": "foo"
      },
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": []
    }
  }
}
```

## Return the aggregation type
デフォルトでは、集約の結果には、集約の名前は含まれますが、そのタイプは含まれません。集約のタイプを返すには、typed_keys クエリパラメータを使用します。

```bash
GET /my-index-000001/_search?typed_keys
{
  "aggs": {
    "my-agg-name": {
      "histogram": {
        "field": "my-field",
        "interval": 1000
      }
    }
  }
}
```
応答は、アグリゲーション名のプレフィックスとしてアグリゲーションタイプを返す。

**IMPORTANT**
一部の集約は、要求内のタイプとは異なる集約タイプを返します。たとえば、terms、significant terms、percentilesの各集約は、集約されたフィールドのデータ型に応じて異なる集約型を返します。

```bash
{
  ...
  "aggregations": {
    "histogram#my-agg-name": { ---1
      "buckets": []
    }
  }
}
```

1. 集約の種類であるヒストグラムの後に、#セパレータと集約の名前であるmy-agg-nameが続きます。

## Use scripts in an aggregation
フィールドが必要な集計に完全に一致しない場合、実行時フィールドで集計する必要があります。

```bash
GET /my-index-000001/_search?size=0
{
  "runtime_mappings": {
    "message.length": {
      "type": "long",
      "script": "emit(doc['message.keyword'].value.length())"
    }
  },
  "aggs": {
    "message_length": {
      "histogram": {
        "interval": 10,
        "field": "message.length"
      }
    }
  }
}
```

スクリプトはフィールド値を動的に計算するため、集計に多少のオーバーヘッドが加わります。計算にかかる時間だけでなく、項やフィルタなどの一部のアグリゲーションでは、ランタイムフィールドを使った最適化の一部が使えない。合計すると、ランタイムフィールドを使用するためのパフォーマンスコストは、アグリゲーションによって異なる。

## Aggregation caches
より高速なレスポンスを得るために、Elasticsearchは頻繁に実行されるアグリゲーションの結果をシャードリクエストキャッシュにキャッシュしています。キャッシュされた結果を得るには、各検索で同じプリファレンス文字列を使用します。検索結果が不要な場合は、サイズを0に設定し、キャッシュがいっぱいにならないようにします。

Elasticsearch は同じ preference 文字列を持つ検索を同じシャードにルーティングします。シャードのデータが検索間で変更されない場合、シャードはキャッシュされた集計結果を返します。

## Limits for long values
アグリゲーションを実行する際、Elasticsearch は数値データを保持し表現するために double 値を使用します。そのため、253以上の長い数値に対する集計は近似値となります。
