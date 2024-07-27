# SQL_CASE式のススメ

## はじめに

[こちら](https://www.amazon.co.jp/gp/product/B07GB4CNKP/ref=ppx_yo_dt_b_d_asin_title_351_o03?ie=UTF8&psc=1)の書籍のアウトプットになります。本の総括だと薄く広くなりそうなので今回はその中のCASE文についてとりあげていきます


## CASE式とは

- SQLにおいて条件分岐が書ける式のこと
- 単純CASE式と検索CASE式の二つがある

以下は単純CASE式と検索CASE式の例、どちらも以下のような処理をする
- sexが1なら男を返す
- sexが2なら女を返す
- それ以外ならその他を返す

### 単純CASE式
switch文のような形で分岐を書ける

```sql
CASE sex
WHEN '1' THEN '男'
WHEN '2' THEN '女'
ELSE 'その他' END
```
### 検索CASE式
```SQL
CASE WHEN sex = '1' THEN '男'
     WHEN sex = '2' THEN '女'
ELSE 'その他' END
```

### CASE式の注意事項

主に以下の注意事項があります

- 各分岐が返すデータ型を統一する
- ENDの書き忘れに注意
- ELSE句は必ず書こう

CASE式の返すデータ型においてある分岐では文字型を返し、別の分岐では数値型を返す、という書き方はできません。またENDを書き忘れることが多いそうです。
ELSEを書かない場合、デフォルトでは`ELSE NULL` となります。NULLはSQLにおいては厄災の火種となるのでオプションとはいえ書いた方がいいでしょう。


## CASE式でできること

詳しくは後述していきます。

- 既存のコード体系を新しい体系に変換して集計する
- 異なる条件の集計を１つのSQLで行なう
- CHECK制約で複数の列の条件関係を定義する
- 条件を分岐させたUPDATE
- テーブル同士のマッチング
- CASE式の中で集約関数を使う

## 既存のコード体系を新しい体系に変換して集計する

集計元の表 PopTbl->集計結果のように県ごとのデータを集計する場合を考えます

### 集計元の表 PopTbl

| pref_name (県名) | population (人口) |
|------------------|-------------------|
| 徳島             | 100               |
| 香川             | 200               |
| 愛媛             | 150               |
| 高知             | 200               |
| 福岡             | 300               |
| 佐賀             | 100               |
| 長崎             | 200               |
| 東京             | 400               |
| 群馬             | 50                |

### 集計結果

| 地方名 | 人口 |
|--------|------|
| 四国   | 650  |
| 九州   | 600  |
| その他 | 450  |

この場合、以下のようにCASE式で記述できます。

```sql
SELECT CASE pref_name
              WHEN '徳島' THEN '四国'
              WHEN '香川' THEN '四国'
              WHEN '愛媛' THEN '四国'
              WHEN '高知' THEN '四国'
              WHEN '福岡' THEN '九州'
			  WHEN '佐賀' THEN '九州'
              WHEN '長崎' THEN '九州'
       ELSE 'その他' END AS district,
       SUM(population)
  FROM PopTbl
 GROUP BY CASE pref_name
              WHEN '徳島' THEN '四国'
              WHEN '香川' THEN '四国'
              WHEN '愛媛' THEN '四国'
              WHEN '高知' THEN '四国'
              WHEN '福岡' THEN '九州'
              WHEN '佐賀' THEN '九州'
              WHEN '長崎' THEN '九州'
          ELSE 'その他' END;
```

ここでは以下の処理をしています
- SELECT文のCASE式では県ごとに地方に分類
- その後地方ごとにカウントするためにGROUPBYでグループ化

また、このSQLではCASE式で同じ記述をしているのが気になる人がいると思います。その場合、ASと併用することで記述を省略可能です。

```SQL
SELECT CASE pref_name
              WHEN '徳島' THEN '四国'
              WHEN '香川' THEN '四国'
              WHEN '愛媛' THEN '四国'
              WHEN '高知' THEN '四国'
              WHEN '福岡' THEN '九州'
              WHEN '佐賀' THEN '九州'
			  WHEN '長崎' THEN '九州'
       ELSE 'その他' END AS district,
       SUM(population)
  FROM PopTbl
 GROUP BY district;
```
## 異なる条件の集計を１つのSQLで行なう

集計元の表 PopTbl2->集計結果のように県ごとの男女のデータを集計する場合を考えます


### 集計元の表 PopTbl2

| pref_name (県名) | sex (性別) | population (人口) |
|------------------|------------|-------------------|
| 徳島             | 1          | 60                |
| 徳島             | 2          | 40                |
| 香川             | 1          | 100               |
| 香川             | 2          | 100               |
| 愛媛             | 1          | 100               |
| 愛媛             | 2          | 50                |
| 高知             | 1          | 100               |
| 高知             | 2          | 100               |
| 福岡             | 1          | 100               |
| 福岡             | 2          | 200               |
| 佐賀             | 1          | 20                |
| 佐賀             | 2          | 80                |
| 長崎             | 1          | 125               |
| 長崎             | 2          | 125               |
| 東京             | 1          | 250               |
| 東京             | 2          | 150               |

### 集計結果

| 県名 | 男   | 女   |
|------|------|------|
| 徳島 | 60   | 40   |
| 香川 | 100  | 100  |
| 愛媛 | 100  | 50   |
| 高知 | 100  | 100  |
| 福岡 | 100  | 200  |
| 佐賀 | 20   | 80   |
| 長崎 | 125  | 125  |
| 東京 | 250  | 150  |


### whereを用いた場合

通常、whereで性別を絞った場合、以下のようになります
この場合、２回SQLを叩く必要があり非効率です
また、この後join等を用いてテーブルを結合する必要もあります

```sql
-- 男性の人口
SELECT pref_name,
       population
  FROM PopTbl2
 WHERE sex = '1';

-- 女性の人口
SELECT pref_name,
       population
  FROM PopTbl2
 WHERE sex = '2';
```
### caseを用いた場合

一方、CASE式を用いた場合は一回のクエリで処理が済みます
sum関数の中で性別が合致している場合はその数を、それ以外は0を返すように条件を分岐させることで実現できます。またそのままクロス表にできるのも利点の一つです

```sql
SELECT pref_name,
       -- 男性の人口
       SUM( CASE WHEN sex = '1' THEN population ELSE 0 END) AS cnt_m,
       -- 女性の人口
       SUM( CASE WHEN sex = '2' THEN population ELSE 0 END) AS cnt_f
  FROM PopTbl2
 GROUP BY pref_name;
```
## CHECK制約で複数の列の条件関係を定義する

[check制約](https://techis.jp/guide/sql/sql_check)を掛けるときもCASE式は活躍します。
条件分岐することで特定の条件を満たす場合に制約を掛けることができます。
例えば以下のSQLでは性別が2のときに給料が20万以下という時代に逆行した制約をかけています。(本の例をそのまま記載しています)

```sql
CONSTRAINT check_salary CHECK
(
    CASE WHEN sex = '2'
    THEN CASE WHEN salary <= 200000
    THEN 1 ELSE 0 END
    ELSE 1 END = 1
)
```
## 条件を分岐させたUPDATE

以下のテーブルで以下のような更新をかけます
- 現在の給料が30万以上の社員は、10％の減給とする
- 現在の給料が25万以上28万未満の社員は、20％の昇給とする

### 更新前の給与テーブル

| name | salary  |
|------|---------|
| 相田  | 300,000 |
| 神崎  | 270,000 |
| 木村  | 220,000 |
| 斎藤  | 290,000 |
### 更新後の給与テーブル
| name | salary  |
|------|---------|
| 相田  | 270,000 |
| 神崎  | 324,000 |
| 木村  | 220,000 |
| 斎藤  | 290,000 |


このような場合でもCASE式を用いれば簡単にかけます
```sql
UPDATE Personnel
SET salary = CASE WHEN salary >= 300000
                  THEN salary * 0.9
                  WHEN salary >= 250000 AND salary < 280000
                  THEN salary * 1.2
                  ELSE salary END;
```

## テーブル同士のマッチング

CourseMasterテーブルとOpenCourseテーブルをマッチングさせてCourseMonthテーブルを作成することを考えます。
### CourseMaster

| course_id | course_name             |
|-----------|-------------------------|
| 1         | 経理入門                |
| 2         | 財務知識                |
| 3         | 簿記検定開講講座        |
| 4         | 税理士                  |

### OpenCourses

| month | course_id |
|-------|-----------|
| 201806| 1         |
| 201806| 3         |
| 201806| 4         |
| 201807| 4         |
| 201808| 2         |
| 201808| 4         |

### CourseMonth

| course_name | 6月 | 7月 | 8月 |
|-------------|-----|-----|-----|
| 経理入門    | ○   | ✕   | ✕   |
| 財務知識    | ✕   | ✕   | ○   |
| 簿記検定    | ○   | ✕   | ○   |
| 税理士      | ○   | ○   | ○   |

このような場合でもCASE式は活躍します。
CASE式ではサブクエリやBETWEEN、LIKE、<, >といった便利な述語群を使えるのでこのような記述も可能です。
```sql
-- テーブルのマッチング：IN 演語の利用
SELECT course_name,
       CASE WHEN course_id IN 
           (SELECT course_id FROM OpenCourses 
            WHERE month = 201806) THEN '〇'
            ELSE 'x' END AS "6月",
       CASE WHEN course_id IN 
           (SELECT course_id FROM OpenCourses 
            WHERE month = 201807) THEN '〇'
            ELSE 'x' END AS "7月",
       CASE WHEN course_id IN 
           (SELECT course_id FROM OpenCourses 
            WHERE month = 201808) THEN '〇'
            ELSE 'x' END AS "8月"
FROM CourseMaster;

-- テーブルのマッチング：EXISTS 演語の利用
SELECT CM.course_name,
       CASE WHEN EXISTS
           (SELECT course_id FROM OpenCourses OC 
            WHERE month = 201806
            AND OC.course_id = CM.course_id) THEN '〇'
            ELSE 'x' END AS "6月",
       CASE WHEN EXISTS
           (SELECT course_id FROM OpenCourses OC 
            WHERE month = 201807
            AND OC.course_id = CM.course_id) THEN '〇'
            ELSE 'x' END AS "7月",
       CASE WHEN EXISTS
           (SELECT course_id FROM OpenCourses OC 
            WHERE month = 201808
            AND OC.course_id = CM.course_id) THEN '〇'
            ELSE 'x' END AS "8月"
FROM CourseMaster CM;

```
## CASE式の中で集約関数を使う

以下のテーブルでこのような処理をすることを考えます

1. 1つだけのクラブに所属している学生については、そのクラブIDを取得する
2. 複数のクラブをかけ持ちしている学生については、主なクラブのIDを取得する

### StudentClub

| std_id (学生番号) | club_id (クラブID) | club_name (クラブ名) | main_club_flg (主なクラブフラグ) |
|-------------------|--------------------|----------------------|----------------------------------|
| 100               | 1                  | 野球                 | Y                                |
| 100               | 2                  | 吹奏楽               | N                                |
| 200               | 2                  | 吹奏楽               | N                                |
| 200               | 3                  | バドミントン         | Y                                |
| 200               | 4                  | サッカー             | N                                |
| 300               | 4                  | サッカー             | N                                |
| 400               | 5                  | 水泳                 | Y                                |
| 500               | 6                  | 囲碁                 | N                                |

### 結果
| std_id | main_club |
|--------|-----------|
| 100    | 1         |
| 200    | 3         |
| 300    | 4         |
| 400    | 5         |
| 500    | 6         |

### Having句を用いた場合

Having句を用いると以下のような記述になります。
例によって例のごとく、2回のクエリが必要になります。

```sql
---条件1のクエリ
SELECT std_id, MAX(club_id) AS main_club
FROM StudentClub
GROUP BY std_id
HAVING COUNT(*) = 1;

---条件2のクエリ
SELECT std_id, club_id AS main_club
FROM StudentClub
WHERE main_club_flg = 'Y';
```

### CASE式を用いた場合

一方、CASE式を用いた場合は以下のように記述できます

```sql
SELECT std_id,
       CASE WHEN COUNT(*) = 1 -- 1つのクラブに専念する学生の場合
            THEN MAX(club_id)
            ELSE MAX(CASE WHEN main_club_flg = 'Y'
                          THEN club_id
                          ELSE NULL END) END AS main_club
FROM StudentClub
GROUP BY std_id;
```
このようにCASE式ならシンプルに記述できるわけです。

## まとめと感想

書籍では以下のようにまとめられています。

1. GROUP BY 句で CASE 式を使うことで、集約単位となるコードや階級を柔軟に設定できる。これは非定形的な集計に大きな威力を発揮する。
2. 集約関数の中に使うことで、行持ちから列持ちへの水平展開も自由自在。
3. 反対に集約関数を条件式に組み込むことで HAVING 句を使わずにクエリをまとめられる。
4. 実装依存の関数より表現力が非常に強力なうえ、汎用性も高まって一石二鳥。
5. そうした利点があるのも、CASE 式が「文」ではなく「式」であるからこそ。
6. CASE 式を駆使することで複数の SQL 文を１つにまとめられ、可読性もパフォーマンスも向上して良いことづくし。

私の感想としては`なにかしら条件分岐をするならCASE式が使えないか検討する`ことでうまくCASE式を活用できそうだと思いました。まだ実務に就いてないので実践でどう活きるのかイメージ付き辛いですがいつか役に立つことを信じてesaに残しておきます。

ここまでご覧いただいてありがとうございました。