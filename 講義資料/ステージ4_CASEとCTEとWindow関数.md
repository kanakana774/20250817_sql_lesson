# **SQL 基礎：高度な SELECT**

## **導入：より複雑なデータ操作のために**

これまでの章では、データの追加・更新・削除の基本、単一テーブル内での集計やフィルタリング、そして複数テーブルの結合方法を学びました。本章では、SQL のより高度な機能に焦点を当てます。

- **CASE 式**: データに基づいて条件付きの処理を行い、値を変換したり、クロス集計を行ったりします。
- **CTE（共通テーブル式）**: 複雑なクエリを分割し、可読性とメンテナンス性を向上させるための強力なツールです。再帰的なデータの展開にも利用できます。
- **ウィンドウ関数**: 集計関数と似ていますが、行をグループ化して集約するのではなく、行のグループ（ウィンドウ）内で計算を行い、その結果を各行にそのまま返します。これにより、ランキングや移動平均などの複雑な分析が可能になります。

これらの機能は、実務でより洗練されたデータ分析やレポート作成を行う上で不可欠な知識となります。

## **条件分岐：CASE 式 (実務での重要度：非常に高 ◎)**

CASE 式は、SQL クエリ内で条件分岐ロジックを実装するために使用されます。プログラミング言語における if-else if-else 文や switch-case 文に似ています。SELECT 句で新しい列を作成したり、WHERE 句で条件を指定したり、ORDER BY 句で並び替え順を制御したりと、様々な場所で利用できます。

### **基本構文**

CASE 式には、主に 2 つの形式があります。

#### **1\. シンプル CASE 式 (Simple CASE Expression)**

特定の列の値に基づいて条件分岐する場合に使用します。

```SQL
CASE 列名
 WHEN 値 1 THEN 結果 1
 WHEN 値 2 THEN 結果 2
 ...
 \[ELSE デフォルト結果\]
END
```

#### **2\. 検索 CASE 式 (Searched CASE Expression)**

複数の条件式（比較演算子や論理演算子など）に基づいて条件分岐する場合に使用します。より柔軟な条件指定が可能です。

```SQL
CASE
 WHEN 条件式 1 THEN 結果 1
 WHEN 条件式 2 THEN 結果 2
 ...
 \[ELSE デフォルト結果\]
END
```

- ELSE 句は省略可能ですが、指定しなかった場合、どの WHEN 条件にも一致しない行の結果は NULL になります。
- 最初の TRUE と評価された WHEN 句に対応する結果が返され、それ以降の WHEN 句は評価されません。**WHEN 句の記述順序は非常に重要です。**

### **例 1: 条件分岐で値を変換（評価の付与）**

学生の点数に基づいて、'優', '良', '可', '不可'の評価を付与します。

**事前データ:**

students テーブル:

| student_id | name    | score |
| :--------- | :------ | :---- |
| 1          | Alice   | 95    |
| 2          | Bob     | 78    |
| 3          | Charlie | 62    |
| 4          | David   | 40    |
| 5          | Eve     | NULL  |

**SQL:**

```SQL
SELECT
 name,
 score,
 CASE
 WHEN score \>= 90 THEN '優'
 WHEN score \>= 80 THEN '良'
 WHEN score \>= 60 THEN '可'
 ELSE '不可' \-- どの条件にも合致しない場合（NULL も含む）
 END AS grade
FROM
 students;
```

**実行結果:**

| name    | score | grade |
| :------ | :---- | :---- |
| Alice   | 95    | 優    |
| Bob     | 78    | 良    |
| Charlie | 62    | 可    |
| David   | 40    | 不可  |
| Eve     | NULL  | 不可  |

説明:  
CASE 式は、WHEN 句の条件式を上から順に評価します。最初に TRUE と評価された WHEN 句に対応する結果が返され、それ以降の WHEN 句は評価されません。  
score が NULL の Eve の行では、score \>= 90 も score \>= 80 も score \>= 60 もすべて FALSE と評価されるため（NULL との比較結果は NULL となり、WHERE 句などでは FALSE 扱いされる）、最終的に ELSE 句の'不可'が適用されます。CASE 式において、NULL との比較結果は常に UNKNOWN となり、UNKNOWN は WHEN 句では FALSE と同じように扱われ、次の WHEN 句または ELSE 句に処理が移ります。

### **例 2: クロス集計（勤怠区分ごとの人数を列化）**

CASE 式を集計関数と組み合わせることで、テーブルの行データを列データに変換する\*\*クロス集計（ピボット）\*\*のような操作が可能です。これはレポート作成で非常に役立ちます。

**事前データ:**

attendance テーブル:

| employee_id | attendance_date | status |
| :---------- | :-------------- | :----- |
| 101         | '2023-10-26'    | 出勤   |
| 102         | '2023-10-26'    | 欠勤   |
| 103         | '2023-10-26'    | 出勤   |
| 101         | '2023-10-27'    | 早退   |
| 102         | '2023-10-27'    | 出勤   |
| 103         | '2023-10-27'    | 欠勤   |

```SQL
with attendance as (
	select 101 as employee_id ,'2023-10-26' as attendance_date,'出勤' as status union all
	select 102 as employee_id ,'2023-10-26' as attendance_date,'欠勤' as status union all
	select 103 as employee_id ,'2023-10-26' as attendance_date,'出勤' as status union all
	select 101 as employee_id ,'2023-10-27' as attendance_date,'早退' as status union all
	select 102 as employee_id ,'2023-10-27' as attendance_date,'出勤' as status union all
	select 103 as employee_id ,'2023-10-27' as attendance_date,'欠勤' as status
)
```

**SQL (CASE 式と SUM/COUNT を使用):**

```SQL
SELECT
 attendance_date,
 COUNT(CASE WHEN status = '出勤' THEN 1 END) AS 出勤者数_COUNT,
 COUNT(CASE WHEN status = '欠勤' THEN 1 END) AS 欠勤者数_COUNT,
 COUNT(CASE WHEN status = '早退' THEN 1 END) AS 早退者数_COUNT,
 SUM(CASE WHEN status = '出勤' THEN 1 ELSE 0 END) AS 出勤者数_SUM,
 SUM(CASE WHEN status = '欠勤' THEN 1 ELSE 0 END) AS 欠勤者数_SUM,
 SUM(CASE WHEN status = '早退' THEN 1 ELSE 0 END) AS 早退者数_SUM
FROM
 attendance
GROUP BY
 attendance_date
ORDER BY
 attendance_date;
```

**実行結果:**

| attendance_date | 出勤者数\_COUNT | 出勤者数\_SUM | 欠勤者数\_COUNT | 欠勤者数\_SUM | 早退者数\_COUNT | 早退者数\_SUM |
| :-------------- | :-------------- | :------------ | :-------------- | :------------ | :-------------- | :------------ |
| 2023-10-26      | 2               | 2             | 1               | 1             | 0               | 0             |
| 2023-10-27      | 1               | 1             | 1               | 1             | 1               | 1             |

**説明:**

- COUNT(CASE WHEN status \= '出勤' THEN 1 END)の挙動に注目してください。status が'出勤'の場合に 1 が返され、それ以外の場合は ELSE 句がないため NULL が返されます。COUNT(列名)は NULL を無視するため、結果的に'出勤'の行数のみがカウントされます。
- SUM(CASE WHEN status \= '出勤' THEN 1 ELSE 0 END)の場合、status が'出勤'でなければ 0 が返されます。SUM()関数は 0 も合計に含めるため、結果は COUNT()と同じになります。  
  どちらの書き方でも結果は同じになりますが、SUM(CASE WHEN ... THEN 1 ELSE 0 END)は、NULL を考慮せず全ての行で 0 または 1 を返すため、より明示的でわかりやすいと考える人も多いです。また、特定の DB システムによっては SUM の方が最適化されやすい場合もあります。

説明用

```SQL
with attendance as (
	select 101 as employee_id ,'2023-10-26' as attendance_date,'出勤' as status union all
	select 102 as employee_id ,'2023-10-26' as attendance_date,'欠勤' as status union all
	select 103 as employee_id ,'2023-10-26' as attendance_date,'出勤' as status union all
	select 101 as employee_id ,'2023-10-27' as attendance_date,'早退' as status union all
	select 102 as employee_id ,'2023-10-27' as attendance_date,'出勤' as status union all
	select 103 as employee_id ,'2023-10-27' as attendance_date,'欠勤' as status
)
-- ① まずはこんなデータ
-- select* from attendance

-- ② 日付とステータスで集計（日ごとの勤怠状況を集計）してみるけど、見づらい
-- SELECT
--   attendance_date,
--   status,
--   COUNT(*) AS cnt
-- FROM attendance
-- GROUP BY attendance_date, status
-- ORDER BY attendance_date, status

-- ③ クロス集計にしてみる（勤怠状況列を横に展開する）
-- SELECT
--  attendance_date,
--  COUNT(CASE WHEN status = '出勤' THEN 1 END) AS 出勤者数_COUNT,
--  COUNT(CASE WHEN status = '欠勤' THEN 1 END) AS 欠勤者数_COUNT,
--  COUNT(CASE WHEN status = '早退' THEN 1 END) AS 早退者数_COUNT,
--  SUM(CASE WHEN status = '出勤' THEN 1 ELSE 0 END) AS 出勤者数_SUM,
--  SUM(CASE WHEN status = '欠勤' THEN 1 ELSE 0 END) AS 欠勤者数_SUM,
--  SUM(CASE WHEN status = '早退' THEN 1 ELSE 0 END) AS 早退者数_SUM
-- FROM
--  attendance
-- GROUP BY
--  attendance_date
-- ORDER BY
--  attendance_date;
```

<details><summary>半分統計分析のだけど、クロス集計の例</summary>

```SQL

-- Passengers CTE: ランダムな乗客IDと基本属性を生成
WITH Passengers AS (
    SELECT
        ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS PassengerID,
        -- Sexを先に決定
        CASE
            WHEN (FLOOR(RANDOM() * 100) + 1)::INT % 100 < 65 THEN 'male' -- 約65%が男性
            ELSE 'female'
        END AS Sex,
        CASE
            WHEN (FLOOR(RANDOM() * 100) + 1)::INT % 100 < 25 THEN 1 -- 約25%が1等客室
            WHEN (FLOOR(RANDOM() * 100) + 1)::INT % 100 < 50 THEN 2 -- 約25%が2等客室
            ELSE 3                       -- 残りが3等客室
        END AS Pclass,
        (FLOOR(RANDOM() * 80) + 1)::INT AS Age -- 1歳から80歳まで
    FROM
        (SELECT 1 FROM GENERATE_SERIES(1, 1000)) AS a -- 1000件のダミーレコードを生成
),
-- AgeGroups CTE: Passengersの年齢に基づいて年齢層を分類 (Survived計算のために先に移動)
AgeGroups AS (
    SELECT
        p.PassengerID,
        p.Sex,
        p.Pclass,
        p.Age,
        CASE
            WHEN p.Age < 12 THEN 'Child'
            WHEN p.Age BETWEEN 12 AND 17 THEN 'Teenager'
            WHEN p.Age BETWEEN 18 AND 59 THEN 'Adult'
            ELSE 'Senior'
        END AS AgeGroup
    FROM
        Passengers p
),
-- SurvivedAdjusted CTE: Sex, Pclass, AgeGroupに基づいてSurvivedを調整
SurvivedAdjusted AS (
    SELECT
        ag.PassengerID,
        ag.Sex,
        ag.Pclass,
        ag.Age,
        ag.AgeGroup,
        CASE
            WHEN ag.Sex = 'female' THEN -- 女性の生存率
                CASE
                    WHEN ag.Pclass = 1 AND ag.AgeGroup = 'Child' AND (FLOOR(RANDOM() * 100) + 1)::INT % 100 < 95 THEN 1 -- 女性、1等客室、子供: 約95%生存
                    WHEN ag.Pclass = 1 AND (FLOOR(RANDOM() * 100) + 1)::INT % 100 < 90 THEN 1 -- 女性、1等客室、非子供: 約90%生存
                    WHEN ag.Pclass = 2 AND ag.AgeGroup = 'Child' AND (FLOOR(RANDOM() * 100) + 1)::INT % 100 < 85 THEN 1 -- 女性、2等客室、子供: 約85%生存
                    WHEN ag.Pclass = 2 AND (FLOOR(RANDOM() * 100) + 1)::INT % 100 < 75 THEN 1 -- 女性、2等客室、非子供: 約75%生存
                    WHEN ag.Pclass = 3 AND ag.AgeGroup = 'Child' AND (FLOOR(RANDOM() * 100) + 1)::INT % 100 < 60 THEN 1 -- 女性、3等客室、子供: 約60%生存
                    WHEN ag.Pclass = 3 AND (FLOOR(RANDOM() * 100) + 1)::INT % 100 < 45 THEN 1 -- 女性、3等客室、非子供: 約45%生存
                    ELSE 0 -- 上記条件に合致しない女性は死亡
                END
            WHEN ag.Sex = 'male' THEN -- 男性の生存率
                CASE
                    WHEN ag.Pclass = 1 AND ag.AgeGroup = 'Child' AND (FLOOR(RANDOM() * 100) + 1)::INT % 100 < 50 THEN 1 -- 男性、1等客室、子供: 約50%生存
                    WHEN ag.Pclass = 1 AND (FLOOR(RANDOM() * 100) + 1)::INT % 100 < 35 THEN 1 -- 男性、1等客室、非子供: 約35%生存
                    WHEN ag.Pclass = 2 AND ag.AgeGroup = 'Child' AND (FLOOR(RANDOM() * 100) + 1)::INT % 100 < 30 THEN 1 -- 男性、2等客室、子供: 約30%生存
                    WHEN ag.Pclass = 2 AND (FLOOR(RANDOM() * 100) + 1)::INT % 100 < 15 THEN 1 -- 男性、2等客室、非子供: 約15%生存
                    WHEN ag.Pclass = 3 AND ag.AgeGroup = 'Child' AND (FLOOR(RANDOM() * 100) + 1)::INT % 100 < 20 THEN 1 -- 男性、3等客室、子供: 約20%生存
                    WHEN ag.Pclass = 3 AND (FLOOR(RANDOM() * 100) + 1)::INT % 100 < 10 THEN 1 -- 男性、3等客室、非子供: 約10%生存
                    ELSE 0 -- 上記条件に合致しない男性は死亡
                END
            ELSE 0 -- その他（理論上は発生しないはず）
        END AS Survived
    FROM
        AgeGroups ag
),
-- EmbarkedPorts CTE: PassengerIDに基づいてランダムに乗船港を割り当て
EmbarkedPorts AS (
    SELECT
        sa.PassengerID,
        sa.Survived,
        sa.Pclass,
        sa.Sex,
        sa.Age,
        sa.AgeGroup,
        CASE (FLOOR(RANDOM() * 3) + 1)::INT
            WHEN 1 THEN 'S' -- Southampton
            WHEN 2 THEN 'C' -- Cherbourg
            ELSE 'Q'        -- Queenstown
        END AS Embarked
    FROM
        SurvivedAdjusted sa
)

-- 最終的な結果の選択: クロス集計に最適な形式
-- SELECT
-- PassengerID,
-- Survived,
-- Pclass,
-- Sex,
-- AgeGroup,
-- Embarked
-- FROM
-- EmbarkedPorts
-- ORDER BY
-- PassengerID;

-- 動機：生存者の特徴を調べたい（通説の検証）
-- ①pk を削除して、group by キーで集計
-- select
-- Survived,
-- Pclass,
-- Sex,
-- AgeGroup,
-- Embarked,
-- count(1) AS 合計
-- from EmbarkedPorts
-- group by Survived,Pclass,Sex,AgeGroup,Embarked

-- ② 女性の方が生存率が高かったてきな通説を見てみる
-- ,crosstab as(
-- select
-- Sex,
-- count(case when Survived=1 then 1 end) as 生存,
-- count(case when Survived=0 then 1 end) as 没
-- from EmbarkedPorts
-- group by Sex
-- )
-- select
-- Sex,
-- concat(ROUND(生存::numeric / (生存 + 没)*100,1),'%') as 生存割合,
-- concat(ROUND(没::numeric / (生存 + 没)*100,1),'%') as 没割合
-- from crosstab
-- order by 生存割合 desc

-- ③ 1 等客室の生存率が高かった説を見てみる
-- ,crosstab as(
-- select
-- Pclass,
-- count(case when Survived=1 then 1 end) as 生存,
-- count(case when Survived=0 then 1 end) as 没
-- from EmbarkedPorts
-- group by Pclass
-- )
-- select
-- Pclass,
-- concat(ROUND(生存::numeric / (生存 + 没)*100,1),'%') as 生存割合,
-- concat(ROUND(没::numeric / (生存 + 没)*100,1),'%') as 没割合
-- from crosstab
-- order by 生存割合 desc

-- ④ 客室と性別で
,crosstab as(
select
Sex,
Pclass,
count(case when Survived=1 then 1 end) as 生存,
count(case when Survived=0 then 1 end) as 没
from EmbarkedPorts
group by Sex,Pclass
)
select
Sex,
Pclass,
concat(ROUND(生存::numeric / (生存 + 没)*100,1),'%') as 生存割合,
concat(ROUND(没::numeric / (生存 + 没)*100,1),'%') as 没割合
from crosstab
order by 生存割合 desc

```

</details>

#### **💡 コラム: PostgreSQL の FILTER 句を使ったクロス集計 (PostgreSQL 特有)**

PostgreSQL では、WHERE 句とは別に集計関数に対して直接フィルタリング条件を指定できる FILTER 句が提供されています。これはクロス集計において非常に簡潔で効率的な書き方として推奨されます。

**SQL (PostgreSQL の FILTER 句を使用):**

```SQL
SELECT
  attendance_date,
  COUNT(*) FILTER (WHERE status = '出勤') AS 出勤者数,
  COUNT(*) FILTER (WHERE status = '欠勤') AS 欠勤者数,
  COUNT(*) FILTER (WHERE status = '早退') AS 早退者数
FROM
  attendance
GROUP BY
  attendance_date
ORDER BY
  attendance_date;
```

**実行結果:**

| attendance_date | 出勤者数 | 欠勤者数 | 早退者数 |
| :-------------- | :------- | :------- | :------- |
| 2023-10-26      | 2        | 1        | 0        |
| 2023-10-27      | 1        | 1        | 1        |

説明:
FILTER (WHERE ...)句を使うことで、CASE 式をネストするよりも直感的に、特定の条件に合致する行のみを集計の対象とすることができます。PostgreSQL を使用する環境では、この FILTER 句を積極的に活用することが推奨されます。

### **例 3: ORDER BY 句での CASE 式の利用（カスタムソートと NULL の優先度制御）**

特定のカテゴリの商品を優先的に表示し、その後は通常の価格順に並び替えるなど、カスタムな並び替え順を定義できます。また、NULL 値を常に最後や最初に持ってくるような制御も可能です。

**事前データ:**

products テーブル:

| product_id | product_name | price   | category    |
| :--------- | :----------- | :------ | :---------- |
| 1          | Laptop       | 1200.00 | Electronics |
| 2          | Mouse        | 25.50   | Electronics |
| 3          | Keyboard     | 75.00   | Electronics |
| 4          | Monitor      | 300.00  | NULL        |
| 5          | Webcam       | 50.00   | Peripherals |
| 6          | Headset      | 150.00  | Peripherals |
| 7          | Bookshelf    | 80.00   | Home        |
| 8          | Projector    | 700.00  | NULL        |

**SQL:**

```SQL
SELECT
  product_name,
  category,
  price
FROM
  products
ORDER BY
  CASE category
    WHEN 'Electronics' THEN 1
    WHEN 'Peripherals' THEN 2
    WHEN 'Home' THEN 3
    ELSE 4 -- その他のカテゴリまたは NULL
  END,
  CASE
    WHEN category IS NULL THEN 1
    ELSE 0
  END, -- NULL を最後に持ってくるための追加条件（0:NULL ではない, 1:NULL）
  price DESC; -- 同じグループ内では価格が高い順
```

**実行結果:**

| product_name | category    | price   |
| :----------- | :---------- | :------ |
| Laptop       | Electronics | 1200.00 |
| Keyboard     | Electronics | 75.00   |
| Mouse        | Electronics | 25.50   |
| Headset      | Peripherals | 150.00  |
| Webcam       | Peripherals | 50.00   |
| Bookshelf    | Home        | 80.00   |
| Projector    | NULL        | 700.00  |
| Monitor      | NULL        | 300.00  |

説明:
ORDER BY 句内の最初の CASE 式が各行に 1 から 4 の数値を割り当て、その数値の昇順で並べ替えます。
二つ目の CASE 式 CASE WHEN category IS NULL THEN 1 ELSE 0 END は、category が NULL の行には 1 を、そうでない行には 0 を返します。これにより、最初の CASE 式で ELSE 4 とされたグループ（NULL カテゴリを含む）内で、NULL の行が非 NULL の行よりも後に並ぶようになります。
最後に price DESC で、同じグループ内の行は価格が高い順に並び替えられます。
💡 実務では、NULL 値をソート順の最初や最後に持ってきたいというニーズが頻繁に発生します。このような場合に CASE WHEN 列 IS NULL THEN ... ELSE ... END が非常に役立ちます。
⇒ ソート条件をカンマ区切りで順番に書けるので、case 式で応用的な書き方ができますよと。

### **例 4: WHERE 句での CASE 式の利用（動的フィルタリング）**

条件によってフィルタリングのロジックを変えたい場合に CASE 式を活用できます。

**シナリオ:** 在庫が多い商品（100 個以上）は価格が 500 ドルより高いものだけを表示し、それ以外の在庫数の商品は価格に関わらず表示する。

**事前データ:**

products テーブル:

| product_id | product_name | price   | stock_quantity | category    |
| :--------- | :----------- | :------ | :------------- | :---------- |
| 1          | Laptop       | 1200.00 | 150            | Electronics |
| 2          | Mouse        | 25.50   | 20             | Electronics |
| 3          | Keyboard     | 75.00   | 30             | Electronics |
| 4          | Monitor      | 300.00  | 120            | NULL        |
| 5          | Webcam       | 50.00   | 10             | Peripherals |
| 6          | Headset      | 150.00  | 200            | Peripherals |
| 7          | Bookshelf    | 80.00   | 5              | Home        |

**SQL (CASE 式を使用):**

```SQL
SELECT
  product_name,
  price,
  stock_quantity
FROM
pr  oducts
WHERE
  CASE
    WHEN stock_quantity >= 100 THEN price > 500 -- 在庫が 100 以上の場合は価格が 500 より大きい
    ELSE TRUE -- それ以外の場合は常に TRUE（フィルタリングしない）
  END;
```

**実行結果:**

| product_name | price   | stock_quantity |
| :----------- | :------ | :------------- |
| Laptop       | 1200.00 | 150            |
| Mouse        | 25.50   | 20             |
| Keyboard     | 75.00   | 30             |
| Webcam       | 50.00   | 10             |
| Bookshelf    | 80.00   | 5              |

**説明:**

- Laptop (stock_quantity=150, price=1200) は stock_quantity \>= 100 に合致し、かつ price \> 500 も満たすため抽出されます。
- Monitor (stock_quantity=120, price=300) は stock_quantity \>= 100 に合致しますが、price \> 500 を満たさないため抽出されません。
- Headset (stock_quantity=200, price=150) も同様に抽出されません。
- 残りの商品は stock_quantity \>= 100 に合致しないため、ELSE TRUE の条件が適用され、price に関わらず全て抽出されます。

#### **⚠️ 実務でのアンチパターン: WHERE 句での CASE 式の利用**

WHERE 句に直接 CASE 式を使用することは可能ですが、**実務的には非推奨**とされることが多いです。その理由は以下の通りです。

1. **可読性の低下**: CASE 式が複雑になると、WHERE 句の条件が非常に読みにくくなります。
2. **パフォーマンスの低下**: CASE 式は行ごとに評価されるため、インデックスが効率的に使用されにくくなる場合があります。これにより、大量のデータを扱う際にパフォーマンスが低下する可能性があります。

推奨される書き方:
上記のシナリオは、通常、論理演算子 (AND, OR) を使って以下のように記述するのが推奨されます。

```SQL
-- 推奨される書き方（論理演算子で制御）
SELECT
  product_name,
  price,
  stock_quantity
FROM
  products
WHERE
  (stock_quantity >= 100 AND price > 500) -- 在庫 100 以上かつ価格 500 超
OR
  (stock_quantity < 100); -- または在庫 100 未満（価格は問わない）
```

この書き方の方が、条件の意図が明確で、データベースのオプティマイザもより効率的に処理を実行しやすい傾向があります。CASE 式は便利ですが、乱用するとかえってクエリの可読性やパフォーマンスを損なう可能性があるため、使用場面を慎重に選ぶことが重要です。
💡 実務の現場では、ユーザーが指定した動的な検索条件（例えば、特定のパラメータが入力されたら A の条件、されなかったら B の条件）を SQL で表現するために CASE 式を使いたくなることがあります。しかし、多くの場合、そのような動的条件はアプリケーション層で SQL を組み立てるか、COALESCE 関数や論理演算子を組み合わせることで、よりパフォーマンス良く、かつ明確に表現できます。

### **例 5: 複数の CASE 式を使用した複雑なカテゴリ分類**

複数の条件を組み合わせて、より詳細なカテゴリ分類を行う例です。

**事前データ:**

transactions テーブル:

| transaction_id | amount | item_name  |
| :------------- | :----- | :--------- |
| 1              | 1500   | Laptop     |
| 2              | 50     | Coffee     |
| 3              | 300    | Books      |
| 4              | 2000   | Smartphone |
| 5              | 10     | Pencil     |
| 6              | 120    | Dinner     |
| 7              | 700    | Tablet     |

**SQL:**

```SQL
SELECT
  transaction_id,
  item_name,
  amount,
  CASE
    WHEN amount >= 1000 THEN '高額商品' -- 最も優先度の高い条件
    WHEN item_name IN ('Laptop', 'Smartphone', 'Tablet') THEN '電子機器'
    WHEN item_name IN ('Coffee', 'Dinner') THEN '飲食費'
    WHEN amount < 100 THEN '低額文具・雑貨'
    ELSE 'その他'
  END AS classified_category
FROM
  transactions;
```

**実行結果:**

| transaction_id | item_name  | amount | classified_category |
| :------------- | :--------- | :----- | :------------------ |
| 1              | Laptop     | 1500   | 高額商品            |
| 2              | Coffee     | 50     | 飲食費              |
| 3              | Books      | 300    | その他              |
| 4              | Smartphone | 2000   | 高額商品            |
| 5              | Pencil     | 10     | 低額文具・雑貨      |
| 6              | Dinner     | 120    | 飲食費              |
| 7              | Tablet     | 700    | 電子機器            |

**説明:**

- Laptop (amount=1500) は最初の WHEN amount >= 1000 で'高額商品'に分類されます。次の WHEN item_name IN ('Laptop', 'Smartphone')には到達しません。
- Coffee (amount=50) は最初の条件を満たさず、次の WHEN item_name IN ('Coffee', 'Dinner')を満たすため'飲食費'に分類されます。
- Books (amount=300) はどの WHEN 句も満たさないため、ELSE 'その他'に分類されます。
- Smartphone (amount=2000) も Laptop と同様に最初の条件で'高額商品'に分類されます。
- Tablet (amount=700) は最初の条件を満たさず、次の WHEN item_name IN ('Laptop', 'Smartphone', 'Tablet')を満たすため'電子機器'に分類されます。

このように、CASE 式は、複数の条件を複合的に評価し、複雑なビジネスロジックを SQL 内で表現する強力な手段となります。
💡 実務でよくある活用例としては、以下のようなものがあります。

- **請求明細の税区分**: CASE WHEN product_type \= 'FOOD' THEN 0.08 ELSE 0.10 END AS tax_rate
- **受注ステータスの集約**: CASE WHEN status IN ('PENDING', 'PROCESSING') THEN '進行中' WHEN status \= 'COMPLETED' THEN '完了' ELSE 'キャンセル' END AS simplified_status
- **年齢層の分類**: CASE WHEN age \< 20 THEN '10 代以下' WHEN age BETWEEN 20 AND 30 THEN '20 代' ... ELSE 'その他' END AS age_group

## **CTE（共通テーブル式 \- Common Table Expressions） (実務での重要度：高 ○)**

CTE は、WITH 句を使用して定義される一時的な結果セットです。複雑な SQL クエリを複数の論理的なステップに分割し、クエリの可読性とメンテナンス性を大幅に向上させることができます。CTE は、その CTE が定義された単一の SELECT, INSERT, UPDATE, DELETE 文の中でしか参照できません。

### **WITH 句：基本的な CTE の定義**

#### **基本構文**

```SQL
WITH CTE 名 AS (
SELECT ... \-- CTE の定義クエリ
)
SELECT ... \-- CTE を使用したメインクエリ
FROM CTE 名
\[JOIN ...\]
\[WHERE ...\]
```

複数の CTE をカンマで区切って定義することも可能です。

```SQL
WITH CTE1 AS (
SELECT ...
),
CTE2 AS (
SELECT ...
FROM CTE1 \-- CTE2 から CTE1 を参照することも可能
)
SELECT ...
FROM CTE2;
```

### **例 1: 複数ステップの計算を CTE で分かりやすく**

顧客ごとの合計購入金額を算出し、その中で合計金額が 1000 ドルを超える顧客だけを抽出します。

**事前データ:**

customers テーブル:

| customer_id | customer_name |
| :---------- | :------------ |
| 1           | Alice         |
| 2           | Bob           |
| 3           | Charlie       |

orders テーブル:

| order_id | customer_id | amount |
| :------- | :---------- | :----- |
| 101      | 1           | 500    |
| 102      | 2           | 700    |
| 103      | 1           | 600    |
| 104      | 3           | 300    |
| 105      | 2           | 400    |

**SQL:**

```SQL
WITH customer_total_sales AS (
  SELECT
    customer_id,
    SUM(amount) AS total_amount
  FROM
    orders
  GROUP BY
    customer_id
)
SELECT
  c.customer_name,
  cts.total_amount
FROM
  customers AS c
JOIN
  customer_total_sales AS cts ON c.customer_id = cts.customer_id
WHERE
  cts.total_amount > 1000;
```

**実行結果:**

| customer_name | total_amount |
| :------------ | :----------- |
| Alice         | 1100         |

説明:
customer_total_sales という CTE を定義することで、「顧客ごとの合計購入金額」という中間結果を明確に分離し、その結果を使って最終的なフィルタリングと結合を行っています。これにより、クエリ全体のロジックが理解しやすくなります。

#### **💡 コラム: CTE、サブクエリ、ビューの使い分けとパフォーマンス**

CTE は、**サブクエリ**や**ビュー**と似たような目的で使われますが、それぞれ特性が異なります。

| 特徴               | サブクエリ (派生テーブル)                                                  | ビュー (VIEW)                                          | CTE (共通テーブル式)                                                                      |
| :----------------- | :------------------------------------------------------------------------- | :----------------------------------------------------- | :---------------------------------------------------------------------------------------- |
| **定義場所**       | FROM 句内 (SELECT ... FROM (SELECT ...) AS alias)                          | データベースオブジェクトとして永続的に定義             | クエリの WITH 句内で一時的に定義                                                          |
| **再利用性**       | クエリ内では 1 回のみ (ネスト可能だが、同じ結果を複数回計算する可能性あり) | 複数クエリから参照可能（データベース全体で再利用）     | クエリ内でのみ複数回参照可能                                                              |
| **最適化**         | RDBMS がクエリ全体を最適化することが多いが、複雑になると難しい場合も       | RDBMS のオプティマイザが元のビュー定義を展開して最適化 | 多くの RDBMS でクエリ全体を最適化するが、一部のケースでは**実行計画のバリア**となることも |
| **パフォーマンス** | クエリの複雑さによる                                                       | 通常は良好（オプティマイザが頑張る）                   | 一般的に良好だが、特に大規模データでは注意が必要な場合も                                  |
| **メンテナンス性** | ネストが深くなると低下                                                     | 永続的な定義なので管理が必要                           | クエリのロジックを段階的に表現でき、可読性が高い                                          |

CTE のパフォーマンスに関する注意点:
特に PostgreSQL の場合、CTE はデフォルトで最適化されない（マテリアライズされる、つまり中間結果が一時的にディスクに書き込まれる）ことがあります。これにより、CTE の内部クエリが複雑で大量のデータを生成する場合、実行計画のバリアとなり、クエリ全体のパフォーマンスが低下する可能性があります。MySQL 8.0 以降や SQL Server、Oracle などでは CTE がより積極的にインライン化（中間結果が物理的に生成されずに、メインクエリに組み込まれる）される傾向があります。
したがって、CTE を使用する際は、そのパフォーマンス特性を理解し、クエリの実行計画を確認する習慣をつけることが重要です。

### **再帰 CTE（Recursive CTE） (実務での重要度：中 △)**

再帰 CTE は、階層構造やツリー構造のデータを展開するために使用されます。例えば、組織図の上下関係、部品構成表、SNS のフォロワー関係などです。

#### **基本構文**

```SQL
WITH RECURSIVE CTE 名 AS (
  -- 非再帰部分（アンカーメンバー）: 再帰の開始点となる行を選択
  SELECT ...
  UNION [ALL]
  -- 再帰部分（リカージョンメンバー）: CTE 自身を参照して次の行を生成
  SELECT ...
  FROM CTE 名 -- ここで CTE 自身を参照
  WHERE ... -- 終了条件
  )

SELECT ... -- CTE を使用したメインクエリ
FROM CTE 名;
```

- UNION（重複排除）または UNION ALL（重複保持）を使用します。パフォーマンスを考慮し、重複排除が不要な場合は UNION ALL を使用します。
- 再帰部分には、再帰を停止するための明確な終了条件が必要です。条件がない場合、無限ループに陥り、エラーになる可能性があります。

### **例 2: 階層構造の展開（従業員とその直属の上司・部下の関係）**

**事前データ:**

employees テーブル:

| employee_id | employee_name | manager_id |
| :---------- | :------------ | :--------- |
| 1           | Alice (CEO)   | NULL       |
| 2           | Bob           | 1          |
| 3           | Charlie       | 1          |
| 4           | David         | 2          |
| 5           | Eve           | 2          |
| 6           | Frank         | 3          |

**SQL:**

```SQL
WITH RECURSIVE employee_hierarchy AS (
-- アンカーメンバー: CEO (manager_id が NULL の従業員) を取得
SELECT
  employee_id,
  employee_name,
  manager_id,
  1 AS level -- 階層レベルを初期化
FROM
  employees
WHERE
  manager_id IS NULL

    UNION ALL

    -- 再帰部分: 上位の階層から次のレベルの部下を取得
    SELECT
        e.employee_id,
        e.employee_name,
        e.manager_id,
        eh.level + 1 AS level -- 階層レベルをインクリメント
    FROM
        employees AS e
    JOIN
        employee_hierarchy AS eh ON e.manager_id = eh.employee_id
    WHERE
        eh.level < 10 -- 💡実務Tips: 無限ループ回避とパフォーマンスのために最大階層制限

)
SELECT
  employee_id,
  employee_name,
  manager_id,
  level
FROM
  employee_hierarchy
ORDER BY
  level, employee_id;
```

実行確認用

```SQL
with RECURSIVE
employees as  (
	select 1 as employee_id, 'Alice (CEO)' as employee_name, NULL as manager_id union all
	select 2 as employee_id, 'Bob        ' as employee_name, 1    as manager_id union all
	select 3 as employee_id, 'Charlie    ' as employee_name, 1    as manager_id union all
	select 4 as employee_id, 'David      ' as employee_name, 2    as manager_id union all
	select 5 as employee_id, 'Eve        ' as employee_name, 2    as manager_id union all
	select 6 as employee_id, 'Frank      ' as employee_name, 3    as manager_id
),
employee_hierarchy AS (
-- アンカーメンバー: CEO (manager_id が NULL の従業員) を取得
SELECT
  employee_id,
  employee_name,
  manager_id,
  1 AS level -- 階層レベルを初期化
FROM
  employees
WHERE
  manager_id IS NULL

    UNION ALL

    -- 再帰部分: 上位の階層から次のレベルの部下を取得
    SELECT
        e.employee_id,
        e.employee_name,
        e.manager_id,
        eh.level + 1 AS level -- 階層レベルをインクリメント
    FROM
        employees AS e
    JOIN
        employee_hierarchy AS eh ON e.manager_id = eh.employee_id
    WHERE
        eh.level < 10 -- 💡実務Tips: 無限ループ回避とパフォーマンスのために最大階層制限

)
SELECT
  employee_id,
  employee_name,
  manager_id,
  level
FROM
  employee_hierarchy
ORDER BY
  level, employee_id;
```

**実行結果:**

| employee_id | employee_name | manager_id | level |
| :---------- | :------------ | :--------- | :---- |
| 1           | Alice (CEO)   | NULL       | 1     |
| 2           | Bob           | 1          | 2     |
| 3           | Charlie       | 1          | 2     |
| 4           | David         | 2          | 3     |
| 5           | Eve           | 2          | 3     |
| 6           | Frank         | 3          | 3     |

**説明:**

1. **アンカーメンバー**: 最初に manager_id が NULL である Alice を level=1 として選択します。
2. **再帰部分**: employee_hierarchy（一つ前の結果セット）と employees テーブルを JOIN し、employee_hierarchy の employee_id が employees の manager_id と一致する行（部下）を検索します。見つかった部下は level が+1 されて追加されます。
3. このプロセスは、新たな部下が見つからなくなるまで繰り返され、最終的に全従業員の階層が展開されます。
   💡 実務 Tips: 無限ループとパフォーマンスの問題を回避するため、再帰 CTE には WHERE eh.level < \[最大階層数\]のような最大階層制限を設けることが非常に重要です。予期せぬ循環参照が存在する場合でも、クエリが暴走するのを防ぐことができます。

## **ウィンドウ関数 (Window Functions) (実務での重要度：非常に高 ◎)**

ウィンドウ関数は、特定の行のセット（**ウィンドウ**）に対して計算を実行し、その結果を**各行に返す**機能です。これは GROUP BY 句と大きく異なります。GROUP BY が集計によって行数を減らす（まとめる）のに対し、ウィンドウ関数は元の行数を維持したまま、集計結果やランキングなどを各行のコンテキストに付加できます。

### **基本構文**

```SQL
関数名()
OVER (
  [PARTITION BY 列リスト]
  [ORDER BY 列リスト [ASC|DESC] [, ...]]
  [ウィンドウフレーム]
)
```

- **OVER()句**: ウィンドウ関数であることを示し、その後の括弧内でウィンドウの定義を行います。
- **PARTITION BY 句 (オプション)**: データを 1 つ以上の列の値に基づいて論理的なパーティション（グループ）に分割します。このパーティション内で関数が独立して計算されます。GROUP BY のグループ化と似ていますが、行は集約されません。
- **ORDER BY 句 (オプション)**: 各パーティション内で、行の順序を定義します。ランキング関数や移動平均などで非常に重要です。
- **ウィンドウフレーム (オプション)**: ORDER BY 句と組み合わせて、現在の行を中心とした計算範囲（フレーム）を定義します。

<details>
<summary>フレームの詳細</summary>

### フレームの基本構造

ウィンドウ関数の OVER 句に追加できます：

```SQL
OVER (
  PARTITION BY ...
  ORDER BY ...
  ROWS | RANGE | GROUPS BETWEEN <開始> AND <終了>
)
```

フレームを構成する要素

- 種類

  - ROWS … 物理的な「行数」で範囲を決める
  - RANGE … 「値の範囲」で決める（ORDER BY 必須）
  - GROUPS … 「同順位グループ単位」で決める

- 開始 / 終了
  - UNBOUNDED PRECEDING … 先頭まで
  - <N> PRECEDING … N 行（または N 単位）前まで
  - CURRENT ROW … 今の行
  - <N> FOLLOWING … N 行（または N 単位）後まで
  - UNBOUNDED FOLLOWING … 末尾まで

### デフォルトのフレーム

- ORDER BY なしの場合
  → パーティション全体が対象（全期間合計など）

- ORDER BY ありの場合
  → RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  （種類：RANGE、開始：先頭、終了：今の行。つまり「先頭から現在行まで」＝累計）

### フレームの種類ごとの特徴

#### (A)ROWS

- 物理的に「前後 ◯ 行」を指定
- 累計や移動平均に向いている

##### 例: 直近 3 行の平均

```SQL
AVG(amount) OVER (
  ORDER BY order_date
  ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
)
```

#### (B) RANGE

- ORDER BY の値の範囲で指定する
- 日付や数値の「連続範囲」に便利
- デフォルトは「値が同じものはまとめて対象」

##### 例: 当日までの累計（同じ日付を一緒に計算）

```SQL
SUM(amount) OVER (
  ORDER BY order_date
  RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)
```

##### 例: 「当日から前 3 日分の売上」

```SQL
SUM(amount) OVER (
  ORDER BY order_date
  RANGE BETWEEN INTERVAL '3 day' PRECEDING AND CURRENT ROW
)
```

#### (C) GROUPS

- ORDER BY の「順位グループ単位」で指定
- RANK() や DENSE_RANK() と一緒に使うと便利

##### 例: 「現在の順位グループとその前 1 グループの合計」

```SQL
SUM(sales) OVER (
  ORDER BY score
  GROUPS BETWEEN 1 PRECEDING AND CURRENT ROW
)
```

#### 主な利用パターン

- 用途:フレーム指定例
- 累計:
  - RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
- 全体合計:
  - RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
- 移動平均:
  - ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
- 直前値との比較:
  - ROWS BETWEEN 1 PRECEDING AND 1 PRECEDING
- 直後 2 行の平均:
  - ROWS BETWEEN 1 FOLLOWING AND 2 FOLLOWING

#### 注意点

- ROWS はシンプル（行数で数える）
- RANGE は値ベースで、同じ値を持つ行が全部含まれる
- GROUPS は順位単位なので「同順位の行」を一括で扱える

#### まとめイメージ図

```
ウィンドウ関数のフレーム
├─ ROWS → 前後 ◯ 行
├─ RANGE → ORDER BY 値の範囲
└─ GROUPS → 同順位グループ単位
```

</details>

### **ランキング関数 (Ranking Functions)**

特定のパーティション内での順位を決定します。

#### **ROW_NUMBER(): 連番 (実務での重要度：高)**

各パーティション内で、ORDER BY 句で指定された順序に基づき、**重複を考慮せず**に 1 からのユニークな連番を付与します。

**例:** 各カテゴリ内で、価格が高い順に商品に連番を付与。

事前データ:
（products テーブルを使用）
**SQL:**

```SQL
SELECT
  product_name,
  price,
  category,
  ROW_NUMBER() OVER (PARTITION BY category ORDER BY price DESC) AS rank_by_price_in_category
FROM
  products
ORDER BY
  category, rank_by_price_in_category;
```

**実行結果:**

| product_name | price   | category    | rank_by_price_in_category |
| :----------- | :------ | :---------- | :------------------------ |
| Laptop       | 1200.00 | Electronics | 1                         |
| Keyboard     | 75.00   | Electronics | 2                         |
| Mouse        | 25.50   | Electronics | 3                         |
| Headset      | 150.00  | Peripherals | 1                         |
| Webcam       | 50.00   | Peripherals | 2                         |
| Monitor      | 300.00  | NULL        | 1                         |
| (NULL)       | 10.00   | Peripherals | 3                         |

### **RANK() / DENSE_RANK(): 順位付け (実務での重要度：中)**

- **RANK()**: 同じ値の行には同じ順位を付与し、次の異なる値の順位は、同順位の数を飛ばして付けられます。
- **DENSE_RANK()**: 同じ値の行には同じ順位を付与し、次の異なる値の順位は、同順位の数を飛ばさずに連続して付けられます。

**例:** 価格の同順位を考慮したランキング。

**事前データ:**

scores テーブル:

| student_id | score |
| :--------- | :---- |
| 1          | 90    |
| 2          | 85    |
| 3          | 90    |
| 4          | 70    |
| 5          | 85    |

**SQL:**

```SQL
SELECT
  student_id,
  score,
  RANK() OVER (ORDER BY score DESC) AS rank_score,
  DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank_score
FROM
  scores
ORDER BY
  score DESC, student_id;
```

**実行結果:**

| student_id | score | rank_score | dense_rank_score |
| :--------- | :---- | :--------- | :--------------- |
| 1          | 90    | 1          | 1                |
| 3          | 90    | 1          | 1                |
| 2          | 85    | 3          | 2                |
| 5          | 85    | 3          | 2                |
| 4          | 70    | 5          | 3                |

**説明:**

- RANK()では、90 点の 2 人が 1 位の後に、85 点の 2 人は 3 位（2 位が飛ばされる）となります。
- DENSE_RANK()では、90 点の 2 人が 1 位の後に、85 点の 2 人は 2 位（順位が詰まる）となります。

### **集計ウィンドウ (Aggregate Window Functions) (実務での重要度：非常に高)**

SUM(), AVG(), COUNT(), MIN(), MAX()などの集計関数を OVER()句と共に使用することで、グループ内の集計値を各行に表示できます。

#### **例: 各商品のカテゴリ内での合計価格と平均価格を算出**

事前データ:
（products テーブルを使用）
**SQL:**

```SQL
SELECT
  product_name,
  price,
  category,
  SUM(price) OVER (PARTITION BY category) AS total_price_in_category,
  AVG(price) OVER (PARTITION BY category) AS avg_price_in_category
FROM
  products
ORDER BY
  category, product_name;
```

**実行結果:**

| product_name | price   | category    | total_price_in_category | avg_price_in_category |
| :----------- | :------ | :---------- | :---------------------- | :-------------------- |
| Keyboard     | 75.00   | Electronics | 1300.50                 | 433.50                |
| Laptop       | 1200.00 | Electronics | 1300.50                 | 433.50                |
| Mouse        | 25.50   | Electronics | 1300.50                 | 433.50                |
| Headset      | 150.00  | Peripherals | 210.00                  | 70.00                 |
| Webcam       | 50.00   | Peripherals | 210.00                  | 70.00                 |
| (NULL)       | 10.00   | Peripherals | 210.00                  | 70.00                 |
| Monitor      | 300.00  | NULL        | 300.00                  | 300.00                |

説明:
SUM(price) OVER (PARTITION BY category) は、category で区切られたグループ内での price の合計を計算し、その結果を各行に表示します。GROUP BY とは異なり、行の数は減りません。

#### **💡 実務で頻出のシナリオ: 累計売上 (Running Total)**

BI ツールやレポート作成で非常によく使われるのが、時系列データにおける累計値の計算です。

**例: 顧客ごとの注文の累計購入額**

**事前データ:**

orders テーブル:

| order_id | customer_id | order_date   | amount |
| :------- | :---------- | :----------- | :----- |
| 101      | 1           | '2023-01-01' | 100    |
| 102      | 2           | '2023-01-01' | 50     |
| 103      | 1           | '2023-01-05' | 150    |
| 104      | 2           | '2023-01-07' | 70     |
| 105      | 1           | '2023-01-10' | 200    |

**SQL:**

```SQL
SELECT
  customer_id,
  order_date,
  amount,
  SUM(amount) OVER (PARTITION BY customer_id ORDER BY order_date) AS cumulative_amount
FROM
  orders
ORDER BY
  customer_id, order_date;
```

**実行結果:**

| customer_id | order_date | amount | cumulative_amount |
| :---------- | :--------- | :----- | :---------------- |
| 1           | 2023-01-01 | 100    | 100               |
| 1           | 2023-01-05 | 150    | 250               |
| 1           | 2023-01-10 | 200    | 450               |
| 2           | 2023-01-01 | 50     | 50                |
| 2           | 2023-01-07 | 70     | 120               |

説明:

- PARTITION BY customer_id で顧客ごとに区切り、ORDER BY order_date で注文日順に並べます。SUM(amount) OVER (...)は、各顧客の、その時点までの注文金額の合計（累計）を計算します。
- ウィンドウフレームを省略した場合、デフォルトで RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW となり、パーティションの最初から現在の行までの累積値が計算されます。

### **移動平均 (Moving Average) (実務での重要度：高)**

時系列データにおいて、特定期間（ウィンドウフレーム）内の平均値を計算し、各時点でのトレンドを滑らかにする分析手法です。

#### **例: 日次売上の 3 日間移動平均**

**事前データ:**

daily_sales テーブル:

| sale_date    | amount |
| :----------- | :----- |
| '2023-01-01' | 100    |
| '2023-01-02' | 120    |
| '2023-01-03' | 110    |
| '2023-01-04' | 150    |
| '2023-01-05' | 130    |
| '2023-01-06' | 140    |

**SQL:**

```SQL
SELECT
  sale_date,
  amount,
  AVG(amount) OVER (
    ORDER BY sale_date
    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW -- 現在の行と過去 2 行（合計 3 日間）
  ) AS three_day_moving_avg
FROM
  daily_sales
ORDER BY
  sale_date;
```

**実行結果:**

| sale_date  | amount | three_day_moving_avg |
| :--------- | :----- | :------------------- |
| 2023-01-01 | 100    | 100.0                |
| 2023-01-02 | 120    | 110.0                |
| 2023-01-03 | 110    | 110.0                |
| 2023-01-04 | 150    | 126.66666666666667   |
| 2023-01-05 | 130    | 130.0                |
| 2023-01-06 | 140    | 140.0                |

**説明:**

- ROWS BETWEEN 2 PRECEDING AND CURRENT ROW は、現在の行と、その前の 2 行（合計 3 行）をウィンドウフレームとして指定します。
- 例えば、'2023-01-03' の移動平均は (100 + 120 + 110) / 3 = 110.0 となります。
- ウィンドウフレームを省略した場合、デフォルトで RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW となり、パーティションの最初から現在の行までの累積値が計算されます。
  💡\*\*「直近 N 件の平均」を計算する際も、この ROWS BETWEEN 句を使ってウィンドウフレームを調整します。例えば、直近 5 件の平均であれば ROWS BETWEEN 4 PRECEDING AND CURRENT ROW となります。\*\*

#### **💡 コラム: GROUP BY とウィンドウ関数の違い**

| 特徴               | GROUP BY 句                                       | ウィンドウ関数 (OVER())                                    |
| :----------------- | :------------------------------------------------ | :--------------------------------------------------------- |
| **行の数**         | 集約によって行数が**減る**（グループごとに 1 行） | 元の行数を**維持**したまま各行に結果を付加                 |
| **集計のスコープ** | 全テーブルまたは WHERE 句で絞り込まれた行全体     | PARTITION BY で定義された論理的なグループ（ウィンドウ）内  |
| **利用場所**       | SELECT 句、HAVING 句 (GROUP BY とセット)          | SELECT 句、ORDER BY 句                                     |
| **結果の利用方法** | グループ単位での集計結果のみ                      | 個々の行のコンテキストで集計結果やランキングなどを利用可能 |

実務では、全体の集計値が必要な場合は GROUP BY、各行の隣接するデータやグループ内での相対的な情報が必要な場合はウィンドウ関数と使い分けます。

## **RDBMS 間の違いに注意！ (実務での重要度：中)**

CASE 式、CTE、ウィンドウ関数は多くの RDBMS で標準的にサポートされていますが、細かい構文や挙動、パフォーマンス特性には差異があります。

| 機能/概念                 | PostgreSQL              | MySQL                  | Oracle                           | SQL Server   |
| :------------------------ | :---------------------- | :--------------------- | :------------------------------- | :----------- |
| CASE 式                   | 標準サポート            | 標準サポート           | 標準サポート                     | 標準サポート |
| WITH 句 (CTE)             | 標準サポート            | 8.0 以降で標準サポート | 標準サポート                     | 標準サポート |
| 再帰 CTE (WITH RECURSIVE) | サポートあり            | 8.0 以降でサポート     | WITH ... CONNECT BY (異なる構文) | サポートあり |
| ウィンドウ関数            | 標準サポート            | 8.0 以降で標準サポート | 標準サポート                     | 標準サポート |
| ROWS BETWEEN の挙動       | 標準的                  | 標準的                 | 標準的                           | 標準的       |
| FILTER 句 (集計関数)      | PostgreSQL のみサポート | サポートなし           | サポートなし                     | サポートなし |

特に、MySQL では古いバージョンで CTE やウィンドウ関数がサポートされていなかったため、バージョンの確認が必要です。Oracle の再帰クエリは CONNECT BY 句という独自の構文も持っています。

実務で特定の RDBMS を使用する際は、必ずその RDBMS の公式ドキュメントを確認するようにしましょう。
