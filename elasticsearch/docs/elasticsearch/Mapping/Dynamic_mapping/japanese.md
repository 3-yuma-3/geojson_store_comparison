# Dynamic mapping
Elasticsearchの最も重要な特徴の一つは、お客様の邪魔をせず、できるだけ早くデータの探索を始められるようにしようとすることです。インデックスを作成し、マッピングタイプを定義し、フィールドを定義する必要がなく、ドキュメントをインデックスするだけでインデックス、タイプ、フィールドが自動的に表示されます。

```bash
PUT data/_doc/1 ---1
{ "count": 5 }
```

1. データインデックス、_docマッピングタイプ、およびデータ型がlongのcountというフィールドを作成する。

新しいフィールドを自動的に検出して追加することをダイナミックマッピングと呼びます。ダイナミックマッピングのルールは、以下のように目的に合わせてカスタマイズすることが可能です。

- Dynamic field mappings
  - ダイナミックフィールドの検出を管理するルール。
- Dynamic templates
  - 動的に追加されるフィールドのマッピングを設定するためのカスタムルール。

インデックステンプレートを使用すると、 自動的あるいは明示的に作成された新しいインデックスに対して、 デフォルトのマッピング、設定、エイリアスを設定することができます。