# **ステップ3**
以下のデータを抽出するクエリを書いてください。

## **1,よく見られているエピソードを知りたいです。**
エピソード視聴数トップ3のエピソードタイトルと視聴数を取得してください.

**1-1 Episodesテーブルに欲しい情報があると思うので、情報を見に行く。**
```sql
mysql> SELECT * FROM Episodes;
```
```text
+------------+-----------+----------------------------+-----------------------------+----------------+--------------+------------+
| episode_id | season_id | title                      | description                 | video_duration | release_date | view_count |
+------------+-----------+----------------------------+-----------------------------+----------------+--------------+------------+
|          1 |         1 | ドラマA 第1話              | 主人公の旅立ち              |           3600 | 2025-01-10   |      50000 |
|          2 |         1 | ドラマA 第2話              | 再会と別れ                  |           3700 | 2025-01-17   |      47000 |
|          3 |         2 | ドラマA2 第1話             | 新たな挑戦                  |           3650 | 2025-06-01   |      52000 |
|          4 |         3 | アニメB 第1話              | 異世界へようこそ            |           1500 | 2025-03-15   |      80000 |
|          5 |         4 | サッカー特集 第1話         | 開幕戦ダイジェスト          |           2700 | 2025-02-01   |      60000 |
+------------+-----------+----------------------------+-----------------------------+----------------+--------------+------------+
5 rows in set (0.001 sec)
```

**1-2 `Episodes`テーブルの`title`と`view_count`を抽出する。**

```sql
 SELECT
 title        AS episode_title,
   view_count   AS episode_views
 FROM Episodes
 ORDER BY view_count DESC, episode_id
 LIMIT 3;
 ```
```text
+----------------------------+---------------+
| episode_title              | episode_views |
+----------------------------+---------------+
| アニメB 第1話              |         80000 |
| サッカー特集 第1話         |         60000 |
| ドラマA2 第1話             |         52000 |
+----------------------------+---------------+
3 rows in set (0.001 sec)
```

**<解説>**
- `SELECT文`でデータを取得する。
- (AS episode_title,)それぞれ`episode_title`と`episode_views`と見やすいカラム名にネーミング。
- (From　Episodes)データのスタート(起点)はEpisodesからです。
- (ORDER BY　~ DESC,)view_countの大きい順に並べ替える。デフォルトは昇順（小さい→大きい／早い→遅い）。
- (episode_id)同率の場合をepisode_id順に並べる。
- (LIMIT 3;)上位３行に限定する


## **2,よく見られているエピソードの番組情報やシーズン情報も合わせて知りたいです。**
エピソード視聴数トップ3の番組タイトル、シーズン数、エピソード数、エピソードタイトル、視聴数を取得してください
- エピソードタイトル、視聴数、エピソード数はEpisodesから(e.title,e.view_count,e.episode_no)
- シーズン数はSeasonsから(s.season_no)
- 番組タイトルはProgramsから(p.title)
    それぞれ取得する。


```mysql
SELECT
  p.title        AS program_title,
  s.season_no    AS season_no,     
  e.episode_no   AS episode_no,    
  e.title        AS episode_title,
  e.view_count   AS episode_views
FROM Programslots ps
JOIN Episodes e ON e.episode_id = ps.episode_id
JOIN Programs p ON p.program_id = ps.program_id
LEFT JOIN Seasons s  ON s.season_id  = e.season_id
ORDER BY e.view_count DESC, e.episode_id
LIMIT 3;
```
```text
+-------------------------+-----------+------------+----------------------------+---------------+
| program_title           | season_no | episode_no | episode_title              | episode_views |
+-------------------------+-----------+------------+----------------------------+---------------+
| アニメB                 |         1 |          1 | アニメB 第1話              |         80000 |
| サッカー特集 2025       |         1 |          1 | サッカー特集 第1話         |         60000 |
| ドラマA S1              |         1 |          1 | ドラマA 第1話              |         50000 |
+-------------------------+-----------+------------+----------------------------+---------------+
3 rows in set (0.001 sec)
```

**<解説>**

- `SELECT文`でデータを取得する。
- (AS program_title,)それぞれ見やすいカラム名にネーミング。
- (FROM Programslots ps)データのスタート地点(起点)はProgramslots。今後はpsと呼ぶ。
- (JOIN)バラバラのテーブルを結合していく。
- (JOIN Episodes e ON e.episode_id = ps.episode_id)
    e.episode_idとpsのFKとして配置したepisode_id(ps.episode_id)を参照&結合。
- (JOIN Programs p ON p.program_id = ps.program_id)
    p.program_idとpsのFKとして配置したprogram_id(ps.program_id)を参照&結合。
    この時点で、`Programslots-Episodes-Programs`と左から順番に結合されている。
- (LEFT JOIN) JOINで結合済みの左辺`[Programslots-Episodes-Programs]`へ結合する。
- (LEFT JOIN Seasons s  ON s.season_id  = e.season_id)
    Seasonsを結合するために、s.season_idとEpisodesにFKとして配置したseason_id(e.season_id)を結合。
    これまで結合してきた`[Programslots-Episodes-Programs]`へ`Seasons`を新たに加えるイメージ。
    Seasonsにないデータは`NULL`が返される。
```sql
    Episodes（左）                 Seasons（右）
e.episode_id | e.season_id     s.season_id | s.season_no
-------------+------------     -------------+------------
1            | 10              10           | 1
2            | NULL            11           | 2
3            | 99              (← 99は存在しない)
```
- (ORDER BY　~ DESC,)view_countの大きい順に並べ替えて、
- (episode_id)同率の場合をepisode_id順に並べる。
- (LIMIT 3;)上位３行に限定する

以下、使用したテーブル内容
```text
Episodes
+----------------+------------+----+-------+-----------+--------------+
|Columns         |DATA TYPE   |NULL|KEY    |DEFAULT    |AUTO INCREMENT|
+----------------+------------+----+-------+-----------+--------------+
|episode_id      |INT         |    |PRIMARY|           |              |
|season_id       |INT         |    |FOREIGN|           |              |
|episode_no      |INT         |OK  |       |           |              |
|title           |VARCHAR(80) |    |UNIQUE |           |              |
|description     |VARCHAR(300)|OK  |       |ComingSoon!|              |
|video_duration  |BIGINT      |    |       |           |              |
|release_date    |DATE        |    |       |           |              |
|view_count      |INT         |    |       |           |              |
+----------------+------------+----+-------+-----------+--------------+
Seasons
+----------------+------------+----+-------+-----------+--------------+
|Columns         |DATA TYPE   |NULL|KEY    |DEFAULT    |AUTO INCREMENT|
+----------------+------------+----+-------+-----------+--------------+
|season_id       |INT         |    |PRIMARY|           |              |
|season_no       |INT         |    |       |           |              |
+----------------+------------+----+-------+-----------+--------------+
Programs
+----------------+------------+----+-------+-----------+--------------+
|Columns         |DATA TYPE   |NULL|KEY    |DEFAULT    |AUTO INCREMENT|
+----------------+------------+----+-------+-----------+--------------+
|program_id      |INT         |    |PRIMARY|           |YES           |
|series_no       |INT         |OK  |       |           |              |
|season_id       |INT         |    |FOREIGN|           |              |
|title           |VARCHAR(80) |    |UNIQUE |           |              |
|description     |VARCHAR(300)|OK  |       |ComingSoon!|              |
+----------------+------------+----+-------+-----------+--------------+
Programslots
+----------------+------------+----+-------+-----------+--------------+
|Columns         |DATA TYPE   |NULL|KEY    |DEFAULT    |AUTO INCREMENT|
+----------------+------------+----+-------+-----------+--------------+
|programslot_id  |INT         |    |PRIMARY|           |              |
|timeslot_id     |INT         |    |FOREIGN|           |              |
|episode_id      |INT         |    |FOREIGN|           |YES           |
|program_id      |INT         |    |FOREIGN|           |YES           |
|view_count      |INT         |    |       |           |              |
+----------------+------------+----+-------+-----------+--------------+
```


## **3,本日の番組表を表示するために、本日、どのチャンネルの、何時から、何の番組が放送されるのかを知りたいです。**
本日放送される全ての番組に対して、チャンネル名、放送開始時刻(日付+時間)、放送終了時刻、シーズン数、エピソード数、エピソードタイトル、エピソード詳細を取得してください。
なお、番組の開始時刻が本日のものを本日方法される番組とみなすものとします。
チャンネル名(Channels.name)
放送開始時刻(Timeslots.start_time)、放送終了時刻(Timeslots.end_time)
シーズン数(Season.season_no)
エピソード数(Episode.episode_no)、エピソードタイトル(Episode.title)、エピソード詳細(Episode.description)
からそれぞれ取得する。


```sql
SELECT
  c.name                    AS channel_name,                 
  t.start_time              AS start_at,                     
  t.end_time                AS end_at,                       
  s.season_no               AS season_no,                    
  e.episode_no              AS episode_no,                   
  e.title                   AS episode_title,                
  e.description             AS episode_description
FROM Timeslots t
JOIN Channels      c  ON c.channel_id    = t.channel_id
JOIN Programslots  ps ON ps.timeslot_id  = t.timeslot_id 
JOIN Episodes      e  ON e.episode_id    = ps.episode_id
LEFT JOIN Seasons  s  ON s.season_id     = e.season_id
WHERE DATE(t.start_time) = CURRENT_DATE
ORDER BY c.name, t.start_time;
```
```text
+--------------+---------------------+---------------------+-----------+------------+---------------+---------------------+
| channel_name | start_at            | end_at              | season_no | episode_no | episode_title | episode_description |
+--------------+---------------------+---------------------+-----------+------------+---------------+---------------------+
| スポーツ     | 2025-08-10 19:00:00 | 2025-08-10 20:00:00 |         1 |          2 | SQ S1E2       | 新キャラ登場        |
| ドラマ1      | 2025-08-10 19:00:00 | 2025-08-10 20:00:00 |         1 |          1 | SQ S1E1       | パイロット回        |
| ペット       | 2025-08-10 19:00:00 | 2025-08-10 20:00:00 |         0 |          1 | 夏の特番      | サマーSP            |
+--------------+---------------------+---------------------+-----------+------------+---------------+---------------------+
3 rows in set (0.001 sec)
```

**<解説>**
- **`SELECT文`**でデータを取得する。
- **(AS channel_name,)**
    それぞれ見やすいカラム名にネーミング。
- **(FROM Timeslots t)**
    データのスタート地点(起点)はTimeslots。今後はtと呼ぶ。
- **(JOIN)**
    バラバラのテーブルを結合していく。
- **(JOIN Channels c ON c.channel_id = t.channel_id)**
    c.channel_idとtのFKとして配置したchannel_id(t.channel_id)を参照&結合。 [Timeslots-Channels]
- **(JOIN Programslots  ps ON ps.timeslot_id  = t.timeslot_id )**
    ps.timeslot_idとtのPKとして配置したtimeslots_id(t.timeslot_id)を参照&結合。[Timeslots-Channels-Programslots]
- **(JOIN Episodes e ON e.episode_id = ps.episode_id)**
    e.episode_idとpsのFKとして配置したepisode_id(ps.episode_id)を参照&結合。[Timeslots-Channels-Programslots-Episodes]
- **(LEFT JOIN)**
    JOINで結合済みの左辺`[Timeslots-Channels-Programslots-Episodes]`へ結合する。
- **(LEFT JOIN Seasons s  ON s.season_id  = e.season_id)**
    Seasonsを結合するために、s.season_idとEpisodesにFKとして配置したseason_id(e.season_id)を結合。
    これまで結合してきた`[Timeslots-Channels-Programslots-Episodes]`へ`Seasons`を新たに加えるイメージ。
    Seasonsにないデータは`NULL`が返される。
- **(DATE)**
    日時型から“日付だけ”を切り出す関数。時刻部分を無視して日付ベースで比較したいときに使用する。
- **(WHERE DATE(t.start_time) = CURRENT_DATE)**
    `t.start_time` の日付部分が “今日” と等しい行だけに絞り込みます。
- **(ORDER BY)**
    順番に並べ替える。デフォルトは昇順（小さい→大きい／早い→遅い）。
- **(ORDER BY c.name, t.start_time;)**
    まずチャンネル名で昇順に並べ、その中で開始時刻が早い順に並べます。チャンネルごとの番組表が時間順で読める形になる。


以下、使用したテーブル内容

```text
Timeslots
+----------------+------------+----+-------+-----------+--------------+
|Columns         |DATA TYPE   |NULL|KEY    |DEFAULT    |AUTO INCREMENT|
+----------------+------------+----+-------+-----------+--------------+
|timeslot_id     |INT         |    |PRIMARY|           |              |
|channel_id      |INT         |    |FOREIGN|           |              |
|start_time      |DATETIME    |    |       |           |              |
|end_time        |DATETIME    |    |       |           |              |
+----------------+------------+----+-------+-----------+--------------+
Programslots
+----------------+------------+----+-------+-----------+--------------+
|Columns         |DATA TYPE   |NULL|KEY    |DEFAULT    |AUTO INCREMENT|
+----------------+------------+----+-------+-----------+--------------+
|programslot_id  |INT         |    |PRIMARY|           |              |
|timeslot_id     |INT         |    |FOREIGN|           |              |
|episode_id      |INT         |    |FOREIGN|           |              |
|program_id      |INT         |    |FOREIGN|           |              |
|view_count      |INT         |    |       |           |              |
+----------------+------------+----+-------+-----------+--------------+

Channels
+----------------+------------+----+-------+-----------+--------------+
|Columns         |DATA TYPE   |NULL|KEY    |DEFAULT    |AUTO INCREMENT|
+----------------+------------+----+-------+-----------+--------------+
|channel_id      |INT         |    |PRIMARY|           |YES           |
|channel_no      |INT         |    |       |           |              |
+----------------+------------+----+-------+-----------+--------------+
Seasons
+----------------+------------+----+-------+-----------+--------------+
|Columns         |DATA TYPE   |NULL|KEY    |DEFAULT    |AUTO INCREMENT|
+----------------+------------+----+-------+-----------+--------------+
|season_id       |INT         |    |PRIMARY|           |              |
|season_no       |INT         |    |       |           |              |
+----------------+------------+----+-------+-----------+--------------+

Episodes
+----------------+------------+----+-------+-----------+--------------+
|Columns         |DATA TYPE   |NULL|KEY    |DEFAULT    |AUTO INCREMENT|
+----------------+------------+----+-------+-----------+--------------+
|episode_id      |INT         |    |PRIMARY|           |YES           |
|season_id       |INT         |    |FOREIGN|           |              |
|episode_no      |INT         |OK  |       |           |              |
|title           |VARCHAR(80) |    |UNIQUE |           |              |
|description     |VARCHAR(300)|OK  |       |ComingSoon!|              |
|video_duration  |BIGINT      |    |       |           |              |
|release_date    |DATE        |    |       |           |              |
|view_count      |INT         |    |       |           |              |
+----------------+------------+----+-------+-----------+--------------+
```

## **4,ドラマというチャンネルがあったとして、ドラマのチャンネルの番組表を表示するために、本日から一週間分、何日の何時から何の番組が放送されるのかを知りたいです。**
ドラマのチャンネルに対して、放送開始時刻、放送終了時刻、シーズン数、エピソード数、エピソードタイトル、エピソード詳細を本日から一週間分取得してください

放送開始時刻(Timeslots.start_time)、放送終了時刻(Timeslots.end_time)
シーズン数(Season.season_no)
エピソード数(Episode.episode_no)、エピソードタイトル(Episode.title)、エピソード詳細(Episode.description)
からそれぞれ取得する。


```sql
SELECT
  t.start_time,
  t.end_time,
  s.season_no    AS season_no,   
  e.episode_no   AS episode_no,  
  e.title        AS episode_title,
  e.description  AS episode_description
FROM Timeslots t
JOIN Channels     c  ON c.channel_id    = t.channel_id
JOIN Programslots ps ON ps.timeslot_id  = t.timeslot_id
JOIN Episodes    e  ON e.episode_id    = ps.episode_id
LEFT JOIN Seasons s  ON s.season_id     = e.season_id
WHERE (c.name LIKE 'ドラマ%')
  AND t.start_time >= CURRENT_DATE
  AND t.start_time <  CURRENT_DATE + INTERVAL 7 DAY
ORDER BY t.start_time;
```
```text
+---------------------+---------------------+-----------+------------+---------------+---------------------+
| start_time          | end_time            | season_no | episode_no | episode_title | episode_description |
+---------------------+---------------------+-----------+------------+---------------+---------------------+
| 2025-08-10 19:00:00 | 2025-08-10 20:00:00 |         1 |          1 | SQ S1E1       | パイロット回        |
| 2025-08-11 19:00:00 | 2025-08-11 20:00:00 |         2 |          1 | SQ2 S2E1      | 新章開幕            |
| 2025-08-12 19:00:00 | 2025-08-12 20:00:00 |         1 |          3 | SQ S1E3       | 急展開              |
| 2025-08-13 19:00:00 | 2025-08-13 20:00:00 |         2 |          1 | SQ2 S2E1      | 新章開幕            |
| 2025-08-14 19:00:00 | 2025-08-14 20:00:00 |         1 |          1 | SQ S1E1       | パイロット回        |
| 2025-08-15 19:00:00 | 2025-08-15 20:00:00 |         1 |          2 | SQ S1E2       | 新キャラ登場        |
| 2025-08-16 19:00:00 | 2025-08-16 20:00:00 |         2 |          2 | SQ2 S2E2      | 謎が深まる          |
+---------------------+---------------------+-----------+------------+---------------+---------------------+
7 rows in set (0.001 sec)
```


**<解説>**
- **`SELECT文`**でデータを取得する。
- **(AS channel_name,)**
    それぞれ見やすいカラム名にネーミング。
- **(FROM Timeslots t)**
    データのスタート地点(起点)はTimeslots。今後はtと呼ぶ。
- **(JOIN)**
    バラバラのテーブルを結合していく。
- **(JOIN Channels c ON c.channel_id = t.channel_id)**
    c.channel_idとtのFKとして配置したchannel_id(t.channel_id)を参照&結合。 [Timeslots-Channels]
- **(JOIN Programslots  ps ON ps.timeslot_id  = t.timeslot_id )**
    ps.timeslot_idとtのPKとして配置したtimeslots_id(t.timeslot_id)を参照&結合。[Timeslots-Channels-Programslots]
- **(JOIN Episodes e ON e.episode_id = ps.episode_id)**
    e.episode_idとpsのFKとして配置したepisode_id(ps.episode_id)を参照&結合。[Timeslots-Channels-Programslots-Episodes]
- **(LEFT JOIN)**
    JOINで結合済みの左辺`[Timeslots-Channels-Programslots-Episodes]`へ結合する。
- **(LEFT JOIN Seasons s  ON s.season_id  = e.season_id)**
    Seasonsを結合するために、s.season_idとEpisodesにFKとして配置したseason_id(e.season_id)を結合。
    これまで結合してきた`[Timeslots-Channels-Programslots-Episodes]`へ`Seasons`を新たに加えるイメージ。
    Seasonsにないデータは`NULL`が返される。
- **WHERE (c.name LIKE 'ドラマ%')**
    「ドラマ」と名がつくチャンネルの番組だけに絞り込み取得する。  
- **(AND t.start_time >= CURRENT_DATE)**
    本日の0:00以降に開始する番組だけを対象に取得する。
    `CURRENT_DATE`は「セッションのタイムゾーンの“今日の0:00”」を意味する。取得範囲の下限（含む）
- **(AND t.start_time <  CURRENT_DATE + INTERVAL 7 DAY)**
    本日から7日後の0:00未満まで、つまり1週間分を取得する。
- **(ORDER BY t.start_time;)**
    開始時刻が早い順に並べます。


