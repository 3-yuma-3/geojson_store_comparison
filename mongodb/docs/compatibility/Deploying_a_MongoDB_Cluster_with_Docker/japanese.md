# Deploying a MongoDB Cluster with Docker
## Using Docker to Deploy a MongoDB Cluster/Replica Set
MongoDBは、Webを意識して作られた汎用データベースです。特に、レプリカセットと呼ばれるクラスターで使用すると、高可用性を実現できます。レプリカセットはノードと呼ばれるMongoDBサーバーのグループで、データの同一コピーが格納されています。1台のサーバーに障害が発生すると、クラッシュしたサーバーが再起動するまでの間、残りの2台が負荷を補います。

MongoDB が提供するデータベース・アズ・ア・サービスである MongoDB Atlas で MongoDB インスタンスを作成すると、自動的にクラスターが作成され、最高の使い勝手が得られるようになります。

しかし、MongoDB のクラスターを実験する必要がある場合は、Docker を使って自分のパソコンにクラスターを作成することができます。このチュートリアルでは、あなた自身の MongoDB Docker クラスターを作成するために必要な手順を説明します。

## Benefits of Docker Deployment
MongoDB のクラスタを作るには、さまざまな方法があります。最も簡単なのは MongoDB Atlas を使う方法です。これは MongoDB が提供するデータベース・アズ・ア・サービスです。ほんの数クリックで、クラウドにホストされた無料のクラスターを作ることができます。

クラスターを試してみてその仕組みをもっとよく理解したい場合や、クラスターを使った開発環境が必要な場合は、自分のコンピューターに MongoDB をインストールしてレプリカセットをデプロイすることができます。

ラップトップにMongoDBをインストールしたくない場合は、Dockerを使用してクラスタを実行することができます。Dockerはコンテナを起動するためのアプリケーションで、アプリケーションとそれを実行するために必要なすべての依存関係を含むパッケージです。

Dockerを使用することは、MongoDBをローカルにインストールせずにレプリカセットを使い始めるための代替方法です。チームの全員が同じクラスタ構成を実行できるようにする便利な方法です。

同じ手法で Kubernetes 上に可用性の高いクラスターを展開することができます。

## Instructions
Dockerクラスタを作成する手順は以下の通りです。

1. Dockerネットワークを作成します。
2. MongoDBのインスタンスを3つ起動する。
3. Replica Set を起動する。

MongoDBクラスターを立ち上げたら、実際に実験してみましょう。

## Prerequisites
このチュートリアルでは、コンテナ用のランタイムを用意する必要があります。Dockerの公式サイトには、お使いのOSに合わせたインストール方法が掲載されています。

## Create a Docker Network
最初のステップは、Dockerネットワークを作成することです。このネットワークでは、このネットワークで動作している各コンテナを互いに参照できるようにします。ネットワークを作成するには、docker network createコマンドを実行します。

```bash
docker network create mongoCluster
```

mongoClusterのパラメータはネットワークの名前です。

コマンドを実行すると、先ほど作成したネットワークの ID が表示されます。

このコマンドは一度だけ実行する必要があることに注意してください。その後、コンテナを再起動すれば、このネットワークを再作成する必要はない。

## Start MongoDB Instances
これで、MongoDB を搭載した最初のコンテナを起動する準備ができました。コンテナを起動するには、docker run コマンドを使用します。

```bash
docker run -d --rm -p 27017:27017 --name mongo1 --network mongoCluster mongo:5 mongod --replSet myReplicaSet --bind_ip localhost,mongo1
```

ここでは、以下のパラメータを指定してコンテナを起動するようにdockerに指示します。

- -d は、このコンテナがデタッチドモード（バックグラウンド）で実行されることを示します。
- -p はポートマッピングを示します。あなたのマシンの27017番ポートに着信したリクエストは、コンテナ内の27017番ポートにリダイレクトされます。
- --name はコンテナの名前を示します。これがこのマシンのホスト名となる。
- --networkは、使用するDockerネットワークを示します。同じネットワークにあるすべてのコンテナは、お互いを見ることができます。
- mongo:5は、Dockerが使用するイメージです。このイメージは、MongoDB Communityサーバーのバージョン5（Dockerがメンテナンスしている）である。MongoDB Enterpriseのカスタムイメージを使用することもできます。

この命令の残りの部分は、コンテナが起動したら実行されるコマンドです。このコマンドは、レプリカセットの準備ができた新しい mongod インスタンスを作成します。

コマンドが正常に実行されると、コンテナ ID を表す長い 16 進数の文字列が表示されるはずです。他の2つのコンテナを起動します。この2つには別の名前と別のポートを使用する必要があります。

```bash
docker run -d --rm -p 27018:27017 --name mongo2 --network mongoCluster mongo:5 mongod --replSet myReplicaSet --bind_ip localhost,mongo2

docker run -d --rm -p 27019:27017 --name mongo3 --network mongoCluster mongo:5 mongod --replSet myReplicaSet --bind_ip localhost,mongo3
```

これで、MongoDB が動作するコンテナが 3 つ揃いました。docker ps を使って、これらのコンテナが動作していることを確認できます。

## Initiate the Replica Set
次のステップは、3つのメンバーで実際のレプリカセットを作成することです。これを行うには、MongoDB Shell を使う必要があります。このCLI (コマンドラインインターフェース) ツールは、デフォルトのMongoDBインストールで利用できますし、単独でインストールすることも可能です。ただし、ラップトップにこのツールがインストールされていない場合は、docker execコマンドでコンテナ内で利用できるmongoshを使用することも可能です。

```bash
docker exec -it mongo1 mongosh --eval "rs.initiate({
 _id: \"myReplicaSet\",
 members: [
   {_id: 0, host: \"mongo1\"},
   {_id: 1, host: \"mongo2\"},
   {_id: 2, host: \"mongo3\"}
 ]
})"
```

このコマンドは Docker に mongosh ツールを mongo1 という名前のコンテナ内で実行するよう指示します。mongosh はその後 rs.initiate() コマンドを評価してレプリカセットを開始しようとします。

rs.initiate() に渡す設定オブジェクトの一部として、レプリカセットの名前（この場合は myReplicaSet）と、レプリカセットの一部となるメンバーのリストを指定する必要があります。コンテナのホスト名には、docker runコマンドの--nameパラメータで指定したコンテナ名を使用する。レプリカセットを構成するために可能なオプションについては、ドキュメントを参照してください。

コマンドが正常に実行されると、mongosh CLI から MongoDB と Mongosh のバージョン番号を示すメッセージが表示され、その後に次のようなメッセージが表示されるはずです。

{ ok: 1 }

## Test and Verify the Replica Set
これでレプリカセットが動作するようになったはずです。すべてが正しく設定されたかどうかを確認したい場合は、 mongosh CLI ツールを使って rs.status() 命令を評価します。これでレプリカセットの状態やメンバーのリストがわかります。

```bash
docker exec -it mongo1 mongosh --eval "rs.status()"
```

また、MongoDB Compass を使ってクラスタに接続し、データベースを作成してドキュメントをいくつか追加することもできます。データはコンテナストレージ内に作成され、コンテナがホストシステムから削除されると破棄されることに注意してください。レプリカセットが動作していることを確認するには、コンテナの1つを docker stop で停止して、データベースからの読み込みを再試行します。

```bash
docker stop mongo1
```

データはまだそこにあります。mongo2 コンテナで rs.status() を使用すると、クラスタがまだ稼働していることが確認できます。

```bash
docker exec -it mongo2 mongosh --eval "rs.status()"
```

レプリカセットはまだ表示されていますが、最初のメンバーがダウンし、他の2つのメンバーのうち1つがプライマリノードとして選出されていることに注意してください。コンテナ mongo1 を再び起動すると、レプリカセットに戻りますが、セカンダリメンバーとして表示されるようになります。

## Next Steps
MongoDB クラスタとは何か、そして Docker を使って個人のパソコンで作る方法はわかったので、MongoDB Atlas で無料でクラスタを始めてみませんか? また、MongoDB University の m103 コースでは、レプリカセット管理の詳細など、基本的な MongoDB クラスタ管理について学ぶことができます。
