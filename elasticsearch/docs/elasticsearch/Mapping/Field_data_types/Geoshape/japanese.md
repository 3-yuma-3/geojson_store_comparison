# Geoshape field type
geo_shape データ型は、矩形、直線、多角形など任意の形状のインデックス作成と検索を容易にする。インデックスを作成するデータが点以外の形状を含む場合は、このマッピングを使用する必要があります。データに点だけが含まれる場合は、geo_point または geo_shape としてインデックスを作成することができる。

この型を使用したドキュメントを使用することができる。

- 内にあるgeoshapesを見つけるために
  - a bounding box
  - 中心点からあるdistance
  - geo_shapeクエリ（例えば、交差する多角形）。
- 地理的なグリッドでドキュメントを集計する。
  - geo_hash
  - または geo_tile のいずれかです。

geo_hex グリッドに対するグリッドアグリゲーションは、geo-shape フィールドに対してはサポートされていません。

## Mapping Options
geo_shape マッピングは、GeoJSON または WKT ジオメトリオブジェクトを geo_shape タイプにマッピングします。これを有効にするには、ユーザーが明示的にフィールドを geo_shape 型にマッピングする必要があります。

- orientation
  - オプション。フィールドのWKTポリゴンのデフォルトの方向。
  - このパラメータは、RIGHT (反時計回り) または LEFT (時計回り) の値のみを設定し、返す。ただし、どちらの値も複数の方法で指定することができる。
  - RIGHTを設定するには、以下の引数のうちの1つ、またはその大文字のバリエーションを使用します。
    - right
    - counterclockwise
    - ccw
  - LEFTを設定するには、以下の引数のうちの1つ、またはその大文字のバリエーションを使用します。
    - left
    - clockwise
    - cw
- ignore_malformed
  - true の場合、不正な GeoJSON または WKT の図形は無視されます。false（デフォルト）の場合、不正なGeoJSONおよびWKTシェイプは例外をスローし、ドキュメント全体を拒否します。
- ignore_z_value
  - true （デフ ォル ト ） の場合、 3 次元のポ イ ン ト が受け入れ ら れますが （ ソ ース に格納 さ れます） 、 イ ンデ ッ ク ス さ れる のは緯度 と 経度の値だけで、 3 次元は無視 さ れ ます。false の場合、緯度と経度（2次元）以上の値を含むジオポイントは例外をスローし、ドキュメント全体が拒否されます。
- coerce
  - true の場合、ポリゴン内の閉じていない線形リングは自動的に閉じられます。

## Indexing approach
ジオシェイプタイプは、形状を三角形のメッシュに分解し、各三角形をBKDツリーの7次元の点としてインデックスを付けることによって作成されます。これにより、すべての空間的関係は元の形状の符号化されたベクトル表現を用いて計算されるため、ほぼ完璧な空間分解能（小数点以下1e-7桁の精度）が得られます。テッセレータの性能は、主にポリゴン／マルチポリゴンを定義する頂点の数に依存します。

**Example**

```bash
PUT /example
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_shape"
      }
    }
  }
}
```

## Input Structure
図形は GeoJSON または Well-Known Text (WKT) フォーマットのいずれかを使用して表現することができます。以下の表は、GeoJSONとWKTのElasticsearchタイプへのマッピングを示したものです。

- Point
  - 単一の地理的な座標。注：ElasticsearchはWGS-84座標のみを使用します。
- LineString
  - 2点以上の点を持つ任意の線。
- Polygon
  - 最初の点と最後の点が一致しなければならない閉じた多角形。したがって、n辺の多角形を作るにはn + 1個の頂点が必要で、最低4個の頂点が必要です。
- MultiPoint
  - 接続されていないが、関連性があると思われる点の配列。
- MultiLineString
  - 別々のラインストリングの配列。
- MultiPolygon
  - 別々のポリゴンの配列。
- GeometryCollection
  - 複数のタイプを共存させることができる点を除き、multi* shapes と同様の GeoJSON 形状 (例: Point と LineString)。
- N/A
  - 左上と右下の点のみを指定して指定する外接矩形（エンベロープ）。

すべてのタイプで、内側のタイプと座標フィールドの両方が必要です。

GeoJSONとWKT、ひいてはElasticsearchでは、座標配列内の正しい座標順序は経度、緯度（X、Y）です。これは、一般的に口語で緯度、経度（Y、X）を使用する多くの地理空間API（例：Google Maps）とは異なるものです。

### Point
点とは、建物の位置やスマートフォンのGeolocation APIで与えられる現在位置など、1つの地理的な座標のことである。以下は、GeoJSONにおける点の例です。

```bash
POST /example/_doc
{
  "location" : {
    "type" : "Point",
    "coordinates" : [-77.03653, 38.897676]
  }
}
```

以下は、WKTにおけるポイントの例である。

```bash
POST /example/_doc
{
  "location" : "POINT (-77.03653 38.897676)"
}
```

### LineString
2つ以上の位置の配列で定義される線分。2点のみを指定することで、ラインストリングは直線を表します。2点以上を指定すると、任意のパスを作成する。以下は、GeoJSONにおけるラインストリングの例である。

```bash
POST /example/_doc
{
  "location" : {
    "type" : "LineString",
    "coordinates" : [[-77.03653, 38.897676], [-77.009051, 38.889939]]
  }
}
```

以下は、WKTにおけるポイントの例である。

```bash
POST /example/_doc
{
  "location" : "LINESTRING (-77.03653 38.897676, -77.009051 38.889939)"
}
```

上記のラインストリングは、ホワイトハウスを起点に米国連邦議会議事堂までの直線を描く。

### Polygon
多角形は、点のリストのリストで定義される。それぞれの（外側の）リストの最初と最後の点は同じでなければならない（多角形は閉じていなければならない）。以下は、GeoJSONでの多角形の例です。

```bash
POST /example/_doc
{
  "location" : {
    "type" : "Polygon",
    "coordinates" : [
      [ [100.0, 0.0], [101.0, 0.0], [101.0, 1.0], [100.0, 1.0], [100.0, 0.0] ]
    ]
  }
}
```

以下は、WKTにおけるポイントの例である。

```bash
POST /example/_doc
{
  "location" : "POLYGON ((100.0 0.0, 101.0 0.0, 101.0 1.0, 100.0 1.0, 100.0 0.0))"
}
```

最初の配列はポリゴンの外側の境界を表し、他の配列は内側の形状（「穴」）を表します。以下は、穴のあるポリゴンのGeoJSONの例です。

```bash
POST /example/_doc
{
  "location" : {
    "type" : "Polygon",
    "coordinates" : [
      [ [100.0, 0.0], [101.0, 0.0], [101.0, 1.0], [100.0, 1.0], [100.0, 0.0] ],
      [ [100.2, 0.2], [100.8, 0.2], [100.8, 0.8], [100.2, 0.8], [100.2, 0.2] ]
    ]
  }
}
```

以下は、WKTにおけるポイントの例である。

```bash
POST /example/_doc
{
  "location" : "POLYGON ((100.0 0.0, 101.0 0.0, 101.0 1.0, 100.0 1.0, 100.0 0.0), (100.2 0.2, 100.8 0.2, 100.8 0.8, 100.2 0.8, 100.2 0.2))"
}
```

#### Polygon orientation
ポリゴンの向きは、頂点の並び順を表します。RIGHT（反時計回り）またはLEFT（時計回り）です。Elasticsearchは、多角形が国際日付変更線（経度±180°）を越えているかどうかを判断するために、多角形の向きを使用します。

orientation mappingパラメータを使用して、WKTポリゴンのデフォルトの向きを設定することができます。これは、WKTの仕様では、デフォルトの方位を指定したり強制したりしないためです。

GeoJSONポリゴンは、orientationマッピングパラメータの値に関係なく、デフォルトの向きであるRIGHTを使用します。これは、GeoJSON の仕様で、外側の多角形は反時計回りに、内側の図形は時計回りに回転するように決められているためです。

ドキュメントレベルの orientation パラメータを使用すると、GeoJSON 多角形のデフォルトの向きをオーバーライドすることができます。たとえば、次のインデックス作成要求では、ドキュメントレベルの向きを LEFT と指定しています。

```bash
POST /example/_doc
{
  "location" : {
    "type" : "Polygon",
    "orientation" : "LEFT",
    "coordinates" : [
      [ [-177.0, 10.0], [176.0, 15.0], [172.0, 0.0], [176.0, -15.0], [-177.0, -10.0], [-177.0, 10.0] ]
    ]
  }
}
```

Elasticsearchは、ポリゴンが国際日付変更線を越えているかどうかを判断するためにのみ、ポリゴンの向きを使用します。もしポリゴンの最小経度と最大経度の差が180°未満であれば、そのポリゴンは日付変更線を越えておらず、その向きは何の影響も与えません。

ポリゴンの最小経度と最大経度の差が180°以上の場合、Elasticsearchはポリゴンのドキュメントレベルの向きがデフォルトの向きと異なるかどうかをチェックします。方位が異なる場合、Elasticsearchはそのポリゴンが国際日付変更線を越えているとみなし、日付変更線部分でポリゴンを分割します。

### MultiPoint
以下は、GeoJSONのポイント一覧の例です。

```bash
POST /example/_doc
{
  "location" : {
    "type" : "MultiPoint",
    "coordinates" : [
      [102.0, 2.0], [103.0, 2.0]
    ]
  }
}
```

以下は、WKTにおけるポイントの例である。

```bash
POST /example/_doc
{
  "location" : "MULTIPOINT (102.0 2.0, 103.0 2.0)"
}
```

### MultiLineString
以下は、GeoJSON のラインストリングのリストの例である。

```bash
POST /example/_doc
{
  "location" : {
    "type" : "MultiLineString",
    "coordinates" : [
      [ [102.0, 2.0], [103.0, 2.0], [103.0, 3.0], [102.0, 3.0] ],
      [ [100.0, 0.0], [101.0, 0.0], [101.0, 1.0], [100.0, 1.0] ],
      [ [100.2, 0.2], [100.8, 0.2], [100.8, 0.8], [100.2, 0.8] ]
    ]
  }
}
```

以下は、GeoJSON のラインストリングのリストの例である。

```bash
POST /example/_doc
{
  "location" : "MULTILINESTRING ((102.0 2.0, 103.0 2.0, 103.0 3.0, 102.0 3.0), (100.0 0.0, 101.0 0.0, 101.0 1.0, 100.0 1.0), (100.2 0.2, 100.8 0.2, 100.8 0.8, 100.2 0.8))"
}
```

### MultiPolygon
以下は、GeoJSONポリゴンのリストの例です（2番目のポリゴンは穴を含む）。

```bash
POST /example/_doc
{
  "location" : {
    "type" : "MultiPolygon",
    "coordinates" : [
      [ [[102.0, 2.0], [103.0, 2.0], [103.0, 3.0], [102.0, 3.0], [102.0, 2.0]] ],
      [ [[100.0, 0.0], [101.0, 0.0], [101.0, 1.0], [100.0, 1.0], [100.0, 0.0]],
        [[100.2, 0.2], [100.8, 0.2], [100.8, 0.8], [100.2, 0.8], [100.2, 0.2]] ]
    ]
  }
}
```

以下は、WKTポリゴンのリストの例です（2番目のポリゴンは穴を含む）。

```bash
POST /example/_doc
{
  "location" : "MULTIPOLYGON (((102.0 2.0, 103.0 2.0, 103.0 3.0, 102.0 3.0, 102.0 2.0)), ((100.0 0.0, 101.0 0.0, 101.0 1.0, 100.0 1.0, 100.0 0.0), (100.2 0.2, 100.8 0.2, 100.8 0.8, 100.2 0.8, 100.2 0.2)))"
}
```

### Geometry Collection
以下は、GeoJSONジオメトリオブジェクトのコレクションの例である。

```bash
POST /example/_doc
{
  "location" : {
    "type": "GeometryCollection",
    "geometries": [
      {
        "type": "Point",
        "coordinates": [100.0, 0.0]
      },
      {
        "type": "LineString",
        "coordinates": [ [101.0, 0.0], [102.0, 1.0] ]
      }
    ]
  }
}
```

以下は、WKTジオメトリーオブジェクトのコレクションの例である。

```bash
POST /example/_doc
{
  "location" : "GEOMETRYCOLLECTION (POINT (100.0 0.0), LINESTRING (101.0 0.0, 102.0 1.0))"
}
```

#### Envelope
Elasticsearchはenvelope型をサポートしており、形状の左上と右下の座標で構成され、[[minLon, maxLat], [maxLon, minLat]] というフォーマットで外接長方形を表現することが可能です。

```bash
POST /example/_doc
{
  "location" : {
    "type" : "envelope",
    "coordinates" : [ [100.0, 1.0], [101.0, 0.0] ]
  }
}
```
以下は、WKT BBOX フォーマットを使ったエンベロープの例である。

注：WKTの仕様では、minLon, maxLon, maxLat, minLatの順で記述することになっています。

```bash
POST /example/_doc
{
  "location" : "BBOX (100.0, 102.0, 2.0, 0.0)"
}
```

#### Circle
GeoJSON と WKT のいずれも、点-半径の円タイプをサポートしていません。代わりに、円を多角形として近似するために、円インジェストプロセッサを使用します。

## Sorting and Retrieving index Shapes
図形の複雑な入力構造とインデックス表現により、現在、図形のソートやそのフィールドを直接取得することはできません。geo_shape の値は、_source フィールドを通してのみ取得可能である。