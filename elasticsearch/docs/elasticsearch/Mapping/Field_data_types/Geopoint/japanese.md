# Geopoint field type
geo_point 型のフィールドは、緯度経度のペアを受け付けます。

- bounding box内、中心点から一定のdistance内、または geo_shape クエリ内 (たとえば、多角形内の点) のジオポイントを検索するため。
- 中心点からのdistanceでドキュメントを集計する
- 地理的なグリッドでドキュメントを集約する: geo_hash, geo_tile, geo_hex のいずれかを使用します。
- distanceをドキュメントの関連性スコアに統合する。
- 距離でドキュメントをソートする。
-
geo_shape や point と同様に、geo_point は GeoJSON および Well-Known Text 形式で指定することができます。しかし、利便性と歴史的な理由から、さらに多くの形式がサポートされています。以下に示すように、ジオポイントの指定方法は合計で 6 通りあります。

```bash
PUT my-index-000001
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_point"
      }
    }
  }
}

PUT my-index-000001/_doc/1
{
  "text": "Geopoint as an object using GeoJSON format",
  "location": { ---1
    "type": "Point",
    "coordinates": [-71.34, 41.12]
  }
}

PUT my-index-000001/_doc/2
{
  "text": "Geopoint as a WKT POINT primitive",
  "location" : "POINT (-71.34 41.12)" ---2
}

PUT my-index-000001/_doc/3
{
  "text": "Geopoint as an object with 'lat' and 'lon' keys",
  "location": { ---3
    "lat": 41.12,
    "lon": -71.34
  }
}

PUT my-index-000001/_doc/4
{
  "text": "Geopoint as an array",
  "location": [ -71.34, 41.12 ] ---4
}

PUT my-index-000001/_doc/5
{
  "text": "Geopoint as a string",
  "location": "41.12,-71.34" ---5
}

PUT my-index-000001/_doc/6
{
  "text": "Geopoint as a geohash",
  "location": "drm3btev3e86" ---6
}

GET my-index-000001/_search
{
  "query": {
    "geo_bounding_box": { ---7
      "location": {
        "top_left": {
          "lat": 42,
          "lon": -72
        },
        "bottom_right": {
          "lat": 40,
          "lon": -74
        }
      }
    }
  }
}
```

1. GeoJSON フォーマットのオブジェクトとして表現され、typeとcooridnatesのキーを持つジオポイント。
2. ジオポイントは Well-Known Text POINT のフォーマットで表現される。"POINT(lon lat)" というフォーマットで表されます。
3. latとlonのキーを持つオブジェクトとして表現されたジオポイント。
4. ジオポイントを配列で表現し、その形式は "POINT(lon, lat)"。[lon, lat] というフォーマットで表現される。
5. ジオポイントを文字列で表現したもので、フォーマットは "lat,lon"。"lat,lon" の文字列で表現されます。
6. geohashで表現されたジオポイント。
7. geo-bounding boxクエリで、ボックスの中に入るジオポイントをすべて見つけます。

**Geopoints expressed as an array or string**
文字列ジオポイントはlat,lonの順に並びますが、配列ジオポイント、GeoJSON、WKTは逆にlon,latの順に並ぶことに注意してください。

この理由は歴史的なものです。地理学者は伝統的に緯度を経度の前に書きますが、GeoJSON や Well-Known Text などの地理データ用に指定された最近のフォーマットは、x を y の前に並べるという数学的慣習に合わせるために、緯度の前に経度（東経を北緯の前に）を並べています。

点はジオハッシュで表現することができます。ジオハッシュは緯度と経度のビットをインターリーブしたbase32エンコードされた文字列です。ジオハッシュの各文字は5ビットずつ追加され、精度が向上します。したがって、ハッシュが長ければ長いほど、精度が高くなります。インデックスを作成するために、ジオハッシュは緯度経度のペアに変換されます。このとき、最初の12文字だけが使われるため、ジオハッシュで12文字以上指定しても精度は上がりません。12文字で60ビットになるので、起こりうる誤差は2cm以下になるはずです。

## Parameters for geo_point fields
- ignore_malformed
  - true の場合、不正確なジオポイントは無視されます。false （デフ ォ ル ト ） の場合、 不正な形の geopoint は例外を発生 さ せ、 文書全体を拒否 し ます。ジオポイントは、緯度が -90 ⇐ 緯度 ⇐の範囲外、あるいは経度が -180 ⇐ 経度 ⇐ の範囲外の場合、不正とみなされます。script パラメータが使われている場合は設定できないことに注意。
- ignore_z_value
  - true の場合（デフォルト）、3 次元のポイントが受け入れられますが（ソースに保存されます）、緯度と経度の値のみがインデックス化され、3 次元は無視されます。false の場合、 緯度 と 経度 （2 次元） 以外の値を含む地理ポ イ ン ト は例外を発生 さ せ、 文書全体が拒否 さ れます。script パラメータを使用している場合は設定できないことに注意しましょう。
- index
  - そのフィールドをすばやく検索できるようにするか。true (デフォルト) と false を指定します。doc_values しか有効になっていないフィールドでも、 ゆっくりではありますが検索することができます。
- null_value
  - 明示的な null 値の代わりとなるジオポイントの値を指定します。デフォルトはNULLで、これはフィールドが見つからないものとして扱われることを意味します。script パラメータが使用されている場合、これは設定できないことに注意。
- on_script_error
  - script パラメータで定義されたスクリプトがインデックス作成時にエラーをスローした場合にどうするかを定義します。fail (デフォルト) を選ぶとドキュメント全体が拒否され、 continue を選ぶとドキュメントの _ignored メタデータフィールドにそのフィールドを登録してインデックス作成を継続します。このパラメータは、script フィールドが設定されている場合にのみ設定できます。
- script
  - このパラメータが設定されている場合、フィールドはソースから直接値を読み取るのではなく、このスクリプトによって生成された値をインデックス化します。入力ドキュメントでこのフィールドに値が設定されている場合、ドキュメントはエラーで拒否されます。スクリプトはランタイムと同じフォーマットであり、ポイントを(lat, lon)のダブル値の組として出力する必要がある。

## Using geopoints in scripts
スクリプトでジオポイントの値にアクセスする場合、値はジオポイントオブジェクトとして返され、.lat と .lon の値にそれぞれアクセスできるようになります。

```python
def geopoint = doc['location'].value;
def lat      = geopoint.lat;
def lon      = geopoint.lon;
```

パフォーマンス上の理由から、lat/lon 値に直接アクセスする方がよいでしょう。

```python
def lat      = doc['location'].lat;
def lon      = doc['location'].lon;
```

## Synthetic source [preview]
geo_point フィールドは、デフォルトの設定で synthetic _source をサポートしています。合成 _source は， ignore_malformed や copy_to と一緒に使うことはできませんし， doc_values を無効にした状態でも使えません。

合成ソースは，常に geo_point フィールドをソートし（最初に緯度，次に経度で），保存されている精度に落とし込みます。たとえば

```bash
PUT idx
{
  "mappings": {
    "_source": { "mode": "synthetic" },
    "properties": {
      "point": { "type": "geo_point" }
    }
  }
}
PUT idx/_doc/1
{
  "point": [
    {"lat":-90, "lon":-80},
    {"lat":10, "lon":30}
  ]
}
```

こうなる

```bash
{
  "point": [
    {"lat":-90.0, "lon":-80.00000000931323},
    {"lat":9.999999990686774, "lon":29.999999972060323}
   ]
}
```
