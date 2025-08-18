# **SQL 基礎：データベースとテーブル**

## **導入：データベースと RDBMS**

この講義では、リレーショナルデータベース（RDB）の基本的な概念と、それらを操作するための SQL（Structured Query Language）の基礎を学びます。特に、PostgreSQL を例に具体的な操作を見ていきますが、ここで学ぶ知識は他の主要な RDBMS（MySQL, Oracle Database, SQL Server など）にも広く応用できます。

### **データベースとは？**

**データベース**とは、整理され、構造化された情報の集合体のことです。データを効率的に保存、管理、検索するために使用されます。

### **RDBMS（リレーショナルデータベース管理システム）とは？**

**RDBMS**は、リレーショナルデータベースを管理するためのソフトウェアです。データを**テーブル**という形式で管理し、テーブル間の関係性（リレーション）を定義できるのが特徴です。

### **テーブルとは？**

**テーブル**は、データベース内でデータを格納する基本的な構造です。Excel のスプレッドシートのように、行と列で構成されます。

- **列（カラム、フィールド）**: 特定の種類のデータを格納します（例: ユーザー名、商品の価格）。
- **行（レコード、タプル）**: 1 つのエンティティ（例: 1 人のユーザー、1 つの商品）に関するすべての情報を格納します。

## **データベースの作成：CREATE DATABASE**

新しいデータベースを作成する際に使用するコマンドです。

### **基本構文**

CREATE DATABASE データベース名;

### **例**

my_company_db という名前のデータベースを作成します。

CREATE DATABASE my_company_db;

### **考慮事項**

- **データベース名**: ユニークでなければなりません。
- **権限**: PostgreSQL では、CREATE DATABASE を実行するには**スーパーユーザー権限**または CREATEDB ロールが必要です。実務では権限管理が重要になるため、適切な権限を持つユーザーで実行しましょう。
- **文字エンコーディング**: 日本語を扱う場合は、UTF8 を指定することが多いです。（例: CREATE DATABASE my_db ENCODING 'UTF8';）
- **所有者（Owner）**: データベースの所有者を指定できます。（例: CREATE DATABASE my_db OWNER user_name;）
- **データベースとスキーマの関係**: RDBMS によっては、「データベース」と「スキーマ」の用語が異なる意味合いで使われることがあります。例えば、Oracle では「スキーマ」がユーザー（アカウント）に紐づく論理的な領域を指し、PostgreSQL では「データベース」の内部に複数の「スキーマ」を持つことができます。製品によって用語の定義や階層が異なるため、混同しないよう注意が必要です。

## **テーブルの作成：CREATE TABLE**

データベース内に新しいテーブルを作成するコマンドです。テーブル名、列名、データ型、および制約を定義します。

### **基本構文**

CREATE TABLE テーブル名 (  
 列名 1 データ型 \[制約\],  
 列名 2 データ型 \[制約\],  
 ...  
 \[テーブル制約\]  
);

### **命名規則の注意点**

テーブル名や列名には、以下の点に注意して命名しましょう。

- **半角英数字とアンダースコア（\_）**: これらを使用することが一般的です。
- **スネークケース**: user_id, product_name のように、単語間をアンダースコアで繋ぐ**スネークケース**が推奨されます。
- **予約語の回避**: SQL のキーワード（例: SELECT, FROM, WHERE など）はテーブル名や列名として使用できません。もしどうしても使用したい場合は、ダブルクォーテーションで囲む必要がありますが、これは非推奨です。
- **読みやすさ**: テーブルや列の役割がわかるような、意味のある名前をつけましょう。

### **データ型（Data Types）**

各列に格納できるデータの種類を定義します。RDBMS によって提供されるデータ型は多岐にわたりますが、ここではよく使われるものを紹介します。

| データ型 (PostgreSQL) | 説明                                                 | 具体例 (Example Value)    | 他の RDBMS での類似例 |
| :-------------------- | :--------------------------------------------------- | :------------------------ | :-------------------- |
| SMALLINT              | 小さな整数値（約$-32,768$～ 32,767）                 | 10, \-1000                | SMALLINT              |
| INTEGER               | 整数値（約$-20$億～ 20 億）                          | 123, \-500000             | INT, NUMBER           |
| BIGINT                | 大きな整数値（約$-9 \\times 10^{18}$～ 9times1018）  | 9876543210, \-10000000000 | BIGINT                |
| NUMERIC(p, s)         | **正確な**数値（p: 全体の桁数, s: 小数点以下の桁数） | 123.45, 999.00            | DECIMAL(p, s)         |
| REAL                  | **単精度**浮動小数点数（**近似値**）                 | 1.23, 0.0001              | FLOAT, FLOAT(24)      |
| DOUBLE PRECISION      | **倍精度**浮動小数点数（**近似値**）                 | 123.456789, 0.000000001   | DOUBLE, FLOAT(53)     |
| VARCHAR(n)            | 可変長文字列（最大 n 文字）                          | 'Hello', '商品 A'         | VARCHAR2(n), NVARCHAR |
| TEXT                  | 可変長文字列（長さ制限なし）                         | '長い文章や説明文'        | LONGTEXT              |
| BOOLEAN               | 真偽値（TRUE, FALSE, NULL）                          | TRUE, FALSE               | TINYINT(1), BIT       |
| DATE                  | 日付（年、月、日）                                   | '2023-10-26'              | DATE                  |
| TIMESTAMP             | 日付と時刻                                           | '2023-10-26 14:30:00'     | DATETIME              |

### **データ型選択のポイント**

- **TEXT 型**: PostgreSQL ではサイズ制限がありませんが、Oracle や SQL Server など他の RDBMS では TEXT 型に相当するデータ型がサイズ制限があったり、非推奨であったりする場合があります。汎用性を考慮するなら VARCHAR(n)を適切な長さで使うのが無難です。
- **BOOLEAN 型**: PostgreSQL や MySQL では BOOLEAN 型が直接サポートされますが、Oracle には BOOLEAN 型が存在しません。その代わりに CHAR(1)型と'T', 'F'などの値を組み合わせて真偽値を表現することが一般的です。
- **数値型 (NUMERIC / DECIMAL vs FLOAT / REAL / DOUBLE PRECISION) の選び方**:

  - **NUMERIC(p, s) または DECIMAL(p, s)（固定小数点数型）**:
    - **特徴**: **正確な**数値を格納します。p (precision) は全体の桁数、s (scale) は小数点以下の桁数を指定します。指定した精度で値を正確に保持するため、丸め誤差が発生しません。
    - **用途**: **金額、金融データ、科学計算で厳密な精度が求められるデータ**など、誤差が許されない場面で必ず使用してください。
    - **例**: NUMERIC(10, 2\) は小数点以下 2 桁、全体で最大 10 桁の数値を格納できます。12345678.90 のような金額に適しています。
  - **FLOAT、REAL、DOUBLE PRECISION（浮動小数点数型）**:

    - **特徴**: **近似値**を格納します。内部的には二進数（2 進数）で表現されるため、特に十進数（10 進数）の小数を扱う場合に、わずかな**丸め誤差**が生じることがあります。REAL は単精度、DOUBLE PRECISION（または FLOAT）は倍精度で、後者の方が精度は高いですが、それでも近似値です。
    - 丸め誤差が生じる理由:  
      コンピュータは基本的に 0 と 1 の二進数で数を表現します。10 進数で簡単に表せる分数（例: 0.1=1/10）でも、二進数で正確に表現できない場合があります。  
      例として、10 進数の 1/3 を考えてみましょう。これを 10 進数の小数で正確に表現しようとすると、$0.3333...$と無限に続きます。途中で打ち切ると「丸め誤差」が生じます。  
      これと同様に、10 進数の 0.1 や 0.2 も、二進数で正確に表現しようとすると無限小数になることがあります。コンピュータは有限のビット数で表現するため、どこかで「丸め」が行われ、それがわずかな誤差として現れるのです。
    - **用途**: 科学計算、物理シミュレーション、グラフィック処理など、わずかな誤差が許容される高速な計算が必要な場面で使われます。
    - **絶対に使ってはいけない場面**: **金額や、正確な計算結果が必須なデータ**（例えば、0.1 \+ 0.2 が厳密に 0.3 になることが保証されないため）。
    - 具体例で見る誤差:  
      以下の SQL を実行してみてください。  
      SELECT 0.1 \+ 0.2;

      期待する結果は 0.3 ですが、浮動小数点数型で計算されると 0.30000000000000004 のようなわずかに異なる値になることがあります。そのため、厳密な比較を行うと予期しない結果になる可能性があります。  
      SELECT 0.1 \+ 0.2 \= 0.3;

      このクエリは、期待に反して FALSE を返すことがあります。これが浮動小数点数の丸め誤差による影響です。

### **列制約（Column Constraints）**

列に特定の条件を課すことで、データの整合性を保ちます。

#### **NOT NULL**

その列に NULL（値がない状態）を許可しません。

CREATE TABLE users (  
 id INTEGER NOT NULL,  
 name VARCHAR(100) NOT NULL  
);

#### **PRIMARY KEY (主キー)**

テーブルの各行を一意に識別するための列または**列の組み合わせ**です。

- NOT NULL と UNIQUE の両方の特性を自動的に持ちます。
- テーブルごとに 1 つだけ設定できます。

CREATE TABLE products (  
 product_id INTEGER PRIMARY KEY, \-- product_id が主キー  
 product_name VARCHAR(255) NOT NULL  
);

#### **複合主キー (Composite Primary Key)**

主キーは、単一の列だけでなく、**複数の列の組み合わせ**で構成することも可能です。これを**複合主キー**と呼びます。複合主キーの場合、その**組み合わせた値がテーブル全体で一意かつ NULL でない**ことを保証します。テーブル全体の主キーは**論理的に 1 つ**とみなされます。

**例:** ある学生が複数の科目を受講している場合に、学生 ID と科目 ID の組み合わせで一意に成績を特定するテーブルを考えます。

CREATE TABLE grades (  
 student_id INTEGER,  
 course_id INTEGER,  
 grade VARCHAR(2),  
 PRIMARY KEY (student_id, course_id) \-- student_id と course_id の組み合わせが主キー  
);

この例では、

- student_id が 1 で course_id が 101 の組み合わせは 1 つしか存在できません。
- student_id が 1 で course_id が 102 の組み合わせは別の行として存在できます。  
  このように、個々の列は重複しても、組み合わせとして重複しないことで一意性を保証します。

#### **主キーの自動採番**

主キーには、新しい行が追加されるたびに自動的に一意の値を生成する機能を持たせることがよくあります。

- **PostgreSQL**: SERIAL や BIGSERIAL 型がよく使われます。最近では SQL 標準に準拠した GENERATED ALWAYS AS IDENTITY 句の使用が推奨されています。  
  \-- SERIAL 型の例  
  CREATE TABLE example_serial (  
   id SERIAL PRIMARY KEY,  
   data TEXT  
  );

  \-- GENERATED ALWAYS AS IDENTITY の例 (PostgreSQL 10 以降推奨)  
  CREATE TABLE example_identity (  
   id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,  
   data TEXT  
  );

- **MySQL**: AUTO_INCREMENT を使用します。  
  \-- MySQL の例  
  CREATE TABLE example_auto_increment (  
   id INT AUTO_INCREMENT PRIMARY KEY,  
   data VARCHAR(255)  
  );

- **Oracle**: SEQUENCE オブジェクトとトリガーを組み合わせて使用するか、IDENTITY 列を定義します。  
  \-- Oracle の例 (IDENTITY 列)  
  CREATE TABLE example_identity_oracle (  
   id NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,  
   data VARCHAR2(255)  
  );

このように、**自動採番の仕組みは RDBMS ごとに大きな違いがある**ため、使用する RDBMS のドキュメントを確認することが重要です。

#### **UNIQUE**

その列の全ての値が一意であることを保証します。NULL は複数存在できます。

CREATE TABLE employees (  
 employee_id INTEGER PRIMARY KEY,  
 email VARCHAR(255) UNIQUE \-- email は重複を許さない  
);

#### **PRIMARY KEY と UNIQUE の違い**

どちらも値の一意性を保証しますが、重要な違いがあります。

| 特徴                   | PRIMARY KEY                                                          | UNIQUE                                                             |
| :--------------------- | :------------------------------------------------------------------- | :----------------------------------------------------------------- |
| **NULL の許容**        | **不可** (NOT NULL を自動的に持つ)                                   | **可** (複数 NULL が存在可能)                                      |
| **テーブルあたりの数** | 1 つのみ (単一列または複合列として)                                  | 複数設定可能                                                       |
| **目的**               | 行を一意に識別するための「主たるキー」                               | 指定された列（または列の組み合わせ）の値が重複しないことを保証する |
| **インデックス**       | 自動的にクラスタードインデックスが作成されることが多い（RDBMS 依存） | 自動的に非クラスタードインデックスが作成されることが多い           |

#### **DEFAULT**

値を指定しなかった場合に、自動的に設定されるデフォルト値を定義します。

CREATE TABLE orders (  
 order_id INTEGER PRIMARY KEY,  
 order_date DATE DEFAULT CURRENT_DATE \-- デフォルトで現在の日付が設定される  
);

#### **CHECK**

その列に挿入される値が、指定された条件を満たしていることを強制します。

CREATE TABLE students (  
 student_id INTEGER PRIMARY KEY,  
 age INTEGER CHECK (age \>= 0 AND age \<= 150\) \-- 年齢は 0 から 150 の範囲  
);

#### **FOREIGN KEY (外部キー)**

他のテーブルの PRIMARY KEY または UNIQUE 列を参照する列です。テーブル間の関連性を定義し、参照整合性を維持します。

例: orders テーブルの customer_id が、customers テーブルの customer_id を参照するようにします。

CREATE TABLE customers (  
 customer_id INTEGER PRIMARY KEY,  
 customer_name VARCHAR(255) NOT NULL  
);

CREATE TABLE orders (  
 order_id INTEGER PRIMARY KEY,  
 customer_id INTEGER,  
 order_date DATE,  
 FOREIGN KEY (customer_id) REFERENCES customers (customer_id) \-- customers テーブルの customer_id を参照  
);

#### **外部キーの参照動作 (ON DELETE, ON UPDATE)**

外部キー制約には、参照先の親テーブルの行が削除されたり、主キーが更新されたりした場合に、子テーブルの行をどのように扱うかを定義するオプションがあります。これは実務で非常に重要です。

| オプション  | 説明                                                                                                                                    |
| :---------- | :-------------------------------------------------------------------------------------------------------------------------------------- |
| NO ACTION   | **デフォルト動作**。親テーブルの行が削除/更新されようとした場合、参照している子テーブルの行が存在すればエラーとなり、操作を拒否します。 |
| RESTRICT    | NO ACTION とほぼ同じです。操作を拒否します。                                                                                            |
| CASCADE     | 親テーブルの行が削除/更新された場合、関連する子テーブルの行も**自動的に削除/更新**されます。                                            |
| SET NULL    | 親テーブルの行が削除/更新された場合、子テーブルの外部キー列の値を NULL に設定します。                                                   |
| SET DEFAULT | 親テーブルの行が削除/更新された場合、子テーブルの外部キー列の値をデフォルト値に設定します。                                             |

**例:**

CREATE TABLE orders (  
 order_id INTEGER PRIMARY KEY,  
 customer_id INTEGER,  
 order_date DATE,  
 FOREIGN KEY (customer_id) REFERENCES customers (customer_id)  
 ON DELETE CASCADE \-- 親テーブル(customers)の顧客が削除されたら、その顧客の注文も削除する  
 ON UPDATE RESTRICT \-- 親テーブル(customers)の顧客 ID が更新されたら、参照している注文があれば更新を拒否する  
);  
\`\`\`NO ACTION\`（または\`RESTRICT\`）がデフォルトであり、親側の削除がエラーになることが多いことを理解しておくことが重要です。

\---

\#\# リテラルとデータ型の扱い

SQL では、データ型を直接指定するだけでなく、値そのものを直接記述することがよくあります。これを\*\*リテラル (Literal)\*\* と呼びます。また、SQL エンジンはリテラルのデータ型を自動的に判断したり、異なるデータ型を組み合わせて使用する際に自動で型変換を行ったりします。

\#\#\# リテラルとは？

\*\*リテラル\*\*は、SQL 文に直接記述される固定値のことです。プログラミング言語における「定数」のようなものです。

| リテラルの種類    | 例                               | 説明                                 |
| :---------------- | :------------------------------- | :----------------------------------- |
| 文字列リテラル    | \`'Hello World'\`, \`'123'\`     | シングルクォーテーションで囲みます。 |
| 数値リテラル      | \`123\`, \`3.14\`, \`-5\`        | 整数や小数をそのまま記述します。     |
| 論理値リテラル    | \`TRUE\`, \`FALSE\`              | 真偽値を記述します。                 |
| NULL リテラル     | \`NULL\`                         | 値がないことを表します。             |
| 日付/時刻リテラル | \`'2023-01-01'\`, \`'10:30:00'\` | シングルクォーテーションで囲みます。 |

\#\#\# NULL の重要性

\`NULL\`は「値がない」「未知」の状態を表し、\*\*空文字 (\`''\`) や数値の \`0\` とは異なります\*\*。これは実務で非常によく誤解される点ですので、明確に理解しておく必要があります。

\* \`NULL\` は値を持たないため、いかなる値とも等しくありません（\`NULL \= NULL\` も \`FALSE\` になります）。  
\* \`NULL\` のチェックには \`IS NULL\` または \`IS NOT NULL\` を使用します。

\#\#\# 型推論 (Type Inference)

SQL エンジンは、記述されたリテラルの形式から、そのリテラルがどのデータ型であるかを自動的に判断しようとします。

\*\*例:\*\*

\* \`SELECT 100;\` ⇒ \`INTEGER\`型と推論されます。  
\* \`SELECT 'Apple';\` ⇒ \`TEXT\`型または\`VARCHAR\`型と推論されます。  
\* \`SELECT 3.14;\` ⇒ \`NUMERIC\`型または\`DOUBLE PRECISION\`型（浮動小数点数）と推論されます。

\#\#\# 暗黙の型変換 (Implicit Type Conversion)

異なるデータ型の値を比較したり、演算したりする場合、SQL エンジンは互換性のあるデータ型に自動的に変換しようとします。これを\*\*暗黙の型変換\*\*と呼びます。

\*\*例:\*\*

\`\`\`sql  
SELECT 100 \+ '50'; \-- '50'が数値に変換され、150 になる（RDBMS によってはエラーになる場合もある）  
SELECT '2023-01-01' \> CURRENT_DATE; \-- 日付文字列が日付型に変換され、比較される

注意点:  
暗黙の型変換は便利ですが、予期せぬ結果やパフォーマンスの問題を引き起こす可能性があります。例えば、文字列と数値を比較する際に、意図しない型変換が行われて比較が正しく行われないことがあります。  
例（意図しない結果の可能性）:  
数値として保存されているカラム item_code があり、VARCHAR 型だとします。  
SELECT \* FROM items WHERE item_code \> '100';  
この場合、item_code が文字列として比較されるため、'2'が'100'より大きいと判断されるなど、数値としての比較と異なる結果になることがあります。

#### **RDBMS による暗黙変換の度合い**

暗黙の型変換の厳格さは、RDBMS によって大きく異なります。

- **PostgreSQL**: 比較的柔軟な型推論と暗黙の型変換を行います。
- Oracle や SQL Server: PostgreSQL と比較して暗黙の型変換が厳格で、意図しない型変換はエラーになることが多く、明示的な型変換がより頻繁に求められます。  
  実務では、利用する RDBMS の挙動を理解し、できる限り明示的な型変換を心がけることが重要です。

### **明示的な型変換 (Explicit Type Conversion)**

暗黙の型変換に頼るのではなく、**明示的にデータ型を変換する**ことを強く推奨します。これにより、意図しない挙動を防ぎ、SQL 文の可読性と安定性を高めることができます。

一般的に、CAST()関数や、RDBMS 固有の型変換演算子（PostgreSQL では::演算子）を使用します。

**例:**

SELECT 100 \+ CAST('50' AS INTEGER); \-- '50'を明示的に INTEGER に変換  
SELECT '123'::INTEGER \+ 456; \-- PostgreSQL 特有の記法で文字列を INTEGER に変換  
SELECT CAST('2023-01-01' AS DATE); \-- 文字列を DATE 型に変換

## **テーブル構造の変更と削除**

### **テーブルの削除：DROP TABLE**

既存のテーブルをデータベースから完全に削除します。

#### **基本構文**

DROP TABLE テーブル名;

#### **例**

users テーブルを削除します。

DROP TABLE users;

#### **重要なオプション**

- IF EXISTS: テーブルが存在しない場合でもエラーを出さずに実行します。スクリプトの実行時に便利です。  
  DROP TABLE IF EXISTS old_table;

- CASCADE: 削除しようとしているテーブルを参照している他のオブジェクト（例: 外部キー制約を持つテーブル）も一緒に削除します。**注意して使用してください！**  
  DROP TABLE customers CASCADE; \-- customers テーブルを削除し、それに関連する外部キーも削除される

#### **実務での注意点**

DROP TABLE はテーブルとデータを完全に削除するため、**本番環境で安易に実行することはほとんどありません**。データ削除が必要な場合は、代わりに以下の方法を検討します。

- **TRUNCATE TABLE**: テーブル構造は残し、全てのデータを高速に削除します。トランザクションログは少なく、ロールバックはできません。
- **DELETE FROM**: WHERE 句で条件を指定して一部のデータを削除したり、全データを削除したりできます。トランザクションログに記録され、ロールバック可能です。
- **論理削除**: テーブルからデータを物理的に削除するのではなく、削除フラグ（例: is_deleted BOOLEAN DEFAULT FALSE）を立てて、そのデータが「削除された」状態であることを示す方法です。データ復旧が容易ですが、クエリが複雑になることがあります。

### **テーブル構造の変更：ALTER TABLE**

既存のテーブルの構造を変更する際に使用します。

#### **列の追加：ADD COLUMN**

新しい列をテーブルに追加します。

ALTER TABLE テーブル名 ADD COLUMN 新しい列名 データ型 \[制約\];

例: products テーブルに price 列を追加します。

ALTER TABLE products ADD COLUMN price NUMERIC(10, 2\) DEFAULT 0.00;

#### **列の削除：DROP COLUMN**

既存の列をテーブルから削除します。

ALTER TABLE テーブル名 DROP COLUMN 列名;

例: employees テーブルから email 列を削除します。

ALTER TABLE employees DROP COLUMN email;

#### **DROP COLUMN の RDBMS 依存性**

ALTER TABLE DROP COLUMN は多くの RDBMS でサポートされていますが、Oracle の古いバージョンなど、一部の RDBMS やバージョンでは直接サポートされていなかったり、特定の制約があったりする場合があります。実務で実行する際は、使用している RDBMS のバージョンを確認しましょう。

#### **列のデータ型変更：ALTER COLUMN TYPE**

既存の列のデータ型を変更します。データの互換性に注意が必要です。

ALTER TABLE テーブル名 ALTER COLUMN 列名 TYPE 新しいデータ型;

例: products テーブルの product_name 列の長さを変更します。

ALTER TABLE products ALTER COLUMN product_name TYPE VARCHAR(500);

#### **制約の追加・削除**

制約の追加:  
ADD CONSTRAINT 句を使用します。  
ALTER TABLE customers ADD CONSTRAINT unique_email UNIQUE (email);

制約の削除:  
DROP CONSTRAINT 句を使用します。  
ALTER TABLE orders DROP CONSTRAINT orders_customer_id_fkey; \-- 外部キー制約の削除  
ALTER TABLE employees DROP CONSTRAINT employees_email_key; \-- UNIQUE 制約の削除

※制約名は RDBMS によって自動生成される場合があるため、確認が必要です。

これで、SQL 基礎の「データベースとテーブル」に関する講義資料は終わりです。

次に進む準備ができましたら、お知らせください。どのような内容の演習問題が必要か、具体的な要件があれば教えていただけますでしょうか？
