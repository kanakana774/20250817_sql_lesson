# **SQL インデックスと実行計画 研修問題**

この章では、データベースのパフォーマンス改善に不可欠な「インデックス」と「実行計画」について、実践的な問題を通じて理解を深めます。各問題は、インデックスの作成方法、その効果、インデックスが利用されないパターン、そして実行計画の読み解き方を学ぶことを目的としています。

### **テーブル定義 (DDL)**

この研修で使用するテーブル定義は以下の通りです。

\-- 顧客  
CREATE TABLE Customers (  
 customer_id INT PRIMARY KEY,  
 name VARCHAR(100),  
 email VARCHAR(100),  
 phone VARCHAR(20),  
 city VARCHAR(50),  
 join_date DATE,  
 membership_id INT  
);

\-- 会員ランク (Customers との参照関係用)  
CREATE TABLE Memberships (  
 membership_id INT PRIMARY KEY,  
 rank_name VARCHAR(50),  
 discount_rate DECIMAL(5,2)  
);

ALTER TABLE Customers  
ADD FOREIGN KEY (membership_id) REFERENCES Memberships(membership_id);

\-- 商品  
CREATE TABLE Products (  
 product_id INT PRIMARY KEY,  
 product_name VARCHAR(100),  
 category_id INT,  
 price DECIMAL(10,2)  
);

\-- 商品カテゴリ  
CREATE TABLE Categories (  
 category_id INT PRIMARY KEY,  
 category_name VARCHAR(50)  
);

\-- 注文  
CREATE TABLE Orders (  
 order_id INT PRIMARY KEY,  
 customer_id INT,  
 order_date DATE,  
 status VARCHAR(20),  
 FOREIGN KEY (customer_id) REFERENCES Customers(customer_id)  
);

\-- 注文明細  
CREATE TABLE OrderItems (  
 order_item_id INT PRIMARY KEY,  
 order_id INT,  
 product_id INT,  
 quantity INT,  
 FOREIGN KEY (order_id) REFERENCES Orders(order_id),  
 FOREIGN KEY (product_id) REFERENCES Products(product_id)  
);

\-- スタッフ  
CREATE TABLE Staff (  
 staff_id INT PRIMARY KEY,  
 name VARCHAR(100),  
 position VARCHAR(50),  
 hire_date DATE,  
 warehouse_id INT  
);

\-- 倉庫 (Staff との参照関係用)  
CREATE TABLE Warehouses (  
 warehouse_id INT PRIMARY KEY,  
 warehouse_name VARCHAR(50),  
 location VARCHAR(50)  
);

ALTER TABLE Staff  
ADD FOREIGN KEY (warehouse_id) REFERENCES Warehouses(warehouse_id);

### **サンプルデータ (DML)**

以下のサンプルデータを用いて問題に取り組んでください。実際のデータベースでは、より多くのデータでテストすることでパフォーマンスの違いが顕著になります。

\-- Memberships  
INSERT INTO Memberships (membership_id, rank_name, discount_rate) VALUES  
(1, 'Bronze', 0.05),  
(2, 'Silver', 0.10),  
(3, 'Gold', 0.15),  
(4, 'Platinum', 0.20);

\-- Customers  
INSERT INTO Customers (customer_id, name, email, phone, city, join_date, membership_id) VALUES  
(101, '佐藤 太郎', 'sato.t@example.com', '090-1111-2222', '東京', '2023-01-15', 1),  
(102, '鈴木 花子', 'suzuki.h@example.com', '080-3333-4444', '大阪', '2023-02-20', 2),  
(103, '田中 次郎', 'tanaka.j@example.com', '070-5555-6666', '東京', '2023-03-10', 1),  
(104, '高橋 美咲', 'takahashi.m@example.com', '090-7777-8888', '福岡', '2023-04-01', 3),  
(105, '渡辺 健太', 'watanabe.k@example.com', '080-9999-0000', '大阪', '2023-05-05', 2),  
(106, '小林 健太', 'kobayashi.k@example.com', '090-1234-5678', '札幌', '2023-06-10', 1),  
(107, '加藤 結衣', 'kato.y@example.com', '080-9876-5432', '東京', '2023-07-01', 3);

\-- Categories  
INSERT INTO Categories (category_id, category_name) VALUES  
(1, '家電'),  
(2, '書籍'),  
(3, '食品'),  
(4, '日用品');

\-- Products  
INSERT INTO Products (product_id, product_name, category_id, price) VALUES  
(1, 'ワイヤレスイヤホン', 1, 12800.00),  
(2, 'プログラミング入門', 2, 2980.00),  
(3, '有機野菜セット', 3, 3500.00),  
(4, '多機能シャープペン', 4, 800.00),  
(5, 'スマートウォッチ', 1, 25000.00),  
(6, 'SQL 実践ガイド', 2, 4500.00),  
(7, '高級チョコレート', 3, 1500.00),  
(8, 'アロマディフューザー', 4, 3200.00),  
(9, 'タブレット PC', 1, 55000.00),  
(10, 'データサイエンス入門', 2, 3800.00),  
(11, '国産牛肉', 3, 7000.00),  
(12, 'エコバッグ', 4, 1200.00);

\-- Orders  
INSERT INTO Orders (order_id, customer_id, order_date, status) VALUES  
(1001, 101, '2023-01-20', 'Completed'),  
(1002, 102, '2023-02-25', 'Completed'),  
(1003, 101, '2023-03-01', 'Completed'),  
(1004, 103, '2023-03-15', 'Pending'),  
(1005, 102, '2023-04-10', 'Completed'),  
(1006, 104, '2023-04-05', 'Completed'),  
(1007, 101, '2023-05-12', 'Shipped'),  
(1008, 105, '2023-05-20', 'Completed'),  
(1009, 103, '2023-06-01', 'Completed'),  
(1010, 102, '2023-06-15', 'Pending'),  
(1011, 107, '2023-07-05', 'Completed'),  
(1012, 107, '2023-07-20', 'Pending');

\-- OrderItems  
INSERT INTO OrderItems (order_item_id, order_id, product_id, quantity) VALUES  
(1, 1001, 1, 1),  
(2, 1001, 2, 1),  
(3, 1002, 3, 2),  
(4, 1002, 4, 3),  
(5, 1003, 1, 1),  
(6, 1003, 6, 1),  
(7, 1004, 5, 1),  
(8, 1005, 7, 1),  
(9, 1005, 8, 2),  
(10, 1006, 9, 1),  
(11, 1007, 10, 1),  
(12, 1007, 1, 1),  
(13, 1008, 11, 1),  
(14, 1009, 2, 1),  
(15, 1009, 12, 5),  
(16, 1010, 3, 1),  
(17, 1011, 5, 1),  
(18, 1012, 1, 1);

\-- Warehouses  
INSERT INTO Warehouses (warehouse_id, warehouse_name, location) VALUES  
(1, '東京倉庫', '東京都 江東区'),  
(2, '大阪倉庫', '大阪府 堺市'),  
(3, '福岡倉庫', '福岡県 福岡市');

\-- Staff  
INSERT INTO Staff (staff_id, name, position, hire_date, warehouse_id) VALUES  
(1, '山本 賢一', 'Manager', '2022-04-01', 1),  
(2, '中村 恵子', 'Clerk', '2022-07-10', 1),  
(3, '小林 大輔', 'Clerk', '2023-01-05', 1),  
(4, '加藤 玲奈', 'Manager', '2022-05-15', 2),  
(5, '吉田 拓也', 'Clerk', '2023-02-01', 2),  
(6, '佐々木 翼', 'Manager', '2022-06-01', 3),  
(7, '林 結衣', 'Clerk', '2023-03-15', 3);

### **問題 1: 単一列インデックスの効果確認**

- **重要度**: ★★★
- **問題目的**: 単一列インデックスの作成とその利用状況を EXPLAIN で確認し、インデックスがどのように検索を高速化するかを理解する。
- **問題**:
  1. まず、Customers テーブルの city 列にインデックスがない状態で、city \= '東京'の顧客を検索するクエリの実行計画を EXPLAIN で確認してください。どのようなスキャンが行われますか？
  2. 次に、Customers テーブルの city 列に単一列インデックス idx_customer_city を作成してください。
  3. インデックス作成後、再度 city \= '東京'の顧客を検索するクエリの実行計画を EXPLAIN で確認してください。スキャン方法にどのような変化が見られますか？
- **答え**:

  1. **インデックス作成前**:  
     EXPLAIN SELECT \* FROM Customers WHERE city \= '東京';

     実行計画の出力例（PostgreSQL の場合）：  
     Seq Scan on customers (cost=0.00..X.XX rows=Y width=Z) のように、シーケンシャルスキャン (Seq Scan) が行われるでしょう。これは、テーブル全体を読み込んで条件に合う行を探すことを意味します。

  2. **インデックス作成**:  
     CREATE INDEX idx_customer_city ON Customers (city);

  3. **インデックス作成後**:  
     EXPLAIN SELECT \* FROM Customers WHERE city \= '東京';

     実行計画の出力例（PostgreSQL の場合）：  
     Index Scan using idx_customer_city on customers (cost=0.00..X.XX rows=Y width=Z) のように、インデックススキャン (Index Scan) が行われるでしょう。

- **解説**:
  - **インデックスがない場合**: city 列にインデックスがないため、DBMS は Customers テーブルの全行を最初から最後まで読み込み（シーケンシャルスキャン）、city \= '東京'の条件に合致する行を探します。データ量が多いとこの処理は非常に遅くなります。
  - **インデックスがある場合**: city 列にインデックスを作成すると、DBMS は idx_customer_city インデックスの B-Tree を辿って「東京」という値が格納されている行の位置を素早く特定できます。その後、特定された位置にあるテーブルの行だけを読み込むため、検索が高速化されます。実行計画では「Index Scan」として表示されます。

### **問題 2: 複合インデックスの効果と左側プレフィックスの原則**

- **重要度**: ★★★★★
- **問題目的**: 複合インデックスの作成、その効果、そして複合インデックスが利用される際の「左側プレフィックスの原則」を EXPLAIN で確認する。
- **問題**:

  1. Products テーブルから category_id \= 1 かつ price \> 10000 の商品を検索するクエリの実行計画を EXPLAIN で確認してください（インデックスがない状態で）。
  2. Products テーブルに category_id と price の複合インデックス idx_product_category_price を作成してください（category_id が先頭、price が 2 番目）。  
     CREATE INDEX idx_product_category_price ON Products (category_id, price DESC);

  3. 以下の各クエリの実行計画を EXPLAIN で確認し、インデックスの利用状況を比較してください。
     - a) category_id \= 1 AND price \> 10000 の両方の条件で検索
     - b) category_id \= 1 のみで検索
     - c) price \> 10000 のみで検索

- **答え**:

  1. **インデックス作成前**:  
     EXPLAIN SELECT \* FROM Products WHERE category_id \= 1 AND price \> 10000;

     実行計画の出力例：Seq Scan on products ... となり、シーケンシャルスキャンが選択されるでしょう。

  2. **複合インデックス作成**:  
     CREATE INDEX idx_product_category_price ON Products (category_id, price DESC);

     （問題文に SQL が含まれているため、ここでは繰り返しなし）

  3. **複合インデックス作成後**:

     - a) category_id \= 1 AND price \> 10000  
       EXPLAIN SELECT \* FROM Products WHERE category_id \= 1 AND price \> 10000;

       実行計画の出力例：Index Scan using idx_product_category_price on products ... となり、複合インデックスが**両方の条件**で利用されるでしょう。

     - b) category_id \= 1 のみ  
       EXPLAIN SELECT \* FROM Products WHERE category_id \= 1;

       実行計画の出力例：Index Scan using idx_product_category_price on products ... となり、複合インデックスが\*\*category_id の条件で利用される\*\*でしょう。

     - c) price \> 10000 のみ  
       EXPLAIN SELECT \* FROM Products WHERE price \> 10000;

       実行計画の出力例：Seq Scan on products ... となり、**シーケンシャルスキャン**が選択されるでしょう。複合インデックスは**利用されません**。

- **解説**:
  - **複合インデックスの構造**: (category_id, price DESC)という複合インデックスは、まず category_id でソートされ、次に category_id が同じ場合は price の降順でソートされた B-Tree を構成します。
  - **左側プレフィックスの原則**:
    - a) category_id と price の両方で検索する場合、インデックスの先頭から category_id で絞り込み、その中で price でさらに絞り込むことができるため、インデックスが効果的に利用されます。
    - b) category_id のみで検索する場合も、インデックスの先頭キーである category_id で絞り込みができるため、インデックスが利用されます。これは、辞書で最初の文字が分かっているだけで探せるのと同じです。
    - c) price のみで検索する場合、インデックスの先頭キーが category_id であるため、price 単独の条件ではインデックスの B-Tree を効率的に辿ることができません。したがって、DBMS はシーケンシャルスキャンを選択します。これが「左側プレフィックスの原則」です。

### **問題 3: カバリングインデックスと Index Only Scan**

- **重要度**: ★★★★
- **問題目的**: カバリングインデックスの作成方法と、それによって「Index Only Scan」が利用され、テーブル本体へのアクセスが不要になるメリットを EXPLAIN で確認する。
- **問題**:

  1. Orders テーブルから customer_id \= 101 の注文の order_id と order_date を取得するクエリの実行計画を EXPLAIN で確認してください（インデックスがない状態、またはプライマリキー以外のインデックスがない状態）。
  2. Orders テーブルに、customer_id をキーとし、order_date と order_id をインデックスに含めるカバリングインデックス（PostgreSQL の INCLUDE 句を使用）idx_orders_customer_date_covering を作成してください。  
     CREATE INDEX idx_orders_customer_date_covering ON Orders (customer_id) INCLUDE (order_date, order_id);

  3. インデックス作成後、再度 customer_id \= 101 の注文の order_id と order_date を取得するクエリの実行計画を EXPLAIN で確認してください。スキャン方法にどのような変化が見られますか？

- **答え**:

  1. **インデックス作成前**:  
     EXPLAIN SELECT order_id, order_date FROM Orders WHERE customer_id \= 101;

     実行計画の出力例：Index Scan using orders_pkey on orders ... （customer_id が外部キーであれば、そのインデックスが使われる可能性、または Seq Scan）。もし customer_id にインデックスがなければ Seq Scan。いずれにせよ、Index Only Scan にはならないでしょう。

  2. **カバリングインデックス作成**:  
     CREATE INDEX idx_orders_customer_date_covering ON Orders (customer_id) INCLUDE (order_date, order_id);

     （問題文に SQL が含まれているため、ここでは繰り返しなし）

  3. **カバリングインデックス作成後**:  
     EXPLAIN SELECT order_id, order_date FROM Orders WHERE customer_id \= 101;

     実行計画の出力例（PostgreSQL の場合）：Index Only Scan using idx_orders_customer_date_covering on orders ... となり、**Index Only Scan**が利用されるでしょう。

- **解説**:
  - **Index Only Scan の利点**: 通常のインデックススキャンは、B-Tree を辿って行の物理的な位置（ポインタ）を見つけ、そのポインタを使ってテーブル本体から必要なデータを読み込みます（これを「ブックマークルックアップ」と呼びます）。しかし、カバリングインデックスの場合、クエリが要求する全ての列（この場合は order_id と order_date）がインデックス自体に含まれているため、テーブル本体へのアクセスが不要になります。
  - **I/O の削減**: テーブル本体へのアクセスはディスク I/O を伴うため、これが不要になることで I/O コストが大幅に削減され、クエリの実行速度が劇的に向上します。
  - PostgreSQL の INCLUDE 句は、キーではないがインデックスに含めたい列を指定するのに使われ、Index Only Scan を実現する強力な機能です。

### **問題 4: インデックスが効かないパターン (関数適用)**

- **重要度**: ★★★★★
- **問題目的**: WHERE 句でインデックス対象の列に関数を適用した場合にインデックスが利用されないことを EXPLAIN で確認し、その対策として関数インデックスの有効性を理解する。
- **問題**:

  1. Products テーブルの product_name 列を全て小文字に変換して'smartwatch'と一致する商品を検索するクエリの実行計画を EXPLAIN で確認してください。product_name 列に単一列インデックス idx_product_name を作成済みの場合でも、どのようなスキャンが行われますか？  
     （idx_product_name インデックスが存在しない場合は、まず CREATE INDEX idx_product_name ON Products (product_name);を実行してください）
  2. 次に、product_name 列に LOWER()関数を適用した関数インデックス idx_product_name_lower を作成してください。  
     CREATE INDEX idx_product_name_lower ON Products (LOWER(product_name));

  3. 関数インデックス作成後、再度小文字変換して'smartwatch'と一致する商品を検索するクエリの実行計画を EXPLAIN で確認してください。スキャン方法にどのような変化が見られますか？

- **答え**:

  1. **関数適用前のインデックス利用状況**:  
     \-- まず idx_product_name が存在することを確認または作成  
     \-- CREATE INDEX idx_product_name ON Products (product_name);  
     EXPLAIN SELECT \* FROM Products WHERE LOWER(product_name) \= 'smartwatch';

     実行計画の出力例：Seq Scan on products ... となり、**シーケンシャルスキャン**が選択されるでしょう。既存の idx_product_name インデックスは利用されません。

  2. **関数インデックス作成**:  
     CREATE INDEX idx_product_name_lower ON Products (LOWER(product_name));

     （問題文に SQL が含まれているため、ここでは繰り返しなし）

  3. **関数インデックス作成後**:  
     EXPLAIN SELECT \* FROM Products WHERE LOWER(product_name) \= 'smartwatch';

     実行計画の出力例：Index Scan using idx_product_name_lower on products ... となり、**インデックススキャン**が利用されるでしょう。

- **解説**:
  - **関数適用時のインデックス不使用**: 通常の B-Tree インデックスは、元の列の値に基づいて構築されています。LOWER(product_name)のように WHERE 句で列に関数を適用すると、DBMS はインデックスに格納されている元の値ではなく、関数適用後の値と検索条件を比較しようとします。しかし、B-Tree は元の値のソート順でしか効率的に検索できないため、インデックスは利用されず、シーケンシャルスキャンにフォールバックします。
  - **関数インデックスによる対策**: 関数インデックスは、列に関数を適用した結果の値をキーとして構築されるインデックスです。このインデックスを作成することで、WHERE LOWER(product_name) \= 'smartwatch'のようなクエリでも、関数適用後の値に基づいた B-Tree を効率的に利用し、インデックススキャンが可能になります。

### **問題 5: JOIN 方式の確認 (Nested Loop Join)**

- **重要度**: ★★★★
- **問題目的**: DBMS がテーブル結合時に選択する主要な JOIN アルゴリズムの一つである Nested Loop Join を EXPLAIN ANALYZE で確認し、その特性と適用条件を理解する。
- **問題**:

  1. Customers テーブルと Orders テーブルを customer_id で結合し、東京在住の顧客の注文情報を取得するクエリの実行計画を EXPLAIN ANALYZE で確認してください。
     - 前提: Customers.city にはインデックス idx_customer_city が、Orders.customer_id にはプライマリキーまたは外部キーインデックスが既に存在すると仮定します。

  SELECT  
   c.name,  
   o.order_id,  
   o.order_date  
   FROM  
   Customers c  
   JOIN  
   Orders o ON c.customer_id \= o.customer_id  
   WHERE  
   c.city \= '東京'  
   ORDER BY  
   c.name, o.order_date;

  2. このクエリで Nested Loop Join が選択される可能性が高い理由について考察してください。

- **答え**:

  1. **クエリの実行計画**:  
     EXPLAIN ANALYZE SELECT  
      c.name,  
      o.order_id,  
      o.order_date  
     FROM  
      Customers c  
     JOIN  
      Orders o ON c.customer_id \= o.customer_id  
     WHERE  
      c.city \= '東京'  
     ORDER BY  
      c.name, o.order_date;

     実行計画の出力例（PostgreSQL の場合）：Nested Loop Join が選択される可能性が高いでしょう。  
     Nested Loop (cost=X.XX..Y.YY rows=Z width=W) (actual time=A.AA..B.BB rows=C loops=1)  
      \-\> Index Scan using idx_customer_city on customers c (cost=... actual time=...)  
      Filter: (c.city \= '東京')  
      \-\> Index Scan using orders_customer_id_fkey on orders o (cost=... actual time=...) \-- (または orders_pkey)  
      Index Cond: (o.customer_id \= c.customer_id)

     （Customers.city にインデックス idx_customer_city がある場合は、まず Customers テーブルをインデックススキャンし、その結果の各 customer_id で Orders テーブルのインデックスを効率的に引くため、Nested Loop が最適と判断されます。）

- **解説**:
  - インデックスがない場合の JOIN (Hash Join/Merge Join):  
    Orders.customer_id にインデックスがない場合、DBMS は効率的なインデックス検索を利用できないため、Hash Join や Merge Join のような別のアルゴリズムを検討します。
    - **Hash Join**: 両方のテーブル（特に大きい方）にインデックスがなくても高速に結合できるため、大規模なテーブル結合でよく選ばれます。Customers テーブルをフィルタリングした結果（「ビルド側」）をメモリ上のハッシュテーブルに構築し、Orders テーブル（「プローブ側」）の各行をそのハッシュテーブルで検索します。
    - **Merge Join**: 両方のテーブルを結合キーでソートする必要があるため、ソートコストがかかります。しかし、既にソートされている場合や、ソートコストがペイするほど大規模なデータの場合に選択されます。
  - インデックスがある場合の JOIN (Nested Loop Join):  
    Orders.customer_id にインデックスを作成すると、DBMS は Nested Loop Join を選択する可能性が高まります。
    - **Nested Loop Join**: 外部テーブル（この場合、Customers テーブルを city='東京'で絞り込んだ結果）の各行について、内部テーブル（Orders テーブル）の結合列に存在するインデックスを使って効率的に対応する行を検索します。これは、外部テーブルの行数が少なく、内部テーブルの結合列に効率的なインデックスがある場合に非常に優れたパフォーマンスを発揮します。
  - この問題を通じて、インデックスの有無が DBMS が選択する JOIN 方式に大きな影響を与え、それがクエリのパフォーマンスに直結することを理解できます。EXPLAIN ANALYZE を使うことで、実際にどの JOIN 方式が選択され、どれくらいのコストがかかっているかを確認することが重要です。

### **実務で役立つパフォーマンスチューニングのワークフロー**

遅いクエリに出会った際に、どのようにパフォーマンスを改善していくかの一般的なワークフローをまとめます。

1. **遅いクエリの特定**:
   - アプリケーションのログやデータベースの監視ツールなどから、実行時間の長いクエリを特定します。
2. **EXPLAIN ANALYZE で実行計画の確認**:
   - 特定したクエリに対して EXPLAIN ANALYZE を実行し、実際の実行計画と統計情報を取得します。
   - この際、EXPLAIN で推定コストだけを見るのではなく、**必ず ANALYZE を使って実際の時間や行数を確認する**ことが重要です。
3. **実行計画の分析**:
   - **テーブルスキャンが頻繁に発生していないか**: 特に、少数の行を絞り込みたい場面で Seq Scan が出ている場合は、インデックスの検討が必要です。
   - **コストの高いノード（ボトルネック）を特定**: actual time が長くかかっているノードや、rows の推定値と実測値が大きく乖離しているノードに注目します。
   - **インデックスの利用状況**: 意図したインデックスが使われているか、Index Only Scan が利用されているかを確認します。
   - **JOIN 方式の確認**: 大規模なテーブル結合で非効率な Nested Loop が選ばれていないかなどを確認します。
4. **改善策の検討と実施**:
   - **インデックスの追加・修正**:
     - 検索条件（WHERE 句）やソート条件（ORDER BY 句）に使われている列に、適切な単一列インデックスや複合インデックスを作成します。
     - SELECT 句に含まれる列が多い場合は、カバリングインデックス（INCLUDE 句）を検討します。
     - 特定の条件で絞り込まれる行が多い場合は、部分インデックスを検討します。
     - LIKE '%...'のようにインデックスが効かないパターンになっていないか確認し、可能であればクエリを書き換えるか、関数インデックスや演算子クラスの指定を検討します。
   - **クエリの書き換え**:
     - サブクエリをより効率的な JOIN や**ウィンドウ関数**に書き換えることを検討します。
     - 不要な JOIN や列の取得がないか確認します。
     - OR 条件を UNION で書き換えるなど、より DBMS が最適化しやすい形に変更します。
   - **統計情報の更新**:
     - テーブルのデータが大幅に増減した場合、ANALYZE コマンド（PostgreSQL の場合）で統計情報を更新すると、DBMS がより正確な実行計画を立てられるようになります。
5. **効果の検証**:
   - 改善策を実施した後、再度 EXPLAIN ANALYZE を実行し、パフォーマンスが実際に向上したか、実行計画が改善されたかを確認します。

この繰り返しによって、データベースのパフォーマンスは持続的に向上していきます。

この研修問題が、インデックスと実行計画に関する理解を深め、実務でのデータベースパフォーマンスチューニングに役立つことを願っています。
