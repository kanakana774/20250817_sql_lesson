# **SQL 基礎：複数テーブル操作とサブクエリ**

## **導入：複数テーブル操作の必要性**

データベースを設計する際、データの重複を避け、整合性を保つために、関連するデータを複数のテーブルに分けて格納することが一般的です。これを「正規化」と呼びます。

正規化の必要性 (概念):  
データを一つの大きなテーブルにまとめてしまうと、以下のような問題が発生しやすくなります。

- **データの重複 (Redundancy)**: 同じ情報が複数の場所に存在すると、ストレージを無駄にするだけでなく、更新時の不整合（例: 顧客の住所変更があった際、複数のレコードを修正し忘れる）を引き起こす可能性があります。
- **データの不整合 (Inconsistency)**: 重複するデータ間で内容が異なる状態が発生しやすくなり、信頼性が低下します。
- **更新異常 (Update Anomaly)**: データの更新、挿入、削除が複雑になり、意図しない結果を招くことがあります。

正規化は、これらの問題を最小限に抑え、データベースのデータが論理的かつ効率的に格納されるようにするプロセスです。具体的には、関連するデータを分離し、テーブル間の関係を外部キーで定義することで、データの重複を減らし、整合性を高めます。

しかし、分析やアプリケーションの目的によっては、これらの分散されたデータを組み合わせて取得する必要があります。

ここで、複数のテーブルを結合する\*\*JOIN**、複数のクエリ結果を縦に結合する**集合演算\*\*、そしてクエリの中に別のクエリを埋め込む**サブクエリ**が非常に重要になります。

## **テーブルの結合：JOIN (実務での重要度：非常に高)**

JOIN 句は、共通の列（キー）を持つ複数のテーブルの行を結合し、新しい結果セットを作成するために使用します。

### **結合の基本**

データは通常、以下のような形で複数のテーブルに分かれて格納されています。

- **products テーブル**: product_id, product_name, price, category_id
- **categories テーブル**: category_id, category_name

products テーブルの category_id は categories テーブルの category_id を参照する**外部キー**となっています。これにより、商品がどのカテゴリに属するかを関連付けることができます。

### **INNER JOIN：内部結合**

両方のテーブルで結合条件に一致する行のみを返します。最も一般的な結合の種類です。

#### **基本構文**

SELECT 列リスト  
FROM テーブル 1  
INNER JOIN テーブル 2 ON 結合条件;

#### **例: 商品名とカテゴリ名を取得**

**事前データ:**

products テーブル:

| product_id | product_name | price   | category_id |
| :--------- | :----------- | :------ | :---------- |
| 1          | Laptop       | 1200.00 | 1           |
| 2          | Mouse        | 25.50   | 1           |
| 3          | Keyboard     | 75.00   | 1           |
| 4          | Monitor      | 300.00  | NULL        |
| 5          | Webcam       | 50.00   | 2           |

categories テーブル:

| category_id | category_name |
| :---------- | :------------ |
| 1           | Electronics   |
| 2           | Peripherals   |
| 3           | Mobile        |

**SQL:**

SELECT  
 p.product_name,  
 p.price,  
 c.category_name  
FROM  
 products AS p \-- p は products テーブルのエイリアス（別名）  
INNER JOIN  
 categories AS c ON p.category_id \= c.category_id;

**実行結果:**

| product_name | price   | category_name |
| :----------- | :------ | :------------ |
| Laptop       | 1200.00 | Electronics   |
| Mouse        | 25.50   | Electronics   |
| Keyboard     | 75.00   | Electronics   |
| Webcam       | 50.00   | Peripherals   |

このクエリでは、products テーブルの category_id と categories テーブルの category_id が一致する行のみが結合されます。products テーブルの Monitor（category_id が NULL）は、categories テーブルに一致する行がないため結果に含まれません。

### **LEFT JOIN / RIGHT JOIN：外部結合**

片方のテーブルの全ての行を保持し、もう片方のテーブルから一致する行があれば結合します。一致する行がない場合は、結合先のテーブルの列は NULL になります。

- **LEFT JOIN (または LEFT OUTER JOIN)**: FROM 句で指定した**左側のテーブル**の全ての行を返します。
- **RIGHT JOIN (または RIGHT OUTER JOIN)**: JOIN 句で指定した**右側のテーブル**の全ての行を返します。

#### **基本構文**

\-- LEFT JOIN  
SELECT 列リスト  
FROM テーブル 1  
LEFT JOIN テーブル 2 ON 結合条件;

\-- RIGHT JOIN  
SELECT 列リスト  
FROM テーブル 1  
RIGHT JOIN テーブル 2 ON 結合条件;

#### **例: 全ての商品と、それに紐づくカテゴリ名を取得（カテゴリが未登録の商品も含む）**

事前データ:  
（INNER JOIN の例と同じ products と categories テーブルを使用）  
**SQL:**

SELECT  
 p.product_name,  
 p.price,  
 c.category_name  
FROM  
 products AS p  
LEFT JOIN  
 categories AS c ON p.category_id \= c.category_id;

**実行結果:**

| product_name | price   | category_name |
| :----------- | :------ | :------------ |
| Laptop       | 1200.00 | Electronics   |
| Mouse        | 25.50   | Electronics   |
| Keyboard     | 75.00   | Electronics   |
| Monitor      | 300.00  | NULL          |
| Webcam       | 50.00   | Peripherals   |

この例では、products テーブルの Monitor のように category_id が NULL である商品や、categories テーブルに存在しない category_id が設定されている商品でも、その行は結果に含まれ、category_name は NULL になります。

RIGHT JOIN は LEFT JOIN の左右を逆にしたものであり、LEFT JOIN で書き換え可能です。例えば、上記の LEFT JOIN は以下のように RIGHT JOIN で同じ結果を得られます。

SELECT  
 p.product_name,  
 p.price,  
 c.category_name  
FROM  
 categories AS c \-- カテゴリテーブルを左に持ってきた  
RIGHT JOIN  
 products AS p ON p.category_id \= c.category_id; \-- プロダクトテーブルを右に持ってきた

そのため、実務では LEFT JOIN が使われることが多いです。

### **FULL OUTER JOIN：完全外部結合 (実務での重要度：中)**

両方のテーブルの全ての行を返します。結合条件に一致する行がない場合は、対応するテーブルの列は NULL になります。

#### **基本構文**

SELECT 列リスト  
FROM テーブル 1  
FULL OUTER JOIN テーブル 2 ON 結合条件;

#### **例: 全ての商品と全てのカテゴリを結合（双方に一致しないものも含む）**

もし products テーブルに存在しないカテゴリ ID を持つ商品があったり、categories テーブルにあるが商品が一つもないカテゴリがあったりする場合に有効です。

**事前データ:**

products テーブル:

| product_id | product_name | price   | category_id |
| :--------- | :----------- | :------ | :---------- |
| 1          | Laptop       | 1200.00 | 1           |
| 2          | Mouse        | 25.50   | 1           |
| 3          | Keyboard     | 75.00   | 1           |
| 4          | Monitor      | 300.00  | NULL        |
| 5          | Webcam       | 50.00   | 2           |

categories テーブル:

| category_id | category_name |
| :---------- | :------------ |
| 1           | Electronics   |
| 2           | Peripherals   |
| 3           | Mobile        |
| 4           | Books         |

**SQL:**

SELECT  
 p.product_name,  
 c.category_name  
FROM  
 products AS p  
FULL OUTER JOIN  
 categories AS c ON p.category_id \= c.category_id;

**実行結果:**

| product_name | category_name |
| :----------- | :------------ |
| Laptop       | Electronics   |
| Mouse        | Electronics   |
| Keyboard     | Electronics   |
| Webcam       | Peripherals   |
| Monitor      | NULL          |
| NULL         | Mobile        |
| NULL         | Books         |

この結果では、Monitor のようにカテゴリに紐づかない商品（category_name が NULL）も、Mobile や Books のように商品が一つもないカテゴリ（product_name が NULL）も表示されます。

#### **💡 コラム: FULL OUTER JOIN がサポートされない RDBMS (MySQL の例)**

MySQL などの一部の RDBMS では FULL OUTER JOIN が直接サポートされていません。その場合、以下のように LEFT JOIN と RIGHT JOIN を UNION ALL で組み合わせることで同等の結果を得られます。

\-- MySQL などで FULL OUTER JOIN の代替として使用  
SELECT  
 p.product_name,  
 c.category_name  
FROM  
 products AS p  
LEFT JOIN  
 categories AS c ON p.category_id \= c.category_id  
UNION ALL  
SELECT  
 p.product_name,  
 c.category_name  
FROM  
 products AS p  
RIGHT JOIN  
 categories AS c ON p.category_id \= c.category_id  
WHERE  
 p.product_id IS NULL; \-- LEFT JOIN で得られない右側の NULL 行のみを抽出

### **自己結合 (Self-Join) (実務での重要度：中)**

同じテーブルを 2 回以上、異なるエイリアス（別名）で JOIN することです。テーブル内の行同士の関係性を表現したい場合によく使われます。

#### **例: 従業員とその上司の名前を取得**

employees テーブルが employee_id と employee_name、そして上司の employee_id を指す manager_id 列を持っていると仮定します。

**事前データ:**

employees テーブル:

| employee_id | employee_name | manager_id |
| :---------- | :------------ | :--------- |
| 1           | Alice         | NULL       |
| 2           | Bob           | 1          |
| 3           | Charlie       | 1          |
| 4           | David         | 2          |

**SQL:**

SELECT  
 e.employee_name AS employee, \-- 従業員自身の名前  
 m.employee_name AS manager \-- 上司の名前  
FROM  
 employees AS e  
LEFT JOIN \-- 上司がいない従業員（CEO など）も表示するため LEFT JOIN  
 employees AS m ON e.manager_id \= m.employee_id;

**実行結果:**

| employee | manager |
| :------- | :------ |
| Alice    | NULL    |
| Bob      | Alice   |
| Charlie  | Alice   |
| David    | Bob     |

このクエリでは、employees テーブルを e（従業員）と m（上司）という 2 つの論理的なテーブルとして扱い、それぞれの employee_id と manager_id を結合しています。Alice は manager_id が NULL なので、LEFT JOIN によって manager が NULL で表示されます。

### **ON 句と WHERE 句の違い (実務での重要度：非常に高)**

JOIN における ON 句と WHERE 句は、どちらも条件を指定しますが、その**評価されるタイミングと効果**が大きく異なります。この違いは特に INNER JOIN 以外の結合（LEFT/RIGHT/FULL OUTER JOIN）で顕著になります。

| 特徴                 | ON 句                                      | WHERE 句                                                            |
| :------------------- | :----------------------------------------- | :------------------------------------------------------------------ |
| **評価タイミング**   | **結合前**（テーブルを結合する際の条件）   | **結合後**（結合された結果セット全体に対するフィルタリング）        |
| **効果**             | 結合の方法や結合対象となる行を定義する     | 結合された結果から、最終的に表示する行を絞り込む                    |
| **外部結合での違い** | 一致しない行も元のテーブルの側は保持される | 一致しない行（NULL になっている行）も条件に合致しなければ削除される |

#### **例: LEFT JOIN における違い**

products テーブルと categories テーブルを考えます。

**シナリオ:** カテゴリ ID が 1 の商品、およびカテゴリに紐づかない全ての商品を表示したい。

事前データ:  
（INNER JOIN の例と同じ products と categories テーブルを使用）

1. **ON 句で条件を指定した場合:**  
   **SQL:**  
   SELECT  
    p.product_name,  
    c.category_name,  
    p.category_id  
   FROM  
    products AS p  
   LEFT JOIN  
    categories AS c ON p.category_id \= c.category_id AND c.category_id \= 1;

   **実行結果:**

| product_name | category_name | category_id |
| :----------- | :------------ | :---------- |
| Laptop       | Electronics   | 1           |
| Mouse        | Electronics   | 1           |
| Keyboard     | Electronics   | 1           |
| Monitor      | NULL          | NULL        |
| Webcam       | NULL          | 2           |

2.  説明:  
    products テーブルの全ての行が保持されます。categories テーブルからは、category_id \= 1 の行だけが結合の対象になります。products.category_id が 1 ではない商品（例: Webcam category_id=2）や、products.category_id が NULL の商品（例: Monitor category_id=NULL）の場合、categories テーブルとの結合条件 c.category_id \= 1 を満たさないため、c.category_name は NULL になります。つまり、左側のテーブルの行は維持され、右側のテーブル側の結合条件が満たされない場合は NULL が埋められます。
3.  **WHERE 句で条件を指定した場合:**  
    **SQL:**  
    SELECT  
     p.product_name,  
     c.category_name,  
     p.category_id  
    FROM  
     products AS p  
    LEFT JOIN  
     categories AS c ON p.category_id \= c.category_id  
    WHERE  
     c.category_id \= 1;

    **実行結果:**

| product_name | category_name | category_id |
| :----------- | :------------ | :---------- |
| Laptop       | Electronics   | 1           |
| Mouse        | Electronics   | 1           |
| Keyboard     | Electronics   | 1           |

4.  説明:  
    まず products と categories が category_id で LEFT JOIN されます。この時点では Monitor と Webcam も category_name が NULL として含まれます。その後、結合された結果セット全体に対して c.category_id \= 1 という条件が適用されます。これにより、categories テーブルからの結合結果が NULL であった行（Monitor）や、category_id が 1 以外だった行（Webcam）は、WHERE c.category_id \= 1 の条件を満たさないため、結果セットから除外されます。これは実質的に INNER JOIN の結果と同じになってしまうことがよくあります。

この違いを理解することは、複雑なクエリを作成する上で非常に重要です。ON 句は「結合の仕方を定義する」ために、WHERE 句は「結合された結果を最終的にフィルタリングする」ために使われます。

## **集合演算 (Set Operations) (実務での重要度：中)**

集合演算は、複数の SELECT 文の結果を垂直に結合するために使用されます。各 SELECT 文の列の数とデータ型が一致している必要があります。

### **UNION：和集合（重複排除）**

複数の SELECT 文の結果を結合し、**重複する行を排除**して返します。

#### **基本構文**

SELECT 列リスト FROM テーブル 1  
UNION  
SELECT 列リスト FROM テーブル 2;

#### **例: 商品名とカテゴリ名の両方を取得し、重複を除外**

例えば、products テーブルから商品名を、categories テーブルからカテゴリ名を取得し、重複する名前を一度だけ表示したい場合。

**事前データ:**

products テーブル (抜粋):

| product_name |
| :----------- |
| Laptop       |
| Electronics  |
| Mouse        |
| Smartphone   |
| (NULL)       |

categories テーブル (抜粋):

| category_name |
| :------------ |
| Electronics   |
| Peripherals   |
| Mobile        |
| Smartphone    |
| (NULL)        |

**SQL:**

SELECT product_name FROM products  
UNION  
SELECT category_name FROM categories;

**実行結果:**

| name        |
| :---------- |
| Electronics |
| Laptop      |
| Mobile      |
| Mouse       |
| NULL        |
| Peripherals |
| Smartphone  |

この結果には、products と categories の共通の名前（Electronics, Smartphone）は一度だけ表示されます。また、どちらかのテーブルに NULL 値が存在する場合、UNION は NULL も一つの値として扱い、重複を除外します。

### **UNION ALL：和集合（重複保持）**

複数の SELECT 文の結果を結合し、**重複する行も全て含めて**返します。UNION よりも高速に動作することが多いため、重複排除が不要な場合はこちらを推奨します。

#### **基本構文**

SELECT 列リスト FROM テーブル 1  
UNION ALL  
SELECT 列リスト FROM テーブル 2;

#### **例: 全ての商品名とカテゴリ名を全て取得（重複も保持）**

事前データ:  
（UNION の例と同じデータを使用）  
**SQL:**

SELECT product_name FROM products  
UNION ALL  
SELECT category_name FROM categories;

**実行結果:**

| name        |
| :---------- |
| Laptop      |
| Electronics |
| Mouse       |
| Smartphone  |
| NULL        |
| Electronics |
| Peripherals |
| Mobile      |
| Smartphone  |
| NULL        |

この結果には、products と categories の全ての名前が、重複も含めて表示されます。

### **INTERSECT：積集合 (実務での重要度：低)**

複数の SELECT 文の結果に**共通して存在する行のみ**を返します。

#### **基本構文**

SELECT 列リスト FROM テーブル 1  
INTERSECT  
SELECT 列リスト FROM テーブル 2;

#### **例: 商品名とカテゴリ名で共通する名前を取得**

事前データ:  
（UNION の例と同じデータを使用）  
**SQL:**

SELECT product_name FROM products  
INTERSECT  
SELECT category_name FROM categories;

**実行結果:**

| name        |
| :---------- |
| Electronics |
| Smartphone  |

もし products テーブルに'Electronics'という名前の商品があり、categories テーブルにも'Electronics'というカテゴリ名があれば、その行だけが結果として表示されます。同様に Smartphone も共通なので表示されます。

### **EXCEPT / MINUS：差集合 (実務での重要度：低)**

最初の SELECT 文の結果から、2 番目の SELECT 文の結果に**存在しない行のみ**を返します。

- **EXCEPT**: PostgreSQL、SQL Server などで使用されます。
- **MINUS**: Oracle で使用されます。

#### **基本構文**

\-- PostgreSQL / SQL Server  
SELECT 列リスト FROM テーブル 1  
EXCEPT  
SELECT 列リスト FROM テーブル 2;

\-- Oracle  
SELECT 列リスト FROM テーブル 1  
MINUS  
SELECT 列リスト FROM テーブル 2;

#### **例: products テーブルに存在するが、categories テーブルには存在しない名前を取得**

事前データ:  
（UNION の例と同じデータを使用）  
**SQL:**

SELECT product_name FROM products  
EXCEPT  
SELECT category_name FROM categories;

**実行結果:**

| name   |
| :----- |
| Laptop |
| Mouse  |

この結果には、'Laptop'や'Mouse'のように、商品名としては存在するがカテゴリ名としては存在しない名前だけが表示されます。

#### **💡 コラム: EXCEPT / MINUS がサポートされない RDBMS (MySQL の例)**

MySQL では EXCEPT や MINUS は直接サポートされていません。同等の結果を得るには、LEFT JOIN と WHERE IS NULL、または NOT EXISTS を組み合わせる方法が一般的です。

\-- MySQL などで EXCEPT の代替として使用  
SELECT p.product_name  
FROM products AS p  
LEFT JOIN categories AS c ON p.product_name \= c.category_name  
WHERE c.category_name IS NULL;

\-- または NOT EXISTS を使用  
SELECT p.product_name  
FROM products AS p  
WHERE NOT EXISTS (SELECT 1 FROM categories AS c WHERE c.category_name \= p.product_name);

### **JOIN と 集合演算の使い分け**

- **JOIN**: 異なるテーブルの**関連する列**を**水平に**結合して、**行を拡張**する場合に使用します。通常、データモデルで定義されたリレーションシップ（外部キーなど）に基づいてデータを組み合わせる際に使います。
- **集合演算 (UNION, INTERSECT, EXCEPT)**: 複数の SELECT 文の**結果セット全体**を**垂直に**結合して、**行を追加**する場合に使用します。各クエリの列構成が同じである必要があります。異なるテーブルから似たような形式のデータをまとめて表示したい場合などに使います。

## **サブクエリ (Subqueries) (実務での重要度：高)**

サブクエリとは、**SQL クエリの中に記述される別の SQL クエリ**のことです。内側のクエリが先に実行され、その結果が外側のクエリに渡されて利用されます。サブクエリは SELECT, FROM, WHERE 句など、様々な場所で使用できます。

### **非相関サブクエリ (Non-correlated Subquery)**

内側のサブクエリが外側のクエリから独立して実行され、一度だけ結果を生成します。その結果は、外側のクエリ全体で利用されます。

#### **例 1: WHERE 句での利用（単一値またはリストを返す場合）**

平均価格より高い商品のリストを取得。

**事前データ:**

products テーブル:

| product_id | product_name | price   | category_id |
| :--------- | :----------- | :------ | :---------- |
| 1          | Laptop       | 1200.00 | 1           |
| 2          | Mouse        | 25.50   | 1           |
| 3          | Keyboard     | 75.00   | 1           |
| 4          | Monitor      | 300.00  | NULL        |
| 5          | Webcam       | 50.00   | 2           |

**SQL:**

SELECT product_name, price  
FROM products  
WHERE price \> (SELECT AVG(price) FROM products); \-- サブクエリが先に平均価格を計算

（補足: 上記 products テーブルの AVG(price)は約 330.1 となります）

**実行結果:**

| product_name | price   |
| :----------- | :------ |
| Laptop       | 1200.00 |
| Monitor      | 300.00  |

このサブクエリは一度だけ実行され、products テーブルの平均価格（約 330.1）を返します。その後、外側のクエリが WHERE price \> 330.1 のように評価されます。

#### **例 2: FROM 句での利用（派生テーブルとして）**

サブクエリの結果を一時的なテーブル（**派生テーブル**または**導出テーブル**）として扱い、その上でさらにクエリを実行します。必ずエイリアスが必要です。

在庫が 100 以上の高額商品（1000 ドル以上）の平均価格を計算。

**事前データ:**

products テーブル:

| product_id | product_name | price   | stock_quantity | category_id |
| :--------- | :----------- | :------ | :------------- | :---------- |
| 1          | Laptop       | 1200.00 | 150            | 1           |
| 2          | Mouse        | 25.50   | 200            | 1           |
| 3          | Keyboard     | 75.00   | 80             | 1           |
| 4          | Monitor      | 300.00  | 120            | NULL        |
| 5          | Webcam       | 50.00   | 50             | 2           |

**SQL:**

SELECT AVG(high_price_product_price) AS avg_high_price_product_price  
FROM (  
 SELECT product_name, price AS high_price_product_price  
 FROM products  
 WHERE stock_quantity \>= 100 AND price \>= 1000  
) AS high_value_products; \-- サブクエリにエイリアスが必要

（補足: サブクエリの結果は Laptop 1200.00 のみ）

**実行結果:**

| avg_high_price_product_price |
| :--------------------------- |
| 1200.00                      |

このサブクエリは、まず条件に合う商品（この例では Laptop のみ）を抽出し、その結果に対して外側のクエリが平均値を計算します。

#### **例 3: SELECT 句での利用（スカラサブクエリ）**

単一の行と単一の列（**スカラ値**）を返すサブクエリです。

各商品について、その商品のカテゴリに属する全商品の平均価格を合わせて表示（カテゴリの平均価格）。

**事前データ:**

products テーブル:

| product_id | product_name | price   | category_id |
| :--------- | :----------- | :------ | :---------- |
| 1          | Laptop       | 1200.00 | 1           |
| 2          | Mouse        | 25.50   | 1           |
| 3          | Keyboard     | 75.00   | 1           |
| 4          | Monitor      | 300.00  | NULL        |
| 5          | Webcam       | 50.00   | 2           |

**SQL:**

SELECT  
 p.product_name,  
 p.price,  
 p.category_id,  
 (SELECT AVG(price) FROM products WHERE category_id \= p.category_id) AS avg_category_price  
FROM  
 products AS p;

（補足: category_id=1 の平均価格 \= (1200+25.5+75)/3 \= 433.5、category_id=2 の平均価格 \= 50/1 \= 50、category_id=NULL の平均価格 \= NULL）

**実行結果:**

| product_name | price   | category_id | avg_category_price |
| :----------- | :------ | :---------- | :----------------- |
| Laptop       | 1200.00 | 1           | 433.50             |
| Mouse        | 25.50   | 1           | 433.50             |
| Keyboard     | 75.00   | 1           | 433.50             |
| Monitor      | 300.00  | NULL        | NULL               |
| Webcam       | 50.00   | 2           | 50.00              |

この例では、SELECT 句のサブクエリは、外側のクエリの p.category_id に依存しており、行ごとに実行されます。これは次に説明する「相関サブクエリ」の性質を持っています。

### **相関サブクエリ (Correlated Subquery) (実務での重要度：中)**

内側のサブクエリが外側のクエリの現在の行の値に依存して実行されるサブクエリです。外側のクエリの行が処理されるたびに、内側のサブクエリが再評価されます。パフォーマンスに影響を与えることがあるため、代替手段（JOIN やウィンドウ関数）を検討することが推奨されます。

#### **💡 コラム: なぜ相関サブクエリはパフォーマンスに影響を与えるのか？**

相関サブクエリは、外側のクエリの**各行**が処理されるたびに**再実行**されます。もし外側のクエリが 1000 行あった場合、内側のサブクエリも 1000 回実行されることになります。これにより、特にデータ量が多い場合や、内側のサブクエリが複雑な場合に、処理時間が大幅に増加する可能性があります。

非相関サブクエリは一度だけ実行されるため、この点が大きな違いとなります。パフォーマンスが問題となる場合は、相関サブクエリを JOIN 句や**ウィンドウ関数**（後の章で学習）に書き換えることで改善できる場合があります。

#### **例: 各カテゴリで最も価格が高い商品を取得**

**事前データ:**

products テーブル:

| product_id | product_name | price   | category_id |
| :--------- | :----------- | :------ | :---------- |
| 1          | Laptop       | 1200.00 | 1           |
| 2          | Mouse        | 25.50   | 1           |
| 3          | Keyboard     | 75.00   | 1           |
| 4          | Monitor      | 300.00  | NULL        |
| 5          | Webcam       | 50.00   | 2           |
| 6          | Headset      | 150.00  | 2           |

**SQL:**

SELECT  
 p1.product_name,  
 p1.price,  
 p1.category_id  
FROM  
 products AS p1  
WHERE  
 p1.price \= (SELECT MAX(p2.price) FROM products AS p2 WHERE p2.category_id \= p1.category_id);

（補足: category_id=1 の最大価格は 1200.00、category_id=2 の最大価格は 150.00、category_id=NULL の最大価格は NULL）

**実行結果:**

| product_name | price   | category_id |
| :----------- | :------ | :---------- |
| Laptop       | 1200.00 | 1           |
| Headset      | 150.00  | 2           |

このクエリでは、内側のサブクエリ SELECT MAX(p2.price) FROM products AS p2 WHERE p2.category_id \= p1.category_id は、外側のクエリ（p1）の現在の category_id に依存します。外側のクエリが products テーブルの各行を処理するたびに、その商品の category_id を使ってサブクエリが実行され、そのカテゴリの最大価格を取得します。

### **EXISTS / NOT EXISTS (実務での重要度：高)**

サブクエリが何らかの行を返すかどうか（存在するかどうか）をチェックします。サブクエリの内部が実際にデータを返す必要はなく、**条件に一致する行が 1 つでも見つかれば TRUE を返し、そうでなければ FALSE を返します**。

IN 句がサブクエリの結果と具体的な値を比較するのに対し、EXISTS は単に「存在するか」をチェックするため、パフォーマンス面で優れている場合があります（特にサブクエリが大量の行を返す可能性がある場合）。

#### **基本構文**

\-- EXISTS  
SELECT 列リスト  
FROM テーブル 1  
WHERE EXISTS (サブクエリ);

\-- NOT EXISTS  
SELECT 列リスト  
FROM テーブル 1  
WHERE NOT EXISTS (サブクエリ);

#### **例: 少なくとも 1 つの注文がある顧客を取得**

**事前データ:**

customers テーブル:

| customer_id | customer_name |
| :---------- | :------------ |
| 1           | Alice         |
| 2           | Bob           |
| 3           | Charlie       |
| 4           | David         |

orders テーブル:

| order_id | customer_id | order_date   |
| :------- | :---------- | :----------- |
| 1        | 1           | '2023-01-01' |
| 2        | 1           | '2023-01-05' |
| 3        | 2           | '2023-02-10' |
| 4        | 4           | '2023-03-15' |

**SQL:**

SELECT customer_name  
FROM customers AS c  
WHERE EXISTS (  
 SELECT 1 \-- 実際に何を選択するかは重要ではない、行の存在が重要  
 FROM orders AS o  
 WHERE o.customer_id \= c.customer_id  
);

**実行結果:**

| customer_name |
| :------------ |
| Alice         |
| Bob           |
| David         |

このクエリは、orders テーブルに customers テーブルの現在の顧客 ID と一致する行が 1 つでも存在すれば、その顧客を結果に含めます。Charlie は注文がないため含まれません。

#### **例: 注文が一度もない顧客を取得**

事前データ:  
（EXISTS の例と同じデータを使用）  
**SQL:**

SELECT customer_name  
FROM customers AS c  
WHERE NOT EXISTS (  
 SELECT 1  
 FROM orders AS o  
 WHERE o.customer_id \= c.customer_id  
);

**実行結果:**

| customer_name |
| :------------ |
| Charlie       |

このクエリは、orders テーブルに customers テーブルの現在の顧客 ID と一致する行が一つも存在しない顧客を結果に含めます。Charlie のみが対象となります。

#### **IN と EXISTS の使い分け (実務での重要度：高)**

両者ともサブクエリを使って条件を絞り込む際に使われますが、内部的な動作と得意なケースが異なります。

- **IN**:
  - サブクエリが返す**値のリスト**に対して、外側のクエリの列が一致するかどうかを評価します。
  - サブクエリの結果が比較的小さいリストである場合に効率的です。
  - サブクエリが NULL を返すと、予期せぬ結果になる可能性があります（IN リストに NULL が含まれる場合、NULL と比較される行は UNKNOWN となり、結果に含まれないため）。
  - WHERE col IN (SELECT val FROM subquery)
- **EXISTS**:
  - サブクエリが**行を返すかどうか**（存在するどうか）のみを評価します。具体的な値を比較するわけではありません。
  - サブクエリが大量の行を返す可能性がある場合に、しばしば IN よりも効率的です。なぜなら、EXISTS は条件に一致する最初の 1 行を見つけた時点でサブクエリの実行を停止できるためです。
  - NULL 値の影響を受けにくいです。
  - WHERE EXISTS (SELECT 1 FROM subquery WHERE outer_col \= inner_col)

**使い分けの指針:**

- **サブクエリの結果が少数の値のリスト**で、かつ NULL 値の心配が少ない場合 → **IN**
- **サブクエリの結果が大量の行**になる可能性があり、または\*\*サブクエリ内で外部クエリの列を参照する（相関サブクエリになる）\*\*場合 → **EXISTS**

### **サブクエリの代替としての JOIN (実務での重要度：高)**

多くの相関サブクエリや、IN 句を使用する非相関サブクエリは、JOIN 句を使用することで書き換え可能です。一般的に、JOIN 句の方がパフォーマンスが優れていることが多いとされています（オプティマイザがより最適化しやすい構造のため）。

相関サブクエリの例を JOIN で書き換え:  
（各カテゴリで最も価格が高い商品を取得）  
\-- 相関サブクエリの元の例  
SELECT  
 p1.product_name,  
 p1.price,  
 p1.category_id  
FROM  
 products AS p1  
WHERE  
 p1.price \= (SELECT MAX(p2.price) FROM products AS p2 WHERE p2.category_id \= p1.category_id);  
\`\`\`sql  
\-- JOIN と GROUP BY で書き換え  
SELECT  
 p.product_name,  
 p.price,  
 p.category_id  
FROM  
 products AS p  
INNER JOIN (  
 SELECT category_id, MAX(price) AS max_price_in_category  
 FROM products  
 GROUP BY category_id  
) AS max_prices  
ON p.category_id \= max_prices.category_id AND p.price \= max_prices.max_price_in_category;

この JOIN を使用したクエリは、まず各カテゴリの最大価格をサブクエリで計算し、その結果を元のテーブルと結合して、最大価格を持つ商品に絞り込んでいます。多くの場合、このような書き換えの方が効率的です。

## **RDBMS 間の違いに注意！ (実務での重要度：中)**

ここまで見てきた JOIN、集合演算、サブクエリの構文や挙動は、多くの RDBMS で共通していますが、細かな部分で違いがあります。

| 機能/概念                      | PostgreSQL                                            | MySQL                                                 | Oracle                                                | SQL Server                                            |
| :----------------------------- | :---------------------------------------------------- | :---------------------------------------------------- | :---------------------------------------------------- | :---------------------------------------------------- |
| FULL OUTER JOIN                | サポートあり                                          | サポートなし（代替で対応）                            | サポートあり                                          | サポートあり                                          |
| EXCEPT / MINUS                 | EXCEPT                                                | サポートなし（代替で対応）                            | MINUS                                                 | EXCEPT                                                |
| LIMIT/OFFSET                   | LIMIT OFFSET                                          | LIMIT OFFSET                                          | ROWNUM / OFFSET FETCH                                 | TOP / OFFSET FETCH                                    |
| サブクエリの最適化             | 進んでいる                                            | 進んでいる                                            | 進んでいる                                            | 進んでいる                                            |
| 相関サブクエリのパフォーマンス | 状況によるが、JOIN やウィンドウ関数への書き換えを推奨 | 状況によるが、JOIN やウィンドウ関数への書き換えを推奨 | 状況によるが、JOIN やウィンドウ関数への書き換えを推奨 | 状況によるが、JOIN やウィンドウ関数への書き換えを推奨 |

特に FULL OUTER JOIN や EXCEPT/MINUS は、RDBMS によってサポート状況や名称が異なるため、使用する際には注意が必要です。

これで、SQL 基礎の「複数テーブル操作とサブクエリ」に関する講義資料は終わりです。

次に進む準備ができましたら、お知らせください。次のセクションでは「高度な SELECT」について学びます。
