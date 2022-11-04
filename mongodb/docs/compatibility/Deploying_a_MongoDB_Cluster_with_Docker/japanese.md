# Spring Boot Integration with MongoDB Tutorial
このチュートリアルでは、Spring Boot を使って MongoDB Atlas クラスタからデータにアクセスします。このチュートリアルに従うには、MongoDB Atlas にサインインする必要があります。

Spring Bootは、自動構成されたマイクロサービスベースのWebフレームワークで、セキュリティとデータベースアクセスのための組み込み機能を提供します。

Spring bootを使えば、（後述するように）あまり多くの設定を変更することなく、スタンドアロンアプリケーションを素早く作成することができます。MongoDBは、データの保存と取り出しが簡単にできるため、最も人気のあるNoSQLデータベースです。Spring BootとMongoDBを組み合わせることで、高速で安全、信頼性が高く、開発時間を最小限に抑えたアプリケーションを実現できます。

このチュートリアルでは、Spring Data MongoDB APIを使用して、Spring BootとMongoDBを統合する方法を説明します。

## How to Use Spring Boot with MongoDB
Spring Bootはプロダクションレディのアプリケーションを素早く作成します。MongoDB と Spring Boot は MongoTemplate クラスと MongoRepository インターフェイスを使ってやりとりします。

MongoTemplate - MongoTemplate はすぐに使える API 一式を実装しています。更新や集約などの操作に適しており、MongoTemplate はカスタムクエリをより細かく制御できます。
MongoRepository - MongoRepository は、ドキュメントのすべてのフィールドや多くのフィールドを含む基本的なクエリに使われます。たとえば、データの作成、ドキュメントの閲覧などです。
両方のアプローチを使った Spring Boot MongoDB の設定は、数行のコードで済みます。

## Getting Started with MongoDB and Spring Boot
Springは、JavaのWebアプリケーションのためのアプリケーションフレームワークです。MVC (Model-View-Controller) フレームワークをベースにしている。Springの依存性注入は、データベースアクセス、セキュリティ、初期化などの機能を処理し、開発者はビジネスロジックに集中することができます。

Spring Bootは、主にREST APIのためにSpringフレームワークの上に構築されています。Spring Bootは、ほとんど設定を必要としない。4つのレイヤーを持ちます。

- プレゼンテーション層 - MVCフレームワークのビュー部分で、フロントエンドを処理します。
- ビジネス層 - すべてのビジネスロジックとバリデーションが行われるコントローラです。
- 永続化レイヤー - このレイヤーは、ビジネスオブジェクトをデータベースオブジェクトに変換します。
- データベース層 - 実際のCRUD（Create, Read, Update, Delete）操作はここで行われる。

MongoDBは高速で、大量の構造化・非構造化データを扱えるため、Webアプリケーションに最適なデータベースといえます。Springフレームワークは強力なコネクターを提供し、MongoDBで簡単にデータベース操作を行うことができます。MongoDBではデータはBSONオブジェクトとして保存されるので、データの検索が簡単になります。

このSpring BootとMongoDBのサンプルチュートリアルでは、PersistenceとDatabaseのレイヤーにだけ注目します。CRUD操作に焦点を当てたいので、統合開発環境(IDE)からプログラムを実行することにします。Spring BootとMongoDBを接続するために、Spring Boot MongoDBコンフィギュレーションを追加します。

## What We Will Build
あるユーザーのために、仮想的な食料品の買い物リストを作ってみましょう。以下のような操作を行います。

- Springアプリケーションで、ID、名前、数量、カテゴリーを持つ食料品アイテムPlain Old Java Object (POJO)を定義します。
- 次に、MongoRepositoryのパブリックインターフェースを使って、作成、読み込み、更新、削除（CRUD）操作を実行します。
- 最後に、MongoTemplate クラスを使ってドキュメントを更新する別の方法を紹介します。

POJO と MongoDB ドキュメントのマッピングについては、Getting started with MongoDB and Java という記事をご覧ください。

## What We Need
必要なのは

- MongoDB Atlas クラスタ (まだアカウントをお持ちでない場合は、今すぐ無料でお試しください)
- Java 1.8
- Spring Initializr
- Maven (Eclipseの "Help -> Install new software "からもインストールできます)
- 必要なライブラリと依存関係のインポートを行うIDE。このプロジェクトでは、Eclipseを使用しました。

まず、Spring Initializrを使用して、以下の設定でSpring Bootプロジェクトを作成します。

Maven Project を選択し、言語は Java (8)、Spring Boot のバージョンは 2.5.3 とします。また、依存関係を追加します。ここでは、Spring WebとSpring Data MongoDBを追加しました。Spring Webは、Apache Tomcatサーバー、REST、Spring MVCをアプリケーションに組み込み、共通の依存関係をすべて一箇所で管理します。

このアプリケーションでは、MongoDB AtlasクラスタからデータにアクセスするためにSpring Data MongoDBの依存関係を使用します。

プロジェクトのメタデータを入力し(上の画像のように)、JARオプションを選択します。Spring Initializrを使用すると、pom.xmlファイルの作成に気を配ることができます。Mavenはpom.xmlを使用して、必要な依存関係をダウンロードします。

これで、すべての設定の準備が整いました。次に、Generateボタンをクリックすると、Spring Bootプロジェクトをブートストラップするために必要なすべてのファイルが生成されます。ブラウザは自動的にZIPファイルをダウンロードします。

ZIPファイルがダウンロードされたら、プロジェクトを解凍してください。IDEからプロジェクトを開いてください。というようなプロジェクト構成が表示されます。

追加した依存関係がpom.xmlにartifactIdとして存在することが確認できます。

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">

      ...
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

    ...

    </dependencies>
      ...

</project>
```

完全なコードはgithubを参照してください。

src/main/java フォルダにコンテンツを作成します。

## MongoDB Model Implementation
ここでいうモデルとは、POJO、つまりGroceryItemクラスのことです。

com.example.mdbspringboot.model というパッケージを作成し、GroceryItem.java というクラスを追加しましょう。

モデルで使用するコレクション名を設定するために@Documentというアノテーションを使用します。コレクションが存在しない場合は、MongoDBが作成します。

```java
@Document("groceryitems")
public class GroceryItem {

        @Id
        private String id;

        private String name;
        private int quantity;
        private String category;

        public GroceryItem(String id, String name, int quantity, String category) {
            super();
            this.id = id;
            this.name = name;
            this.quantity = quantity;
            this.category = category;
        }
}
```

Eclipse の Source -> Generate Getters and Setters オプションを使うと、このコードのゲッターとセッターを作ることができます。MongoDB ドキュメントの主キー _id を @Id アノテーションで指定します。何も指定しないと、MongoDB はドキュメントを作成するときに _id フィールドを生成します。

## Spring Boot MongoDB API Implementation
API の実装は、リポジトリで行われます。これはモデルとデータベースの間のリンクとして機能し、CRUD操作のためのすべてのメソッドを持ちます。

com.example.mdbspringboot.repository というパッケージを作成して、リポジトリのファイルをすべて保存することにしましょう。

まず MongoRepository インターフェースを継承した ItemRepository パブリックインターフェースを作成します。

```java
public interface ItemRepository extends MongoRepository<GroceryItem, String> {

    @Query("{name:'?0'}")
    GroceryItem findItemByName(String name);

    @Query(value="{category:'?0'}", fields="{'name' : 1, 'quantity' : 1}")
    List<GroceryItem> findAll(String category);

    public long count();

}
```

最初のメソッド findItemByName は、クエリのパラメータ、つまりクエリをフィルタリングするフィールドを必要とします。これは、@Query というアノテーションで指定します。2番目のメソッドは、カテゴリフィールドを使用して、特定のカテゴリのすべての項目を取得します。クエリのレスポンスには、フィールドの名前と数量だけを投影したいので、これらのフィールドを 1 に設定します。メソッドcount()はそのまま再利用します。

## MongoDB and Spring Boot CRUD Examples
これで Spring Application を作成し、メソッドを実行して何が起こるかを確認する準備ができました。

MongoDB Atlasに接続するために、src/main/resourcesフォルダのapplication.propertiesファイルに接続文字列を指定します。クラスタの接続文字列はAtlasのUIで確認することができます。他のファイルに接続関連のコードを記述する必要はありません。Spring Bootがデータベース接続を代行してくれます。

```java
spring.data.mongodb.uri=mongodb+srv://<username>:<pwd>@<cluster>.mongodb.net/mygrocerylist
spring.data.mongodb.database=mygrocerylist
```

ここでデータベース名も指定しています。もしデータベースが存在しなければ、MongoDB がデータベースを作成します。

この Spring Boot MongoDB の例では、Controller と View は使いません。CommandLineRunner を使って、出力をコンソールで見ることにします。

メインクラス MdbSpringBootApplication.java をルートパッケージ com.example.mdbspringboot.MongoDB に作成し、MdbSpringBootApplication.java をルートパッケージに作成します。

```java
@SpringBootApplication
@EnableMongoRepositories
public class MdbSpringBootApplication implements CommandLineRunner{

    @Autowired
    ItemRepository groceryItemRepo;

    public static void main(String[] args) {
        SpringApplication.run(MdbSpringBootApplication.class, args);
    }
```

MdbSpringBootApplicationクラスは、Springアプリケーションを実行するためにCommandLineRunnerインターフェースを実装しています。ItemRepositoryはAutowiredで、Springが自動的に見つけることができるようになっています。Springは@SpringBootApplicationアノテーションを使ってApplication Contextを初期化します。また、@EnableMongoRepositoriesを使ってMongo Repositoriesを有効にしています。これで、プロジェクトの構造は以下のようになります。

それでは、CRUD操作のために、メインクラスにリポジトリのメソッドを追加してみましょう。

## Create operation using Spring Boot MongoDB
新しいドキュメントを作成するには、save メソッドを使用します。保存メソッドは MongoRepository インターフェイスを実装した SimpleMongoRepository クラスで利用できます。ItemRepository インターフェースは MongoRepository.ItemRepository クラスを継承しています。

saveメソッドは、GroceryItemオブジェクトをパラメータとして受け取ります。ここでは、5つのグローサリーアイテム（ドキュメント）を作成し、saveメソッドを使ってMongoDBに保存します。

```java
//CREATE
    void createGroceryItems() {
        System.out.println("Data creation started...");
        groceryItemRepo.save(new GroceryItem("Whole Wheat Biscuit", "Whole Wheat Biscuit", 5, "snacks"));
        groceryItemRepo.save(new GroceryItem("Kodo Millet", "XYZ Kodo Millet healthy", 2, "millets"));
        groceryItemRepo.save(new GroceryItem("Dried Red Chilli", "Dried Whole Red Chilli", 2, "spices"));
        groceryItemRepo.save(new GroceryItem("Pearl Millet", "Healthy Pearl Millet", 1, "millets"));
        groceryItemRepo.save(new GroceryItem("Cheese Crackers", "Bonny Cheese Crackers Plain", 6, "snacks"));
        System.out.println("Data creation complete...");
    }
```

## Read operations using Spring Boot MongoDB
このアプリケーションでは、4つの異なる読み取り操作を行っています。

- findAll() メソッドを使用して、すべてのドキュメント (食料品) を取得します。
- findItemByName メソッドを使用して、その名前フィールドによって単一のアイテム (ドキュメント) を取得します。
- カテゴリに基づいたアイテムのリストを取得します。
- アイテムの数を取得します。

```java
// READ
    // 1. Show all the data
     public void showAllGroceryItems() {

         groceryItemRepo.findAll().forEach(item -> System.out.println(getItemDetails(item)));
     }

     // 2. Get item by name
     public void getGroceryItemByName(String name) {
         System.out.println("Getting item by name: " + name);
         GroceryItem item = groceryItemRepo.findItemByName(name);
         System.out.println(getItemDetails(item));
     }

     // 3. Get name and quantity of a all items of a particular category
     public void getItemsByCategory(String category) {
         System.out.println("Getting items for the category " + category);
         List<GroceryItem> list = groceryItemRepo.findAll(category);

         list.forEach(item -> System.out.println("Name: " + item.getName() + ", Quantity: " + item.getQuantity()));
     }

     // 4. Get count of documents in the collection
     public void findCountOfGroceryItems() {
         long count = groceryItemRepo.count();
         System.out.println("Number of documents in the collection: " + count);
     }
```

読み出し操作の出力を読みやすい形式で表示するヘルパーメソッドを作成することができます。

```java
 // Print details in readable form

     public String getItemDetails(GroceryItem item) {

         System.out.println(
         "Item Name: " + item.getName() +
         ", \nQuantity: " + item.getQuantity() +
         ", \nItem Category: " + item.getCategory()
         );

         return "";
     }
```

## Update operation using Spring Boot MongoDB
例えば、食料品リストを「snacks」ではなく「munchies」に変更するとします。その場合、カテゴリが "snacks "の文書をすべて変更する必要がある。そのためには、まずカテゴリーが「snacks」の文書をすべて取得し、カテゴリーを「munchies」に設定し、すべての文書を保存しなければならない。

```java
public void updateCategoryName(String category) {

         // Change to this new value
         String newCategory = "munchies";

         // Find all the items with the category snacks
         List<GroceryItem> list = groceryItemRepo.findAll(category);

         list.forEach(item -> {
             // Update the category in each document
             item.setCategory(newCategory);
         });

         // Save all the items in database
         List<GroceryItem> itemsUpdated = groceryItemRepo.saveAll(list);

         if(itemsUpdated != null)
             System.out.println("Successfully updated " + itemsUpdated.size() + " items.");
     }
```

## Delete operation using Spring Boot MongoDB
もし、アイテムやカテゴリーを更新する代わりに、リストから食料品アイテムを削除したい場合は、それが可能です。あらかじめ定義されているdeleteByIdメソッドを使って、特定のIDを持つ食料品アイテムを削除することができます。

```java
// DELETE
     public void deleteGroceryItem(String id) {
         groceryItemRepo.deleteById(id);
         System.out.println("Item with id " + id + " deleted...");
     }
```

すべての項目を削除するには、groceryItemRepo.deleteAll();メソッドを使用します。すべての文書を削除しても、コレクションは削除されません。

## Putting the CRUD operations together
次に、上記のメソッドを呼び出すために、CommandLineRunner.run()メソッドを実装します。

```java
public void run(String... args) {

        System.out.println("-------------CREATE GROCERY ITEMS-------------------------------\n");

        createGroceryItems();

        System.out.println("\n----------------SHOW ALL GROCERY ITEMS---------------------------\n");

        showAllGroceryItems();

        System.out.println("\n--------------GET ITEM BY NAME-----------------------------------\n");

        getGroceryItemByName("Whole Wheat Biscuit");

        System.out.println("\n-----------GET ITEMS BY CATEGORY---------------------------------\n");

        getItemsByCategory("millets");

          System.out.println("\n-----------UPDATE CATEGORY NAME OF SNACKS CATEGORY----------------\n");

        updateCategoryName("snacks");

        System.out.println("\n----------DELETE A GROCERY ITEM----------------------------------\n");

        deleteGroceryItem("Kodo Millet");

        System.out.println("\n------------FINAL COUNT OF GROCERY ITEMS-------------------------\n");

        findCountOfGroceryItems();

        System.out.println("\n-------------------THANK YOU---------------------------");

    }
```

system.out文は、出力を美しくするためだけのものです。

以下は、このプログラムを実行したときに期待される出力です。

```java
-------------CREATE GROCERY ITEMS-------------------------------

Data creation started...
Data creation complete...

----------------SHOW ALL GROCERY ITEMS---------------------------

Item Name: Whole Wheat Biscuit,
Item Quantity: 5,
Item Category: snacks

Item Name: XYZ Kodo Millet healthy,
Item Quantity: 2,
Item Category: millets

Item Name: Dried Whole Red Chilli,
Item Quantity: 2,
Item Category: spices

Item Name: Healthy Pearl Millet,
Item Quantity: 1,
Item Category: millets

Item Name: Bonny Cheese Crackers Plain,
Item Quantity: 6,
Item Category: snacks


--------------GET ITEM BY NAME-----------------------------------

Getting item by name: Whole Wheat Biscuit
Item Name: Whole Wheat Biscuit,
Item Quantity: 5,
Item Category: snacks


-----------GET ITEMS BY CATEGORY---------------------------------

Getting items for the category millets
Name: XYZ Kodo Millet healthy, Quantity: 2
Name: Healthy Pearl Millet, Quantity: 1

-----------UPDATE CATEGORY NAME OF SNACKS CATEGORY----------------

Successfully updated 2 items.

----------DELETE A GROCERY ITEM----------------------------------

Item with id Kodo Millet deleted...

------------FINAL COUNT OF GROCERY ITEMS-------------------------

Number of documents in the collection = 4

-------------------THANK YOU---------------------------
```

## Update operation with Spring Boot MongoDB using MongoTemplate
特定のフィールドを使って更新操作を行うには、MongoTemplate クラスを使うこともできます。このクラスは org.springframework.data.mongodb.core.query パッケージに含まれる機能をそのまま提供します。何行もコードを書く必要はなく、一度のデータベース操作で更新が完了します。MongoTemplate を使えば、集約などのもっと複雑な処理もできます (このチュートリアルでは扱いません)。

MongoTemplate クラスを更新に使うには、更新クエリを作成するカスタムリポジトリを作成する必要があります。

それでは、食料品の数量を更新するメソッドを書いてみましょう。

インターフェイス CustomItemRepository を作成します。

```java
public interface CustomItemRepository {

    void updateItemQuantity(String name, float newQuantity);

}
```

インターフェースに必要なメソッドをいくつでも追加し、CustomItemRepositoryImplクラスで実装を提供することができます。

```java
@Component
public class CustomItemRepositoryImpl implements CustomItemRepository {

    @Autowired
    MongoTemplate mongoTemplate;

    public void updateItemQuantity(String name, float newQuantity) {
        Query query = new Query(Criteria.where("name").is(name));
        Update update = new Update();
        update.set("quantity", newQuantity);

        UpdateResult result = mongoTemplate.updateFirst(query, update, GroceryItem.class);

        if(result == null)
            System.out.println("No documents updated");
        else
            System.out.println(result.getModifiedCount() + " document(s) updated..");

    }

}
```

MongoTemplateは@Autowiredなので、Springはオブジェクトの依存関係をインジェクトします。また、@Componentアノテーションをつけることで、Spring自身がCustomItemRepositoryのインターフェイスを検出できるようになります。

次のステップは、メインクラスからこのメソッドを呼び出すことです。groceryItemRepoを宣言したのと同様に、customRepoを宣言しましょう。

```java
    @Autowired
    CustomItemRepository customRepo;
```

次に、customRepoメソッドを呼び出すメソッドをメインクラス内に記述します。

```java
// UPDATE
     public void updateItemQuantity(String name, float newQuantity) {
         System.out.println("Updating quantity for " + name);
         customRepo.updateItemQuantity(name, newQuantity);
     }
```

runメソッドに上記のメソッドを追加し、アプリケーションの実行時に呼び出すようにします。

```java
System.out.println("\n-----------UPDATE QUANTITY OF A GROCERY ITEM------------------------\n");

        updateItemQuantity("Bonny Cheese Crackers Plain", 10);
```

という結果が出るはずです。

```java
-----------UPDATE QUANTITY OF A GROCERY ITEM------------------------

Updating quantity for Bonny Cheese Crackers Plain
1 document(s) updated..
```

先ほど説明したように、MongoRepository では検索、設定、保存の 3 つの操作をしなければならなかったのに比べて、 1 回のデータベーストランザクションで更新ができます。MongoTemplate には updateMulti() メソッドもあり、複数のドキュメントを一度に更新することができます。

## It’s Easy to Connect MongoDB Atlas with Spring Boot
この記事を通して、MongoDBのSpring Boot連携が簡単にできることを実証しました。

- pom.xml にスターターデータ MongoDB の artifactid (Spring Initializr プロジェクトを作成するときに追加した依存関係) を記述する。
- application.propertiesファイルにMongoDBクラスタへの接続文字列を指定するためのプロパティを用意する

MongoDB Atlasへの接続は他のコードを必要としない。MongoDB アトラスは、どこにでもデータを保存してアクセスできる便利な方法です。このチュートリアルに沿って自分で作業したい場合は、 MongoDB Atlas のページにアクセスして始めましょう。この簡単なチュートリアルに沿って作業し、Spring BootとMongoDBの使い方を理解した方は、REST APIs with Java, Spring Boot, and MongoDBを読んで、高度な操作方法を学び、ビジネス (controller) とプレゼンテーション (view) 層も使ってSpring Bootアプリケーションを使うメリットを十分に味わってください。さらに、MongoDB Universityのコース「MongoDB for Java Developers」の全容を確認することもできます。

## FAQ
### How does MongoDB connect to Spring Boot?
MongoDB は MongoRepository インターフェースと MongoTemplate クラスの2つの方法で Spring Boot に接続することができます。MongoRepository は CrudRepository インターフェースを継承していて、基本的な CRUD 操作を実行するメソッドが含まれています。カスタムクエリやアグリゲーションを書いたり、 クエリフィルタをより細かく制御したりするには MongoTemplate クラスを使います。MongoTemplate クラスは、集約や検索、更新、upsert、インデックス、削除などの操作をサポートするインターフェイスを実装しています。

### Which database is best for Spring Boot?
SQLデータベースとNoSQLデータベースはどちらもSpring Bootに適しています。しかし、MongoDBのようなNoSQLデータベースは、ドキュメントベースの構造を埋め込んでいるため、いくつかの利点があります。ドキュメント構造は、データを記述し、トラバースするのにより自然な方法です。一緒に表示されたデータは一緒に残ります。SpringはMongoTemplateやMongoRepositoryといったコネクターも提供しており、MongoDBですべてのデータベース操作を実行できる。

### What is Spring Boot used for?
Spring Bootフレームワークは、デフォルトの構成でプロダクションレディのWebアプリケーションを作成するために使用されます。開発者は大規模なコードを書く必要がありません。Spring Bootは、開発時間を大幅に短縮します。Webアプリケーションによく使われる、以下のようなライブラリが自動的に追加されます。

- spring-webmvc
- tomcat
- validation-api

など、Webアプリケーションでよく使われるライブラリが自動的に追加され、依存関係の管理が容易になります。

また、Spring Bootはサーブレットコンテナを内蔵しています。pom.xmlにspring-boot-starter-webという依存関係を追加することで、スタンドアロンアプリケーションとしてJavaプログラムを実行できるようになります。

### What is a Spring Boot with MongoDB CRUD example?
Spring BootとMongoDBのCRUDの一般的な例としては、ユーザーの食料品の買い物リストが考えられます。ユーザーは以下のことをしたいかもしれません。

- 食料品リストにアイテムを追加する(作成する)。
- 例えば、間違って卵を追加してしまったので、卵を削除する。
- 例えば、牛乳を2パックではなく、4パック購入する。

CRUD 操作は MongoRepository と MongoTemplate を使って行われます。pom.xml に spring-boot-starter-data-mongodb という依存関係を追加することで、両方を使うことができるようになります。
