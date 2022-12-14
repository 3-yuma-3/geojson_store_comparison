# 2d Indexes
2 次元平面上の点として保存されたデータには、2d インデックスを使用します。2d インデックスは、MongoDB 2.2 以前で使われていたレガシーな座標ペアを対象としています。

次のような場合は 2 次元インデックスを使います。

- データベースに MongoDB 2.2 以前のレガシーな座標ペアがある。
- 位置データを GeoJSON オブジェクトとして保存するつもりがない場合。

地理空間クエリの詳細については、地理空間クエリを参照ください。

## Considerations
MongoDB 4.0 以降、$geoNear パイプラインステージにキーオプションを指定して、 使用するインデックス付きフィールドパスを指定できるようになりました。これにより、複数の 2 次元インデックスと複数の 2 次元球インデックスを持つコレクションで $geoNear ステージが使えるようになります。

- コレクションに複数の 2d インデックスと複数の 2dsphere インデックスがある場合、key オプションを使用して使用するインデックス付きフィールドパスを指定する必要があります。
- キーを指定しないと、複数の 2d インデックスや 2dsphere インデックスからインデックスを選択することが曖昧になるため、複数の 2d インデックスや 2dsphere インデックスを持つことはできません。

キーを指定せず、2 次インデックスと 2 次インデックスの両方あるいはどちらか一方だけを指定した場合、 MongoDB はまず 2 次インデックスを探します。2 次元インデックスが存在しない場合は、MongoDB は 2 次元球のインデックスを探します。

位置データに GeoJSON オブジェクトが含まれている場合、2d インデックスを使用しないでください。レガシー座標ペアと GeoJSON オブジェクトの両方でインデックスを作成するには、2dsphere インデックスを使用します。

コレクションをシャーディングする際、2d インデックスをシャード・キーとして使用することはできない。ただし、別のフィールドをシャード キーとして使用することで、シャードされたコレクションに地理空間インデックスを作成することができる。

## Behavior
2d インデックスは、平らなユークリッド平面上での計算をサポートしています。2d インデックスは、球面上の距離のみの計算 (つまり $nearSphere) もサポートしていますが、球面上の幾何学的な計算 (たとえば $geoWithin) では、データを GeoJSON オブジェクトとして保存して、2dsphere インデックスを使用します。

2d インデックスは、2 つのフィールドを参照することができます。最初のフィールドは、位置フィールドでなければなりません。2d 複合インデックスは、まず location フィールドを選択し、その結果を追加基準でフィルタリングするクエリを作成します。2次元の複合インデックスは、クエリをカバーすることができます。

## sparse Property
2d インデックスは常にスパースで、スパースオプションは無視されます。ドキュメントに 2 次インデックスフィールドがない (あるいはフィールドが null か空の配列) 場合、MongoDB はそのドキュメントのエントリを 2 次インデックスに追加しません。挿入の場合は、MongoDB はドキュメントを挿入しますが、2d インデックスには追加しません。

2d インデックスキーと他の型のキーを含む複合インデックスの場合は、 そのインデックスがドキュメントを参照しているかどうかを決めるのは 2d インデックスフィールドだけです。

## Collation Option
2d インデックスは単純なバイナリ比較のみをサポートし、照合順序オプションはサポートしません。

単純でない照合順序を持つコレクションに 2 次元インデックスを作成するには、明示的に {collation.Locale: {collation.Locale} を指定する必要があります。{locale: "simple"} を指定する必要があります。を明示的に指定する必要があります。
