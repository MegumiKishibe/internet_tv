# **DBセットアップマニュアル**
用意したテーブル設計を元に実際にデータベースを作成する手順を３ステップに分けて説明します。

#### ⚠️前提条件
- MySQLがインストール済みであること
- MySQLサーバーが起動していること
- rootユーザーでログインできること
- データベース名:internet_tv_db
- 使用するテーブルはtable_definition.txtを確認すること
- データ設計までの知識を有していること

---

## **Step1: データベース作成**
**1-1 ターミナルでmysqlを起動させ、ログインする**
- 起動コマンド
```bash
brew services start mysql
```
- ログインコマンド
```bash
mysql -u root
```
ログインに成功すると、プロンプトが`mysql>`に変わります。

**1-2 新しいデータベースを作成する**
```sql 
CREATE DATABASE internet_tv_db 
CHARACTER SET utf8mb4 
COLLATE utf8mb4_general_ci;
```

作成に成功すると`Query OK, 1 row affected (0.027 sec)`などと
要求(query)が成功したことを知らせる言葉が返ってきます。

### <解説>
- `CREATE DATABASE データベース名`
    新しいデータベースを名前をつけて作成します。
- `CHARACTER SET utf8mb4`
    文字コードをUTF-8の拡張版に設定し、絵文字や多言語を扱える様にします。
- `COLLATE utf8mb4_general_ci;`
    文字列の比較ルールを設定します。(照合順序)
    ciはcase-insensitive(大文字小文字を区別しない)の意味。

**1-3 データベースが作成できているか確認してみる**
`mysql> SHOW DATABASES;`コマンドを実行します。
```sql
SHOW DATABASES;
```
```text
+--------------------+
| Database           |
+--------------------+
| employees          |
| information_schema |
| internet_tv_db     |
| mysql              |
| performance_schema |
| route_db           |
| sys                |
+--------------------+
7 rows in set (0.018 sec)
```
`internet_tv_db`が一覧に表示されており、ちゃんと作成されていることが確認できました。


では、次のステップでテーブルを作成していきましょう！

## **Step2: テーブル構築**
**2-1 編集するデータベースを宣言する**
mysql>
```sql
USE internet_tv_db;
```
```text
Database changed
```

### <解説>
- `USE データベース名`
    →今から編集するデータベース名を宣言し、切り替えさせます。
- `Database changed`
    →無事に宣言したデータベースが存在し切り替えることができると表示されるメッセージです。


**2-2 テーブルを作成する**
table_definition.txtを参考にテーブル設計からテーブルを作成していきます。
Query例))
`mysql>`
```sql
 CREATE TABLE Channels (
    -> channel_id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    -> channel_no INT NOT NULL
    -> );
```
```text
Query OK, 0 rows affected (0.024 sec)
```

**<解説>**
- `CREATE TABLE テーブル名`
新しいテーブルを作成します。ここではテーブル名を`Channels`にしています。
- `channel_id INT NOT NULL AUTO_INCREMENT PRIMARY KEY`
channel_idカラムに対して以下の条件を設定しています。
- `INT`整数型
- `NOT NULL`未入力禁止(必須入力)
- `AUTO_INCREMENT`自動採番
- `PRIMARY KEY`主キー

**2-3 テーブルができているか確認する**
`SHOW TABLES;`コマンドを実行します。
```sql
mysql> SHOW TABLES;
``` 
```text
+--------------------------+
| Tables_in_internet_tv_db |
+--------------------------+
| Channels                 |
+--------------------------+
1 row in set (0.002 sec)
```
`Channels`がinternet_tv_dbのテーブル一覧に表示されており、作成が成功したことが確認できました。

**2-4 テーブルの中身を確認する**
`mysql> DESCRIBE Channels;` コマンドを実行します。
```sql
DESCRIBE Channels;
```
```text
+------------+------+------+-----+---------+----------------+
| Field      | Type | Null | Key | Default | Extra          |
+------------+------+------+-----+---------+----------------+
| channel_id | int  | NO   | PRI | NULL    | auto_increment |
| channel_no | int  | NO   |     | NULL    |                |
+------------+------+------+-----+---------+----------------+
2 rows in set (0.007 sec)
```
要求した通りの設定になっていることが確認できました。


**2-5 上記の要領で table_definition.txt に記載された全てのテーブルを作成する**

【手順の繰り返し例】
1. table_definition.txt 内の次のテーブル定義を確認する
2. `CREATE TABLE` 文を実行する
3. `SHOW TABLES;` で一覧に追加されたことを確認する
4. `DESCRIBE テーブル名;` でカラム定義を確認する
5. 次のテーブル定義に進む

この 1〜5 を全テーブル分繰り返す


## カラム定義に使われる項目リスト(🔰)
**データ型**
```sql
+-------------+---------------------------------+
|INT / INTEGER|整数                              |
|BIGINT       |大きい整数(膨大な数値を扱う時使用)  　 |
|VARCHAR(100) |文字列(100)には設定したい文字数を入れる|
|DATE         |日付(YYYY-MM-DD)                  |
|DATETIME     |日付＋時刻(YYYY-MM-DD HH:MM:SS)    |
+-------------+---------------------------------+
```
**制約**
```sql
+--------------+------------------------------------+
|NOT NULL      |NULLを許可しない（必須項目）         　　|
|AUTO_INCREMENT|自動採番(主キーに付けるのが一般的)     　 |
|PRIMARY KEY   |主キー(テーブル内で一意、NULL不可)       |
|UNIQUE        |値の重複を許可しない(NULLは1つ許可される) |
|DEFAULT       |デフォルト値を設定                  　　|
|FOREIGN KEY   |外部キー制約(別テーブルの主キーと関連付け) |
+--------------+------------------------------------+
```


## **Step3: サンプルデータ入力**
**3-1作成したテーブルに実際にデータを入れてみます。**
今回はChatGPTにサンプルデータを作成してもらいました。
データの入力コマンドは`INSERT INTO`を使います。
例))
```sql
INSERT INTO Channels (channel_id, channel_no) VALUES
(1, 101), 
(2, 102), 
(3, 103); 
```

**<解説>**
- `INSERT INTO`コマンドを実行します。
- テーブル名に（カラム１,カラム２）の順番でデータを入力します。

```sql
INSERT INTO テーブル名 (カラム１, カラム２) VALUES
(カラム1, カラム2),
(カラム1, カラム2),
(カラム1, カラム2);
```

**3-2実行結果を確認します。**
`SELECT * FROM Channels;`コマンドを実行します。

```sql
 SELECT * FROM Channels;
 ```
 ```text
+------------+------------+
| channel_id | channel_no |
+------------+------------+
|          1 |        101 |
|          2 |        102 |
|          3 |        103 |
+------------+------------+
3 rows in set (0.000 sec)
```
入力した通りのデータが入っていることが確認できました。
