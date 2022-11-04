# Docker and MongoDB
Docker社は2013年に設立され、コンテナの概念を普及させた会社です。それ以来、多くのソフトウェア開発者が日々のワークフローの一部としてコンテナを採用しています。これらのコンテナは、分離された一貫性のある環境でアプリケーションを実行することで、開発者を支援することができます。この記事では、コンテナについて、また、MongoDB を使用したアプリケーションを開発する際にコンテナをどのように使用できるかを学びます。

## What Is a Docker Container?
Dockerコンテナは、アプリケーションとその必要な設定や依存関係をすべて含む1つのユニットです。どのようなオペレーティングシステムやハードウェア上でもアプリケーションを実行できるように、必要なものをすべて含んだ大きなZIPファイルを想像してください。Dockerは、これらのコンテナを実行するためのツールです。

コンテナのコンセプトは、70年代にまでさかのぼります。そのルーツは、Unixオペレーティングシステムの出現にある。オペレーティングシステムの一部として、プロセスとその子プロセスのルートディレクトリを変更するchrootコマンドが導入されました。これがプロセスの分離の始まりでした。

それから数年が経ち、現在のコンテナの状態になりました。コンテナは、アプリケーションの実行に必要なすべてをその環境から分離するための方法です。Docker社は、コンテナをあらゆるOS上で動作させることができるDocker Engineを発表し、コンテナの概念を普及させました。そのエンジンのおかげで、コンテナはLinux、macOS、Windows上で統一的に実行できるようになりました。

コンテナはDockerエンジン上で単一のプロセスとして実行されます。

コンテナは、与えられたユーザー空間の中で、独自の分離されたプロセスで実行されます。これは、独自の完全なオペレーティングシステムを必要とする仮想マシンと比較して、実行に必要なリソースを大幅に削減できる利点があります。そのため、アプリケーションをコンテナとして実行する場合の総コストは、仮想マシン上で実行する場合よりもはるかに小さくなります。

また、コンテナ技術は、ソフトウェア開発におけるマイクロサービス革命の中核を担っています。その小さなフットプリントにより、自己完結し分離された複数のアプリケーションを簡単に実行することができます。また、コンテナによって、小さなマイクロサービスを並行して実行させることも可能になります。Kubernetesのようなオーケストレーションツールを使えば、固有のマイクロサービスをダウンタイムなしに簡単にアップグレードすることも可能です。

コンテナを使用するもう1つの利点は、その性質上、エフェメラルであることです。コンテナを再起動すると、常に同じ状態になり、実行中に発生した可能性のある変更はすべてなくなります。これは、アプリケーションが常に安定して動作することを保証するために素晴らしいことです。また、入力されたデータはすべて削除され、テスト実行は毎回新しく始まるので、システム上でテストを行うソフトウェア開発者にとっても優れた機能です。

一方、データベースのコンテナを実行する場合のように、データの保存が必要な場合は、やや困難な場合があります。データの保存が必要な場合は、コンテナからアクセス可能なボリュームをマウントする必要がある。Kubernetesなどのツールを使えば、永続的なボリュームをマウントすることができる。

## Can MongoDB Run in a Docker Container?
MongoDBはコンテナで動作させることができます。Docker Hubで公開されている公式イメージには、MongoDBのコミュニティ版が含まれており、Dockerチームによってメンテナンスされています。このイメージは開発環境で使用することができます。本番環境では、EnterpriseバージョンのMongoDBまたはMongoDB Atlasを搭載したカスタムビルドのコンテナを推奨します。

MongoDB Enterprise をコンテナで実行したい場合は、MongoDB のドキュメントにある指示に従ってカスタムイメージを構築する必要があります。

## How to Use MongoDB With Docker
コンテナ環境で MongoDB を使うには、さまざまな方法があります。以下のセクションでは、MongoDB を使ってコンテナを実行・管理するための方法をいくつか紹介します。

## Prerequisites
以下の手順を実行するためには、Docker Desktopをインストールする必要があります。お使いのオペレーティングシステムに対応したインストール方法は、同社のウェブサイトで確認できます。

## Running MongoDB as a Docker Container
Dockerを使ってMongoDBコンテナを起動するには、次のコマンドを実行します。

```bash
docker run --name mongodb -d mongo
```

このコマンドは、利用可能な最新バージョンを実行している MongoDB サーバーをデタッチドモードで (バックグラウンドプロセスとして) 起動します。ベストプラクティスとして、一貫性を保つために MongoDB のバージョンを指定するときはタグを使うことをおすすめします。

ローカルで動作している別のアプリケーションから MongoDB サーバーにアクセスする必要がある場合は、 -p 引数でポートを公開する必要があります。

```bash
docker run --name mongodb -d -p 27017:27017 mongo
```

この方法を使うと、mongodb://localhost:27017のMongoDBインスタンスに接続できるようになります。データを視覚化して分析するための MongoDB の GUI である Compass で試すことができます。

そのコンテナのライフサイクルの一部として作成されたデータは、コンテナが削除されると破棄されます。ローカルマシンにデータを永続化させたい場合は、-v引数でボリュームをマウントします。

```bash
docker run --name mongodb -d -v YOUR_LOCAL_DIR:/data/db mongo
```

アプリケーションがコンテナ自体の中で動いている場合は、 --network を使ってアプリケーションと同じ Docker ネットワークの一部として MongoDB を実行することができます。この方法では、ネットワーク内の他のコンテナ化されたアプリケーションから mongodb://mongodb:27017 にある MongoDB に接続することになります。

```bash
docker run --name mongodb -d --network mynetwork mongo
```

MongoDB をルートユーザーで初期化するには、環境変数 MONGO_INITDB_ROOT_USERNAME と MONGO_INITDB_ROOT_PASSWORD を使用します。これらの環境変数は、指定したユーザー名とパスワードでルート権限を持つユーザーを作成します。

```bash
docker run --name mongodb -d -e MONGO_INITDB_ROOT_USERNAME=AzureDiamond -e MONGO_INITDB_ROOT_PASSWORD=hunter2 mongo
```

これらの国旗は、必要に応じて組み合わせることができることを覚えておいてください。

## Connecting to MongoDB from Another Docker Container
多くの場合、アプリケーションはコンテナ内で実行され、そのコンテナの外側で実行されているデータベースに接続する必要があります。そのための最良の方法は、環境変数を使用することです。環境変数を使用することで、コンテナを実行する場所によって接続文字列を異なる値に設定することができます。

例えば、同じコンテナを本番環境と開発サーバーの両方で動作させることができます。その場合、運用環境のアプリケーションは MongoDB Atlas あるいはコンテナ化された MongoDB Enterprise サーバーに接続し、開発環境のインスタンスはローカルの MongoDB インスタンスに接続することになるでしょう。サーバーのアドレスを示す接続文字列は、docker run コマンドで環境変数として渡されます。

```bash
docker run -d --name MYAPP -e MONGODB_CONNSTRING=mongodb+srv://username:password@clusterURL MYAPP:1.0
```

あなたのコードベースでは、以下のコードで環境変数を読み取ることができます。

Java

```java
String connString = System.getenv("MONGODB_CONNSTRING");
```

環境変数を使うと、コンテナ内で動いているアプリケーションからクラウドベースやローカルに関わらず任意の MongoDB インスタンスに接続できるようになります。この方法については MongoDB Developer Hub にある完全なチュートリアルを見ればわかるでしょう。

## Using MongoDB with Docker Compose
アプリケーションと MongoDB コンテナの両方が同じマシン上で動作している場合、Docker Compose を使ってそれらを一緒に起動したり停止したりすることができます。Docker Compose は、MongoDB Enterprise や MongoDB Atlas の全機能を必要としない開発環境やテスト環境に向いています。

docker-compose.yaml ファイルに、アプリケーションの一部となるすべてのコンテナを記述します。コンテナのひとつは MongoDB サーバーです。ベストプラクティスとして、前のセクションで説明したように、接続文字列を環境変数としてアプリケーションに渡します。

```yml
version: '3'
services:
  myapplication:
    image: myapp:1.0
    environment:
      - MONGODB_CONNSTRING=mongodb://AzureDiamond:hunter2@mongodb
    ports:
      - 3000:3000
  mongodb:
    image: mongo:5.0
    environment:
      - MONGO_INITDB_ROOT_USERNAME=AzureDiamond
      - MONGO_INITDB_ROOT_PASSWORD=hunter2
```

ファイルが存在するディレクトリから、docker-composeコマンドを実行します。

```bash
docker-compose up
```

このコマンドは、アプリケーションとローカルのMongoDBインスタンスの両方を起動します。

## Managing MongoDB from a Container
MongoDB サーバーの管理やデータのアクセス、インポート、エクスポートを行うには、もうひとつの MongoDB コンテナを使用します。

MongoDB Atlas サーバーへの Mongo Shell セッションを開くには、mongosh を使ってクラスタの URL を指定します。

```bash
docker run -it mongo:5.0 mongosh "
"mongo+srv://username:password@clusterURL/database"
```

mongoexport ツールを使って既存のコレクションを .json ファイルにエクスポートしたい場合は、別の MongoDB コンテナからこのコマンドを使用します。結果の JSON ファイルにアクセスできるようにするには、ボリュームをマウントする必要があります。

```bash
docker run -it -v $(pwd):/tmp mongo:5.0 mongoexport --collection=COLLECTION --out=/tmp/COLLECTION.json "mongo+srv://username:password@clusterURL/database"
```

データをコレクションにインポートする必要がある場合は、mongoimport ツールを使います。これも mongo イメージから利用可能です。この場合も、コンテナ内からローカルマシンに保存されているファイルにアクセスするには、ボリュームをマウントする必要があります。

```bash
docker run -it -v $(pwd):/tmp mongo:5.0 mongoimport --drop --collection=COLLECTION "mongodb+srv://user:password@clusterURL/database" /tmp/COLLECTION.json
```

MongoDBのインストール時にパッケージされている他のツールも、同じ方法でアクセスできます。

## What Are the Benefits of Using Docker?
ソフトウェア開発者として日々の生活の一部としてコンテナを使用することには、多くの利点があります。

- 一貫性。コンテナ技術を使用することで、チームの全員がまったく同じランタイムと設定を使用することを保証できます。また、本番環境と開発環境の一貫性が保たれるため、デプロイの摩擦が大幅に軽減されます。
- 軽量。Dockerコンテナは、仮想マシンと比較して、起動が早く、消費するリソースも最小限です。
- エフェメラル（儚い）。コンテナ・ファイル・システムのいかなる変更も、終了時に破棄されます。この無常性により、起動のたびに新鮮な環境を確保できます。

## Docker and MongoDB Atlas
DockerをMongoDBと一緒に使うには、MongoDBが提供するDatabase-as-a-ServiceであるMongoDB Atlasに接続するコンテナ化されたアプリケーションが最適です。開発環境に適した無料ティアが用意されています。この無料ティアを使えば、チームでデータベースを共有して共有テストデータセットを使ったり、チームメンバーごとに個別のテストデータベースを用意したりすることも簡単にできる。

アプリケーションの残りの部分はコンテナで実行し、すべての環境での一貫性を確保することができます。アプリケーションは、環境変数で渡される接続文字列を使って MongoDB に接続します。

接続文字列環境変数でアトラスに接続するコンテナ。

MongoDB Atlas を使えば、システムを構成するさまざまなマイクロサービスをすべて同じ共有データベースに接続することが簡単にできます。また、MongoDB Atlas は、レプリカセットのセットアップや設定、管理の手間をかけずに MongoDB クラスターを運用できるメリットもすべて備えています。

## Next Steps
この記事では、コンテナはあらゆるオペレーティングシステムやインフラストラクチャで一貫した方法でアプリケーションを実行する分離されたプロセスであることを学びました。また、MongoDB を実行するためにコンテナを起動する方法と、コンテナ化したアプリケーションから MongoDB に接続する方法についても学びました。

MongoDB Atlas はコンテナ化されたアプリケーションで簡単に使えるので、あなたやあなたのチームがデータにアクセスするのが簡単になります。MongoDB とコンテナについては、次のリソースでさらに詳しく知ることができます。
