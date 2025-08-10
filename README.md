# Internet TV Database

インターネットTVの番組編成・番組情報を管理するサンプルデータベース。
MySQL を用いたスキーマ、サンプルデータ、代表的な SQL クエリを含む。

---

## 目次

* 特徴
* リポジトリ構成
* 前提条件
* セットアップ
* データモデル（ER 図）
* 代表テーブル
* 動作確認チェックリスト

---

## 特徴

* 第3正規形を意識したスキーマ（Channel / Timeslot / Programslot / Program / Season / Episode / Genre / Program\_Genre）
* Programslot による複数チャンネル・再放送の表現
* サンプルデータと代表クエリを同梱

---

## リポジトリ構成

```
.
├─ README.md
├─ docs/
│  └─ er_diagram.mmd          # Mermaid 図（別ファイル）
├─ entities.txt               # エンティティ一覧
├─ 1NF.txt
├─ 2NF.txt
├─ 3NF.txt                    
├─ table_definition.md       
├─ sample_data.txt            # サンプルデータ（INSERT 文）
├─ db_setup_manual.md         # セットアップ手順
└─ step3.md                   # データ投入やクエリの解説
```

---

## 前提条件

* MySQL 8.0 以降
* `mysql` クライアントが使用可能であること

---

## セットアップ

1. データベース作成

   ```sql
   CREATE DATABASE internet_tv_db
     DEFAULT CHARACTER SET utf8mb4
     DEFAULT COLLATE utf8mb4_0900_ai_ci;
   ```
2. 接続と選択

   ```sql
   mysql -u root -p
   USE internet_tv_db;
   ```
3. テーブル作成（DDL 実行）

   ```sql
   SOURCE table_definition.txt;
   ```
4. サンプルデータ投入

   ```sql
   SOURCE sample_data.txt;
   ```
5. 確認

   ```sql
   SHOW TABLES;
   SELECT COUNT(*) FROM Channels;
   ```

---

## データモデル（ER 図）

* 図ファイル: [docs/er\_diagram.mmd](docs/er_diagram.mmd)
* 参考メモ: `er_diagram.txt`

---

## 代表テーブル

（詳細は `table_definition.txt` を参照）

* **Channels**: `channel_id (PK)`, `channel_no`, `name`
* **Timeslots**: `timeslot_id (PK)`, `channel_id (FK)`, `start_time`, `end_time`
* **Programslots**: `programslot_id (PK)`, `timeslot_id (FK)`, `program_id (FK)`, `episode_id (FK)`, `view_count`
* **Programs**: `program_id (PK)`, `series_no (NULL 可)`, `season_id (FK)`, `title`, `description`
* **Seasons**: `season_id (PK)`, `season_no`
* **Episodes**: `episode_id (PK)`, `season_id (FK, NULL 可)`, `title`, `description`, `video_duration`, `release_date`, `view_count`
* **Genres**: `genre_id (PK)`, `name`
* **Program\_Genre**: `program_genre_id (PK)`, `program_id (FK)`, `genre_id (FK)`

---

## 動作確認チェックリスト

* `SHOW TABLES;` で 8 テーブルが表示される
* `SELECT COUNT(*) FROM Channels;` が 0 以上
* 代表クエリがエラーなく実行できる

---