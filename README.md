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
* 代表クエリ
* 動作確認チェックリスト
* ライセンス

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
├─ er_diagram.txt             # ER 図メモ/PlantUML 等（任意）
├─ 1NF.txt
├─ 2NF.txt
├─ 3NF.txt                    # 正規化メモ
├─ table_definition.txt       # DDL（メイン）
├─ table_definitions.txt      # ※重複がある場合はどちらかに統一
├─ sample_data.txt            # サンプルデータ（INSERT 文）
├─ db_setup_manual.md         # セットアップ手順
└─ step3.md                   # データ投入やクエリの解説（任意）
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

## 代表クエリ

### 1) 本日の番組表

```sql
SELECT
  c.name           AS channel_name,
  t.start_time     AS start_at,
  t.end_time       AS end_at,
  s.season_no      AS season_no,
  e.title          AS episode_title,
  e.description    AS episode_description
FROM Timeslots      AS t
JOIN Channels       AS c  ON c.channel_id   = t.channel_id
JOIN Programslots   AS ps ON ps.timeslot_id = t.timeslot_id
JOIN Programs       AS p  ON p.program_id   = ps.program_id
LEFT JOIN Seasons   AS s  ON s.season_id    = p.season_id
LEFT JOIN Episodes  AS e  ON e.episode_id   = ps.episode_id
WHERE DATE(t.start_time) = CURDATE()
ORDER BY c.name, t.start_time;
```

### 2) チャンネル別の番組枠数

```sql
SELECT
  c.channel_id,
  c.name,
  COUNT(t.timeslot_id) AS timeslots
FROM Channels AS c
LEFT JOIN Timeslots AS t ON t.channel_id = c.channel_id
GROUP BY c.channel_id, c.name
ORDER BY timeslots DESC, c.channel_id;
```

### 3) 視聴数ランキング（エピソード単位）

```sql
SELECT
  e.episode_id,
  e.title,
  e.view_count
FROM Episodes e
ORDER BY e.view_count DESC
LIMIT 20;
```

---

## 動作確認チェックリスト

* `SHOW TABLES;` で 8 テーブルが表示される
* `SELECT COUNT(*) FROM Channels;` が 0 以上
* 代表クエリがエラーなく実行できる

---

## ライセンス

未設定。必要に応じて `LICENSE` を追加する。
