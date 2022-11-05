# Explicit mapping
Elasticsearchが推測するよりも、お客様のデータについて知っていることの方が多いでしょう。ですから、ダイナミックマッピングは使い始めには便利ですが、いつかはお客様自身が明示的にマッピングを指定したいと思うようになるでしょう。

フィールドマッピングはインデックスの作成時や既存のインデックスにフィールドを追加する際に作成することができます。

## Create an index with an explicit mapping
create index APIを使用すると、明示的なマッピングで新しいインデックスを作成することができます。

```bash
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "age":    { "type": "integer" },
      "email":  { "type": "keyword"  },
      "name":   { "type": "text"  }
    }
  }
}
```

1. Creates age, an integer fiel
2. Creates email, a keyword field
3. Creates name, a text field

## Add a field to an existing mapping
update mapping API を使用すると、既存のインデックスに 1 つ以上の新しいフィールドを追加できます。

次の例では、キーワードフィールドである employee-id をインデックスマッピングパラメータ値として false で追加しています。これは、employee-id フィールドの値は保存されるが、インデックス化されず、検索に利用できないことを意味します。

```bash
PUT /my-index-000001/_mapping
{
  "properties": {
    "employee-id": {
      "type": "keyword",
      "index": false
    }
  }
}
```

## Update the mapping of a field
サポートされているマッピング・パラメータを除き、既存のフィールドのマッピングやフィールド・タイプを変更することはできません。既存のフィールドを変更すると、すでにインデックスが作成されているデータが無効になる可能性があります。

データストリームのバックインデックス内のフィールドのマッピングを変更する必要がある場合は、「データストリームのマッピングと設定の変更」を参照してください。

他のインデックスのフィールドのマッピングを変更する必要がある場合は、正しいマッピングを持つ新しいインデックスを作成し、そのインデックスにデータを再インデックスします。

フィールド名を変更すると、古いフィールド名ですでにインデックスされているデータが無効になってしまいます。代わりに、エイリアスフィールドを追加して、別のフィールド名を作成します。

## View the mapping of an index
get mapping APIを使用すると、既存のインデックスのマッピングを表示することができます。

```bash
GET /my-index-000001/_mapping
```

APIは以下のレスポンスを返します。

```bash
{
  "my-index-000001" : {
    "mappings" : {
      "properties" : {
        "age" : {
          "type" : "integer"
        },
        "email" : {
          "type" : "keyword"
        },
        "employee-id" : {
          "type" : "keyword",
          "index" : false
        },
        "name" : {
          "type" : "text"
        }
      }
    }
  }
}
```

## View the mapping of specific fields
一つあるいは複数の特定のフィールドのマッピングを表示したいだけなら、 フィールドマッピングの取得 API を使用します。

これは、インデックスの完全なマッピングを必要としない場合や、 インデックスに多数のフィールドが含まれている場合に便利です。

以下のリクエストは、employee-id フィールドのマッピングを取得します。

```bash
GET /my-index-000001/_mapping/field/employee-id
```

APIは以下のレスポンスを返します。

```bash
{
  "my-index-000001" : {
    "mappings" : {
      "employee-id" : {
        "full_name" : "employee-id",
        "mapping" : {
          "employee-id" : {
            "type" : "keyword",
            "index" : false
          }
        }
      }
    }
  }
}

```
