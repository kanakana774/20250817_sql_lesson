SQL Window 関数研修問題

テーブル定義 (DDL)

```SQL
-- 顧客
CREATE TABLE Customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    phone VARCHAR(20),
    city VARCHAR(50),
    join_date DATE,
    membership_id INT -- 後でFOREIGN KEYが追加されます
);

-- 会員ランク
CREATE TABLE Memberships (
    membership_id INT PRIMARY KEY,
    rank_name VARCHAR(50),
    discount_rate DECIMAL(5,2)
);

-- CustomersテーブルにFOREIGN KEYを追加
ALTER TABLE Customers
ADD FOREIGN KEY (membership_id) REFERENCES Memberships(membership_id);

-- 商品
CREATE TABLE Products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    category_id INT,
    price DECIMAL(10,2)
);

-- 商品カテゴリ
CREATE TABLE Categories (
    category_id INT PRIMARY KEY,
    category_name VARCHAR(50)
);

-- 在庫
CREATE TABLE Inventory (
    product_id INT,
    warehouse_id INT,
    stock_quantity INT,
    PRIMARY KEY (product_id, warehouse_id),
    FOREIGN KEY (product_id) REFERENCES Products(product_id)
);

-- 倉庫
CREATE TABLE Warehouses (
    warehouse_id INT PRIMARY KEY,
    warehouse_name VARCHAR(50),
    location VARCHAR(50)
);

-- 注文
CREATE TABLE Orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    status VARCHAR(20),
    FOREIGN KEY (customer_id) REFERENCES Customers(customer_id)
);

-- 注文明細
CREATE TABLE OrderItems (
    order_item_id INT PRIMARY KEY,
    order_id INT,
    product_id INT,
    quantity INT,
    FOREIGN KEY (order_id) REFERENCES Orders(order_id),
    FOREIGN KEY (product_id) REFERENCES Products(product_id)
);

-- 支払い
CREATE TABLE Payments (
    payment_id INT PRIMARY KEY,
    order_id INT,
    payment_date DATE,
    amount DECIMAL(10,2),
    method VARCHAR(20),
    FOREIGN KEY (order_id) REFERENCES Orders(order_id)
);

-- 配送
CREATE TABLE Shipments (
    shipment_id INT PRIMARY KEY,
    order_id INT,
    shipped_date DATE,
    delivery_date DATE,
    status VARCHAR(20),
    FOREIGN KEY (order_id) REFERENCES Orders(order_id)
);

-- 商品レビュー
CREATE TABLE Reviews (
    review_id INT PRIMARY KEY,
    product_id INT,
    customer_id INT,
    rating INT,
    comment TEXT,
    review_date DATE,
    FOREIGN KEY (product_id) REFERENCES Products(product_id),
    FOREIGN KEY (customer_id) REFERENCES Customers(customer_id)
);

-- スタッフ
CREATE TABLE Staff (
    staff_id INT PRIMARY KEY,
    name VARCHAR(100),
    position VARCHAR(50),
    hire_date DATE,
    warehouse_id INT,
    FOREIGN KEY (warehouse_id) REFERENCES Warehouses(warehouse_id)
);
```

サンプルデータ (DML)

```SQL
-- Memberships
INSERT INTO Memberships (membership_id, rank_name, discount_rate) VALUES
(1, 'Bronze', 0.05),
(2, 'Silver', 0.10),
(3, 'Gold', 0.15),
(4, 'Platinum', 0.20);

-- Customers
INSERT INTO Customers (customer_id, name, email, phone, city, join_date, membership_id) VALUES
(101, '佐藤 太郎', 'sato.t@example.com', '090-1111-2222', '東京', '2023-01-15', 1),
(102, '鈴木 花子', 'suzuki.h@example.com', '080-3333-4444', '大阪', '2023-02-20', 2),
(103, '田中 次郎', 'tanaka.j@example.com', '070-5555-6666', '東京', '2023-03-10', 1),
(104, '高橋 美咲', 'takahashi.m@example.com', '090-7777-8888', '福岡', '2023-04-01', 3),
(105, '渡辺 健太', 'watanabe.k@example.com', '080-9999-0000', '大阪', '2023-05-05', 2);

-- Categories
INSERT INTO Categories (category_id, category_name) VALUES
(1, '家電'),
(2, '書籍'),
(3, '食品'),
(4, '日用品');

-- Products
INSERT INTO Products (product_id, product_name, category_id, price) VALUES
(1, 'ワイヤレスイヤホン', 1, 12800.00),
(2, 'プログラミング入門', 2, 2980.00),
(3, '有機野菜セット', 3, 3500.00),
(4, '多機能シャープペン', 4, 800.00),
(5, 'スマートウォッチ', 1, 25000.00),
(6, 'SQL実践ガイド', 2, 4500.00),
(7, '高級チョコレート', 3, 1500.00),
(8, 'アロマディフューザー', 4, 3200.00),
(9, 'タブレットPC', 1, 55000.00),
(10, 'データサイエンス入門', 2, 3800.00),
(11, '国産牛肉', 3, 7000.00),
(12, 'エコバッグ', 4, 1200.00);

-- Warehouses
INSERT INTO Warehouses (warehouse_id, warehouse_name, location) VALUES
(1, '東京倉庫', '東京都 江東区'),
(2, '大阪倉庫', '大阪府 堺市'),
(3, '福岡倉庫', '福岡県 福岡市');

-- Inventory
INSERT INTO Inventory (product_id, warehouse_id, stock_quantity) VALUES
(1, 1, 50), (1, 2, 30),
(2, 1, 100), (2, 3, 20),
(3, 2, 40),
(4, 1, 200), (4, 3, 150),
(5, 1, 20),
(6, 1, 70), (6, 2, 50),
(7, 2, 80),
(8, 1, 120),
(9, 3, 15),
(10, 1, 60),
(11, 2, 25),
(12, 1, 180), (12, 3, 100);

-- Orders
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
(1010, 102, '2023-06-15', 'Pending');

-- OrderItems
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
(16, 1010, 3, 1);

-- Payments
INSERT INTO Payments (payment_id, order_id, payment_date, amount, method) VALUES
(1, 1001, '2023-01-20', 15780.00, 'Credit Card'),
(2, 1002, '2023-02-25', 9400.00, 'Bank Transfer'),
(3, 1003, '2023-03-01', 17300.00, 'Credit Card'),
(4, 1004, '2023-03-15', 25000.00, 'Credit Card'),
(5, 1005, '2023-04-10', 7900.00, 'E-wallet'),
(6, 1006, '2023-04-05', 55000.00, 'Credit Card'),
(7, 1007, '2023-05-12', 16600.00, 'Credit Card'),
(8, 1008, '2023-05-20', 7000.00, 'Bank Transfer'),
(9, 1009, '2023-06-01', 8980.00, 'Credit Card'),
(10, 1010, '2023-06-15', 3500.00, 'E-wallet');

-- Shipments
INSERT INTO Shipments (shipment_id, order_id, shipped_date, delivery_date, status) VALUES
(1, 1001, '2023-01-21', '2023-01-23', 'Delivered'),
(2, 1002, '2023-02-26', '2023-03-01', 'Delivered'),
(3, 1003, '2023-03-02', '2023-03-04', 'Delivered'),
(4, 1004, NULL, NULL, 'Pending'),
(5, 1005, '2023-04-11', '2023-04-13', 'Delivered'),
(6, 1006, '2023-04-06', '2023-04-09', 'Delivered'),
(7, 1007, '2023-05-13', NULL, 'In Transit'),
(8, 1008, '2023-05-21', '2023-05-23', 'Delivered'),
(9, 1009, '2023-06-02', '2023-06-04', 'Delivered'),
(10, 1010, NULL, NULL, 'Pending');

-- Reviews
INSERT INTO Reviews (review_id, product_id, customer_id, rating, comment, review_date) VALUES
(1, 1, 101, 5, '音質が素晴らしいです！', '2023-01-25'),
(2, 2, 101, 4, '初心者にも分かりやすい内容でした。', '2023-02-01'),
(3, 3, 102, 5, '新鮮で美味しい野菜でした。', '2023-03-05'),
(4, 6, 101, 5, 'SQLの理解が深まりました。', '2023-03-10'),
(5, 5, 103, 3, '機能は良いがバッテリー持ちがもう少し。', '2023-03-20'),
(6, 9, 104, 5, '高性能で満足しています。', '2023-04-10'),
(7, 1, 102, 4, 'デザインも良く満足です。', '2023-04-15');

-- Staff
INSERT INTO Staff (staff_id, name, position, hire_date, warehouse_id) VALUES
(1, '山本 賢一', 'Manager', '2022-04-01', 1),
(2, '中村 恵子', 'Clerk', '2022-07-10', 1),
(3, '小林 大輔', 'Clerk', '2023-01-05', 1),
(4, '加藤 玲奈', 'Manager', '2022-05-15', 2),
(5, '吉田 拓也', 'Clerk', '2023-02-01', 2),
(6, '佐々木 翼', 'Manager', '2022-06-01', 3),
(7, '林 結衣', 'Clerk', '2023-03-15', 3);

```

## 問題 1: 各顧客の最初の注文日と最新の注文日

### 重要度: ★★★

### 問題目的: FIRST_VALUE と LAST_VALUE を使用した、特定グループ内での最初と最後の値の取得方法を理解する。

### 問題: 各顧客について、最初の注文日と最新の注文日を特定し、それぞれの注文と共に表示してください。

### 答え:

```SQL
SELECT
    o.customer_id,
    o.order_id,
    o.order_date,
    FIRST_VALUE(o.order_date) OVER (PARTITION BY o.customer_id ORDER BY o.order_date ASC) AS first_order_date,
    LAST_VALUE(o.order_date) OVER (PARTITION BY o.customer_id ORDER BY o.order_date ASC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS latest_order_date
FROM
    Orders o
ORDER BY
    o.customer_id, o.order_date;
```

バリエーション: MIN と MAX の集計関数を OVER 句と組み合わせて使用することもできますが、FIRST_VALUE と LAST_VALUE はより明示的にウィンドウ関数であることを示します。

```SQL
SELECT
    o.customer_id,
    o.order_id,
    o.order_date,
    MIN(o.order_date) OVER (PARTITION BY o.customer_id) AS first_order_date_min_max,
    MAX(o.order_date) OVER (PARTITION BY o.customer_id) AS latest_order_date_min_max
FROM
    Orders o
ORDER BY
    o.customer_id, o.order_date;
```

### 解説:

- PARTITION BY o.customer_id: 結果セットを customer_id ごとに分割し、各顧客内で独立したウィンドウを形成します。
- ORDER BY o.order_date ASC: 各顧客のウィンドウ内で、order_date を昇順に並べ替えます。これは FIRST_VALUE と LAST_VALUE がどの行を「最初」または「最後」と見なすかを決定するために重要です。
- FIRST_VALUE(o.order_date): ウィンドウフレーム（デフォルトでは ORDER BY 句で指定された順序で最初の行）における order_date の値を返します。
- LAST_VALUE(o.order_date) OVER (...) ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING: LAST_VALUE はデフォルトのウィンドウフレームでは CURRENT ROW までしか見ないため、ウィンドウ全体を参照するように ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING（現在の行から見て、前のすべての行から後のすべての行まで）と明示的にフレームを指定する必要があります。これにより、各顧客のウィンドウ全体における最新の order_date が取得されます。
- MIN(o.order_date) OVER (PARTITION BY o.customer_id): MIN や MAX をウィンドウ関数として使用する場合、ORDER BY 句は必須ではありません。PARTITION BY で定義された各グループの最小値または最大値を返します。これは FIRST_VALUE や LAST_VALUE と異なる挙動を示すため、注意が必要です。

## 問題 2: 各顧客の注文ごとの累計注文額

### 重要度: ★★★★

### 問題目的: SUM 関数と ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW を使用した、累計計算の基本を理解する。

### 問題: 各顧客について、注文日順に累計注文額（OrderItems テーブルの quantity \* Products.price の合計）を計算してください。

### 答え:

```SQL
SELECT
    o.customer_id,
    o.order_id,
    o.order_date,
    SUM(oi.quantity * p.price) AS order_total,
    SUM(SUM(oi.quantity * p.price)) OVER (PARTITION BY o.customer_id ORDER BY o.order_date ASC, o.order_id ASC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cumulative_order_total
FROM
    Orders o
JOIN
    OrderItems oi ON o.order_id = oi.order_id
JOIN
    Products p ON oi.product_id = p.product_id
GROUP BY
    o.customer_id, o.order_id, o.order_date
ORDER BY
    o.customer_id, o.order_date, o.order_id;
```

バリエーション: UNBOUNDED PRECEDING のみを使用した場合（ROWS BETWEEN 句なし）も同様の結果になりますが、ROWS BETWEEN を明示することで挙動をより明確にできます。

```SQL
SELECT
    o.customer_id,
    o.order_id,
    o.order_date,
    SUM(oi.quantity * p.price) AS order_total,
    SUM(SUM(oi.quantity * p.price)) OVER (PARTITION BY o.customer_id ORDER BY o.order_date ASC, o.order_id ASC) AS cumulative_order_total_implicit_frame -- ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW が暗黙的に適用される
FROM
    Orders o
JOIN
    OrderItems oi ON o.order_id = oi.order_id
JOIN
    Products p ON oi.product_id = p.product_id
GROUP BY
    o.customer_id, o.order_id, o.order_date
ORDER BY
    o.customer_id, o.order_date, o.order_id;
```

### 解説:

- サブクエリまたは GROUP BY 句での集計: まず、各注文の合計金額を SUM(oi.quantity \* p.price)で計算し、customer_id, order_id, order_date でグループ化します。
- PARTITION BY o.customer_id: 累計計算を顧客ごとに独立して行います。
- ORDER BY o.order_date ASC, o.order_id ASC: 各顧客のウィンドウ内で、注文を日付順（同じ日付の場合は order_id 順）に並べ替えます。この順序で累計が計算されます。
- ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW: ウィンドウフレームを定義します。これは、「現在の行から見て、その前のすべての行から現在の行まで」を対象とするという意味です。これにより、各行でその時点までの累計値が計算されます。ORDER BY 句のみを指定した場合、このフレームは暗黙的に適用されることが一般的ですが、明示することでコードの意図がより明確になります。

## 問題 3: 各商品カテゴリ内の商品価格ランキング

### 重要度: ★★★★★

### 問題目的: ROW_NUMBER, RANK, DENSE_RANK の違いと使用方法を理解する。特にランキング作成における必須知識。

### 問題: 各商品カテゴリ内で、価格の高い順に商品のランキングを付けてください。

同価格の商品には異なる順位を割り当てる（連続した番号）。

同価格の商品には同じ順位を割り当て、次の順位は同価格の商品の数だけスキップする。

同価格の商品には同じ順位を割り当て、次の順位はスキップしない（連続した順位）。

### 答え:

```SQL
SELECT
    p.product_name,
    c.category_name,
    p.price,
    ROW_NUMBER() OVER (PARTITION BY c.category_id ORDER BY p.price DESC) AS row_num,
    RANK() OVER (PARTITION BY c.category_id ORDER BY p.price DESC) AS rank_num,
    DENSE_RANK() OVER (PARTITION BY c.category_id ORDER BY p.price DESC) AS dense_rank_num
FROM
    Products p
JOIN
    Categories c ON p.category_id = c.category_id
ORDER BY
    c.category_name, p.price DESC;
```

### バリエーション: 特定のカテゴリの上位 N 件を取得する場合など、WHERE 句やサブクエリでランキング結果を絞り込むことができます。例えば、各カテゴリで価格の高い上位 2 つの商品を取得する場合。

```SQL
WITH RankedProducts AS (
    SELECT
        p.product_name,
        c.category_name,
        p.price,
        RANK() OVER (PARTITION BY c.category_id ORDER BY p.price DESC) AS rank_num
    FROM
        Products p
    JOIN
        Categories c ON p.category_id = c.category_id
)
SELECT
    product_name,
    category_name,
    price,
    rank_num
FROM
    RankedProducts
WHERE
    rank_num <= 2
ORDER BY
    category_name, price DESC;
```

### 解説:

- PARTITION BY c.category_id: カテゴリごとに独立したランキングを作成します。これにより、各カテゴリ内で商品の価格を比較し、順位を付けます。
- ORDER BY p.price DESC: 各カテゴリ内で、price が高い順に並べ替えます。
- ROW_NUMBER(): 各ウィンドウ内で、順序付けされた行に連続した一意の番号を割り当てます。同順位の行があったとしても、異なる番号が振られます。
- RANK(): 各ウィンドウ内で、順序付けされた行に順位を割り当てます。同順位の行には同じ番号が振られ、次の順位は同順位の行の数だけスキップされます。例えば、1 位が 2 つあった場合、次は 3 位になります。
- DENSE_RANK(): 各ウィンドウ内で、順序付けされた行に順位を割り当てます。同順位の行には同じ番号が振られますが、次の順位はスキップされず、連続した番号になります。例えば、1 位が 2 つあった場合、次は 2 位になります。

これらの関数は、トップ N 分析や、グループ内での相対的な位置を特定する際によく使用されます。

## 問題 4: 各倉庫のスタッフの入社日を基準としたグループ内移動平均給与（架空）

### 重要度: ★★★★

### 問題目的: AVG 関数と ROWS BETWEEN を使用した移動平均の計算方法を理解する。

### 問題: 各倉庫について、スタッフの入社日順に、現在行と過去 2 行を含んだ 3 名分のスタッフの給与（架空の給与を例として staff_id \* 1000 として計算）の移動平均を計算してください。

### 答え:

```SQL
SELECT
    s.staff_id,
    s.name,
    w.warehouse_name,
    s.hire_date,
    (s.staff_id * 1000) AS hypothetical_salary, -- 架空の給与
    AVG(s.staff_id * 1000) OVER (PARTITION BY w.warehouse_id ORDER BY s.hire_date ASC ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg_salary
FROM
    Staff s
JOIN
    Warehouses w ON s.warehouse_id = w.warehouse_id
ORDER BY
    w.warehouse_name, s.hire_date;
```

### バリエーション: RANGE BETWEEN を使用すると、値の範囲でウィンドウを定義できます。例えば、RANGE BETWEEN INTERVAL '30' DAY PRECEDING AND CURRENT ROW のように指定することで、過去 30 日間の平均を計算できます。（RDBMS によって日付計算の構文が異なるため、ここでは例として示します。）

```SQL
-- PostgreSQLの場合の例 (RDBMSによって構文が異なります)
SELECT
    s.staff_id,
    s.name,
    w.warehouse_name,
    s.hire_date,
    (s.staff_id * 1000) AS hypothetical_salary,
    AVG(s.staff_id * 1000) OVER (
        PARTITION BY w.warehouse_id
        ORDER BY s.hire_date ASC
        RANGE BETWEEN INTERVAL '60 day' PRECEDING AND CURRENT ROW -- 現在の日付から過去60日間の平均 (仮の例)
    ) AS moving_avg_salary_range
FROM
    Staff s
JOIN
    Warehouses w ON s.warehouse_id = w.warehouse_id
ORDER BY
    w.warehouse_name, s.hire_date;
```

### 解説:

- PARTITION BY w.warehouse_id: 各倉庫内で独立した移動平均を計算します。
- ORDER BY s.hire_date ASC: 各倉庫のウィンドウ内で、スタッフを hire_date の昇順に並べ替えます。
- ROWS BETWEEN 2 PRECEDING AND CURRENT ROW: ウィンドウフレームを定義します。これは、「現在の行から見て、その前の 2 行と現在の行」の計 3 行を対象とすることを意味します。この 3 行の hypothetical_salary の平均が AVG()関数によって計算されます。
- ROWS と RANGE の違い:
- ROWS: 行数に基づいてウィンドウフレームを定義します。物理的な行の数を数えます。
- RANGE: 値の範囲に基づいてウィンドウフレームを定義します。ORDER BY 句で指定された列の値の範囲に基づきます。例えば、日付型の列であれば「過去 30 日以内」のように指定できます。

### 問題 5: 各注文の前の注文からの経過日数

### 重要度: ★★★★

### 問題目的: LAG 関数を使用した、前行のデータを参照する方法を理解する。時系列データ分析で頻出。

### 問題: 各顧客について、注文日順に、それぞれの注文がその顧客の前の注文から何日後にあったかを計算してください。最初の注文には NULL を返してください。

### 答え:

```SQL
SELECT
    o.customer_id,
    o.order_id,
    o.order_date,
    LAG(o.order_date, 1, NULL) OVER (PARTITION BY o.customer_id ORDER BY o.order_date ASC) AS previous_order_date,
    JULIANDAY(o.order_date) - JULIANDAY(LAG(o.order_date, 1, NULL) OVER (PARTITION BY o.customer_id ORDER BY o.order_date ASC)) AS days_since_previous_order -- SQLiteの場合
    -- DATEDIFF(o.order_date, LAG(o.order_date, 1, NULL) OVER (PARTITION BY o.customer_id ORDER BY o.order_date ASC)) AS days_since_previous_order -- SQL Serverの場合
    -- o.order_date - LAG(o.order_date, 1, NULL) OVER (PARTITION BY o.customer_id ORDER BY o.order_date ASC) AS days_since_previous_order -- PostgreSQLの場合
FROM
    Orders o
ORDER BY
    o.customer_id, o.order_date;
```

### バリエーション: LEAD 関数を使用すると、次行のデータを参照できます。例えば、次の注文までの日数を計算するのに使えます。また、日数の計算方法は RDBMS によって異なるため、使用する RDBMS に合わせて調整が必要です。以下は次の注文までの日数を計算する例です。

```SQL
SELECT
    o.customer_id,
    o.order_id,
    o.order_date,
    LEAD(o.order_date, 1, NULL) OVER (PARTITION BY o.customer_id ORDER BY o.order_date ASC) AS next_order_date,
    JULIANDAY(LEAD(o.order_date, 1, NULL) OVER (PARTITION BY o.customer_id ORDER BY o.order_date ASC)) - JULIANDAY(o.order_date) AS days_until_next_order -- SQLiteの場合
    -- DATEDIFF(LEAD(o.order_date, 1, NULL) OVER (PARTITION BY o.customer_id ORDER BY o.order_date ASC), o.order_date) AS days_until_next_order -- SQL Serverの場合
    -- LEAD(o.order_date, 1, NULL) OVER (PARTITION BY o.customer_id ORDER BY o.order_date ASC) - o.order_date AS days_until_next_order -- PostgreSQLの場合
FROM
    Orders o
ORDER BY
    o.customer_id, o.order_date;
```

### 解説:

- LAG(column, offset, default): 現在の行から指定された offset（オフセット）だけ前の行の column の値を返します。offset のデフォルトは 1 です。default は、前の行が存在しない場合に返す値（通常は NULL）です。
- PARTITION BY o.customer_id: 顧客ごとに独立して前の注文を検索します。
- ORDER BY o.order_date ASC: 各顧客の注文を日付順に並べ替えます。LAG 関数はこの順序に基づいて前の行を特定します。
- LEAD(column, offset, default): LAG とは逆に、現在の行から指定された offset だけ後の行の column の値を返します。
- 日付の差分計算: JULIANDAY() (SQLite), DATEDIFF() (SQL Server), 単純な日付の減算 (PostgreSQL) など、RDBMS によって日付の差分を計算する方法が異なります。実務では使用する RDBMS のドキュメントを確認してください。

### 問題 6: 各注文が全顧客の平均注文額と比較してどれくらい乖離しているか

### 重要度: ★★★★

### 問題目的: ウィンドウ関数と集約関数の組み合わせ、および全体平均との比較を行う方法を理解する。

### 問題: 各注文について、その注文の合計金額（OrderItems.quantity \* Products.price の合計）と、全注文の平均合計金額との差分を計算してください。

### 答え:

```SQL
WITH OrderTotals AS (
    SELECT
        o.order_id,
        SUM(oi.quantity * p.price) AS order_total
    FROM
        Orders o
    JOIN
        OrderItems oi ON o.order_id = oi.order_id
    JOIN
        Products p ON oi.product_id = p.product_id
    GROUP BY
        o.order_id
)
SELECT
    ot.order_id,
    ot.order_total,
    AVG(ot.order_total) OVER () AS overall_avg_order_total,
    ot.order_total - AVG(ot.order_total) OVER () AS difference_from_avg
FROM
    OrderTotals ot
ORDER BY
    ot.order_id;
```

### バリエーション: overall_avg_order_total は、サブクエリで事前に計算して結合することも可能ですが、OVER ()を使用することで、より簡潔に記述できます。

```SQL
WITH OrderTotals AS (
    SELECT
        o.order_id,
        SUM(oi.quantity * p.price) AS order_total
    FROM
        Orders o
    JOIN
        OrderItems oi ON o.order_id = oi.order_id
    JOIN
        Products p ON oi.product_id = p.product_id
    GROUP BY
        o.order_id
),
OverallAverage AS (
    SELECT
        AVG(order_total) AS avg_total
    FROM
        OrderTotals
)
SELECT
    ot.order_id,
    ot.order_total,
    oa.avg_total AS overall_avg_order_total,
    ot.order_total - oa.avg_total AS difference_from_avg
FROM
    OrderTotals ot, OverallAverage oa
ORDER BY
    ot.order_id;
```

### 解説:

- WITH OrderTotals AS (...): まず、CTE (Common Table Expression) を使って、各注文の合計金額を前処理で計算しておきます。これにより、メインクエリが読みやすくなります。
- AVG(ot.order_total) OVER (): ここがこの問題の重要なポイントです。OVER ()のように PARTITION BY 句を省略すると、結果セット全体を一つのウィンドウと見なします。つまり、OrderTotals CTE から返されるすべての注文の合計金額の平均が計算され、それが各行に適用されます。これにより、各注文の合計金額を全体の平均と比較できます。
- サブクエリでの計算との比較: OVER ()を使わずに、別のサブクエリで全体の平均を計算し、それを結合することも可能です。しかし、OVER ()を使用する方がコードがより簡潔になり、特に同じウィンドウ関数を複数回使用する場合に効率的です。どちらを使うかは、可読性やパフォーマンス要件によって判断すると良いでしょう。
