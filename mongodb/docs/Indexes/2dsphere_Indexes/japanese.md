# 2dsphere Indexes
## Overview
2dsphere インデックスは、地球のような球体上の形状を計算するクエリをサポートします。2dsphere インデックスは、MongoDB のすべての地理空間クエリ (包含、交差、近接のクエリ) をサポートしています。地理空間クエリの詳細については、地理空間クエリを参照ください。

2dsphere インデックスは、GeoJSON オブジェクトとレガシー座標ペアで保存されたデータをサポートしています (2dsphere インデックス化フィールドの制限も参照ください)。レガシー座標ペアの場合、インデックスによってデータが GeoJSON ポイントに変換されます。

## Versions
- Version 3
  - MongoDB 3.2 では、2dsphere インデックスにバージョン 3 が導入されました。バージョン 3 は、MongoDB 3.2 以降で作成される 2dsphere インデックスのデフォルトバージョンです。
- Version 2
  - MongoDB 2.6 では、2dsphere インデックスのバージョン 2 が導入されました。バージョン 2 は、MongoDB 2.6 および 3.0 系で作成される 2dsphere インデックスのデフォルトバージョンです。

デフォルトのバージョンを上書きして別のバージョンを指定するには、インデックス作成時にオプション { "2dsphereIndexVersion" を指定します。<version> } を含めると、インデックスを作成できます。

### sparse Property
バージョン 2 以降の 2dsphere インデックスは常にスパースで、スパースオプションは無視されます。ドキュメントに 2dsphere インデックスフィールドがない (あるいはフィールドが null か空の配列である) 場合、MongoDB はそのドキュメントのエントリをインデックスに追加しません。挿入の場合は、MongoDB はドキュメントを挿入しますが、2dsphere インデックスには追加しません。

2dsphere インデックスキーと他の型のキーを含む複合インデックスの場合は、 2dsphere インデックスフィールドだけがそのインデックスがドキュメントを参照しているかどうかを判断します。

MongoDB の以前のバージョンでは、2dsphere (バージョン 1) インデックスしかサポートしていません。2dsphere (バージョン 1) のインデックスはデフォルトではスパースではないので、 位置フィールドが NULL のドキュメントは拒否されます。

### Additional GeoJSON Objects
バージョン 2 以降の 2dsphere インデックスには、追加の GeoJSON オブジェクトのサポートが含まれています。MultiPoint、MultiLineString、MultiPolygon、および GeometryCollection。サポートされているすべての GeoJSON オブジェクトの詳細については、「GeoJSON オブジェクト」を参照してください。

## Considerations
### geoNear and $geoNear Restrictions
MongoDB 4.0 以降、$geoNear パイプラインステージにキーオプションを指定して、 使用するインデックス付きフィールドパスを指定することができます。これにより、複数の 2dsphere インデックスと複数の 2d インデックスを持つコレクションで $geoNear ステージが使えるようになります。

コレクションに複数の 2dsphere インデックスと複数の 2d インデックスがある場合は、key オプションを使用して、使用するインデックス付きフィールドパスを指定する必要があります。

キーを指定しないと、複数の 2d インデックスや 2dsphere インデックスからインデックスの選択があいまいになってしまうからです。

キーを指定せず、2dsphere インデックスと 2d インデックスの両方あるいはどちらか一方だけを指定した場合は、 MongoDB はまず 2d インデックスを探します。2d インデックスが存在しない場合は、MongoDB は使用する 2dsphere インデックスを探します。

### Shard Key Restrictions
コレクションをシャーディングする際、2dsphere インデックスをシャード・キーとして使用することはできません。しかし、別のフィールドをシャードキーとして使用することで、シャードされたコレクションに地理空間インデックスを作成することができる。

### 2dsphere Indexed Field Restrictions
2dsphere インデックスを持つフィールドは、座標ペアまたは GeoJSON データの形式でジオメトリデータを保持する必要があります。2dsphere インデックスを持つフィールドにジオメトリ以外のデータを持つドキュメントを挿入しようとしたり、インデックスを持つフィールドがジオメトリ以外のデータを持つコレクションに 2dsphere インデックスを構築しようとすると、操作に失敗することになる。

### Limited Number of Index Keys
2dsphere インデックス用のキーを生成するために、mongod は GeoJSON のシェイプを内部表現にマップします。その結果、内部表現は大きな値の配列になる可能性があります。

mongod が配列を保持するフィールドにインデックスキーを生成するときは、 配列の各要素に対してインデックスキーを生成します。複合インデックスの場合、mongod は各フィールドに生成されるキーのセットのデカルト積を計算します。両方のセットが大きい場合は、デカルト積を計算することでメモリの制限を超える可能性があります。

indexMaxNumGeneratedKeysPerDocument は、メモリ不足のエラーを防ぐために、一つの文書に対して生成されるキーの最大数を制限します。デフォルトは1文書あたり100000インデックスキーです。この制限を上げることは可能ですが、indexMaxNumGeneratedKeysPerDocumentパラメータが指定する以上のキーが操作に必要な場合、その操作は失敗します。

## Create a 2dsphere Index
2dsphereインデックスを作成するには、db.collection.createIndex()メソッドを使用して、インデックスタイプとして文字列リテラル "2dsphere "を指定します。

```bash
db.collection.createIndex( { <location field> : "2dsphere" } )
```

ここで、<location field>はGeoJSONオブジェクトかレガシー座標のペアを値とするフィールドである。

1つの位置フィールドと他の1つのフィールドを参照できる複合2Dインデックスとは異なり、複合2dsphereインデックスは、複数の位置と位置以外のフィールドを参照することができます。

以下の例では、位置データをGeoJSON Pointとしてlocというフィールドに格納するドキュメントを持つコレクションプレースを考えてみましょう。

```bash
db.places.insertMany( [
   {
      loc : { type: "Point", coordinates: [ -73.97, 40.77 ] },
      name: "Central Park",
      category : "Parks"
   },
   {
      loc : { type: "Point", coordinates: [ -73.88, 40.78 ] },
      name: "La Guardia Airport",
      category : "Airport"
   }
] )
```

### Create a 2dsphere Index
次の操作は、位置フィールドlocに2dsphereインデックスを作成するものである。

```bash
次の操作は、位置フィールドlocに2dsphereインデックスを作成するものである。
```

### Create a Compound Index with 2dsphere Index Key
複合インデックスには、2dsphere インデックスキーと、地理空間以外のインデックスキーとを組み合わせることができる。例えば、次の操作は、最初のキーlocが2dsphereインデックスキーで、残りのキーcategoryとnamesが非地域空間インデックスキー、具体的にはそれぞれ降順（-1）、昇順（1）キーである複合インデックスを作成するものである。

```bash
db.places.createIndex( { loc : "2dsphere" , category : -1, name: 1 } )
```

2dインデックスとは異なり、複合2dsphereインデックスは、locationフィールドが最初のフィールドである必要はない。例えば

```bash
db.places.createIndex( { category : 1 , loc : "2dsphere" } )
```
