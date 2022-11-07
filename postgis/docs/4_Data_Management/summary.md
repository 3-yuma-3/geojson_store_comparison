# PostGIS 3.3.0 マニュアル


## 4. データ管理
### 4.1. 空間データ モデル
#### 4.1.1. OGC ジオメトリ
- 基本的な geometry のデータ型の説明

#### 4.1.2. SQL/MM Part 3 - 曲線
#### 4.1.3. WKTとWKB
- geometry の入出力関数についての説明

- [ST_AsText](http://cse.naro.affrc.go.jp/yellow/pgisman/3.3.0/ST_AsText.html)
  - ジオメトリ/ジオグラフィのWell-Known Text (WKT)表現を返します。
- [ST_GeomFromText](http://cse.naro.affrc.go.jp/yellow/pgisman/3.3.0/ST_GeomFromText.html)
  - OGC Well-Known Text表現からPostGIS ST_Geometryオブジェクトを生成します。

> WKTの入出力は関数ST_AsTextとST_GeomFromTextによって提供されます。
> ```sql
> text WKT = ST_AsText(geometry);
> geometry = ST_GeomFromText(text WKT, SRID);
> ```
>
> 例えば、WKTとSRIDからの空間オブジェクトの生成と挿入のステートメントは次の通りです。
>
> ```sql
> INSERT INTO geotable ( geom, name )
>   VALUES ( ST_GeomFromText('POINT(-126.4 45.32)', 312), 'A Place');
> ```

### 4.2. ジオメトリデータタイプ
- geometry 型は平面

#### 4.2.1. PostGIS EWKBとEWKT
> これらの書式を使う入出力は次の関数を使うと有効です。
>
> ```
> バイト配列 EWKB = ST_AsEWKB(geometry);
> テキスト EWKT = ST_AsEWKT(geometry);
> ジオメトリ = ST_GeomFromEWKB(bytea EWKB);
> ジオメトリ = ST_GeomFromEWKT(text EWKT);
> ```
>
> たとえば、EWKTを使ってPostGISの空間オブジェクトを作成し挿入するステートメントは次の通りです。
>
> ```sql
> INSERT INTO geotable ( geom, name )
>   VALUES ( ST_GeomFromEWKT('SRID=312;POINTM(-126.4 45.32 15)'), 'A Place' )
> ```

### 4.3. ジオグラフィデータタイプ
- geography 型は球面モデル


#### 4.3.1. ジオグラフィテーブルの生成

> ```sql
> CREATE TABLE global_points (
>     id SERIAL PRIMARY KEY,
>     name VARCHAR(64),
>     location geography(POINT,4326)
>   );
> ```

#### 4.3.2. ジオグラフィテーブルの使用
#### 4.3.3. ジオグラフィ型を使用すべき時
> ジオグラフィ型によって、経度緯度座標でデータを格納できるようになりましたが、ジオグラフィで定義されている関数が、ジオメトリより少ないのと、実行にCPU時間がかかる、というところが犠牲になっています。

#### 4.3.4. ジオグラフィに関する高度なよくある質問
##### 4.3.4.1. 球または回転楕円体のどちらで計算するのでしょうか?
##### 4.3.4.2. 日付変更線や極に関してはどうなっていますか?
##### 4.3.4.3. 処理できる最も長い弧はどうなりますか?
##### 4.3.4.4. なぜヨーロッパやロシアといった大きな範囲の面積計算はとても遅いのですか?

### 4.4. ジオメトリ検証
#### 4.4.1. 単純ジオメトリ
- 各 geometry について図示
  - POINT, MULTIPOINT, LINESTRING, MULTILINESTRING, POLYGON

> 単純なジオメトリは、自己交差や自己接触といった異常な幾何学上のポイントを持たないジオメトリです。

> ジオメトリが単純かどうかを試すにはST_IsSimple関数を使います。次のようにします。
>
> ```sql
> SELECT
>    ST_IsSimple('LINESTRING(0 0, 100 100)') AS straight,
>    ST_IsSimple('LINESTRING(0 0, 100 100, 100 0, 0 100)') AS crossing;
>
>  straight | crossing
> ----------+----------
>  t        | f
> ```

#### 4.4.2. 妥当なジオメトリ
POLYGON, MULTIPOLYGONの妥当条件が記載・図示されている

> ジオメトリの妥当性は主に2次元ジオメトリ (POLYGONとMULTIPOLYGON)に適用されます。妥当性はポリゴンジオメトリが平面領域を明確にモデル化できる規則によって定義されます。

> 線ジオメトリについては、LINESTRINGが少なくとも二つのポイントを持ち、長さが0でない (少なくとも二つの異なるポイントを持つことと同じ)、というのが唯一の妥当性規則です。単純でない (自己交差がある)ラインは妥当です。
>
> ```sql
> SELECT
>    ST_IsValid('LINESTRING(0 0, 1 1)') AS len_nonzero,
>    ST_IsValid('LINESTRING(0 0, 0 0, 0 0)') AS len_zero,
>    ST_IsValid('LINESTRING(10 10, 150 150, 180 50, 20 130)') AS self_int;
>
>  len_nonzero | len_zero | self_int
> -------------+----------+----------
>  t           | f        | t
> ```

#### 4.4.3. 妥当性の管理
> PostGISは妥当なジオメトリも不正なジオメトリも、生成も格納もできます。このため、不正なジオメトリを検出し、フラグを付け、訂正することができます。OGC妥当性規則が求める規則 (長さが0のラインストリングや逆穴を持つポリゴン等)よりも厳格であることもあります。
>
> PostGISが提供する関数の多くは、引数ジオメトリが妥当であるとの仮定によっています。
>
> ジオメトリが妥当かをテストするにはST_IsValid関数を使います。次のようにします。
>
> ```sql
> SELECT ST_IsValid('POLYGON ((20 180, 180 180, 180 20, 20 20, 20 180))');
> -----------------
>  t
> ```
>
> ジオメトリの不正性の性質と位置に関する情報はST_IsValidDetail関数で得られます。次のようにします。
>
> ```sql
> SELECT valid, reason, ST_AsText(location) AS location
>     FROM ST_IsValidDetail('POLYGON ((20 20, 120 190, 50 190, 170 50, 20 20))') AS t;
>
>  valid |      reason       |                  location
> -------+-------------------+---------------------------------------------
>  f     | Self-intersection | POINT(91.51162790697674 141.56976744186045)
> ```
>
> 不正なジオメトリを自動的に訂正することが望ましいような状況があります。その際はST_MakeValid関数を使います (ST_MakeValidは不正な入力を許す特別な関数です)。
>
> 複雑なジオメトリの不正性テストには多大なCPU時間を取ることになるため、デフォルトでは、ジオメトリのロード時にPostGISは妥当性の確認をしません。データソースが信用できない場合には、チェック制約を使って、テーブル上で妥当性を強制的に確認することができます。次のようにします。
>
> ```sql
> ALTER TABLE mytable
>   ADD CONSTRAINT geometry_valid_check
>         CHECK (ST_IsValid(geom));
> ```

### 4.5. 空間参照系
> 空間参照系 (Spatial Reference System, SRS) (座標参照系、Coordinate Reference System, CRSとも呼ばれます)は、ジオメトリが地表上の位置をどのように参照するかを定義しています。SRSには次の通り三種あります。
>
> 測地 (geodetic) 空間参照系は、地表に直接対応付けられる極座標系 (経度と緯度)を使います。
>
> 投影 (projected)空間参照系は、回転楕円体面を「平面にする」ための数学的な投影変換を使います。距離、面積、角度といった量を直接計測することが可能な位置座標系です。この座標系はデカルト座標系ですので、原点と二つの直交軸 (通常は来北と東方向)が定義されています。個々の投影座標系は、定まった距離単位 (通常はメートルかフィート)を使います。投影座標系は、歪みを避けて定義された座標範囲に納めるために、適応範囲を制限してもいいことになっています。
>
> 局所 (local)座標系は、地表への参照がないデカルト座標系です。PostGISではSRID値を0に指定します。

#### 4.5.1. SPATIAL_REF_SYSテーブル
> PostGISが使用するSPATIAL_REF_SYSテーブルは利用可能な空間参照系を定義するOGC準拠のデータベーステーブルです。このテーブルは、数値でSRIDを持ち、文字列で座標系の記述を持っています。
>
> spatial_ref_sysの定義は次の通りです。
>
> ```sql
> CREATE TABLE spatial_ref_sys (
>   srid       INTEGER NOT NULL PRIMARY KEY,
>   auth_name  VARCHAR(256),
>   auth_srid  INTEGER,
>   srtext     VARCHAR(2048),
>   proj4text  VARCHAR(2048)
> )
> ```

#### 4.5.2. ユーザ定義空間参照系
> PostGISspatial_ref_sysテーブルにはPROJ投影ライブラリで処理される最も一般的な空間参照系定義3000件以上があります。しかし、そこに無い多くの座標系があります。空間参照系に関する必要な情報がある場合は、SRS定義をテーブルに追加できます。PROJに詳しいなら独自の空間参照系を定義することもできます。ほとんどの空間参照系は地域的なものであり、目的の範囲外で使用する場合は意味を持たない点に注意してください。
>
> PostGISのコアセットに入っていない空間参照系を探すための素晴らしい資料がhttp://spatialreference.org/にあります。


> 割当外のSRIDとPROJ定義を使って米国中央のランベルト正角円錐図法の独自座標系をロードする例を次に示します。
>
> ```sql
> INSERT INTO spatial_ref_sys (srid, proj4text)
> VALUES ( 990000,
>   '+proj=lcc  +lon_0=-95 +lat_0=25 +lat_1=25 +lat_2=25 +x_0=0 +y_0=0 +datum=WGS84 +units=m +no_defs'
> );
> ```

### 4.6. 空間テーブル
#### 4.6.1. 空間テーブルを作る
#### 4.6.2. GEOMETRY_COLUMNSビュー
#### 4.6.3. 手動でジオメトリカラムをgeometry_columnsに登録する
### 4.7. 空間データのロード
#### 4.7.1. SQLを使ってロードする
#### 4.7.2. シェープファイルローダを使う
### 4.8. 空間データの抽出
#### 4.8.1. SQLを使ってデータを抽出する
#### 4.8.2. ダンパを使う
### 4.9. 空間インデックス
#### 4.9.1. GiSTインデックス
#### 4.9.2. BRINインデックス
#### 4.9.3. SP-GiSTインデックス
#### 4.9.4. インデックス使用のチューニング
