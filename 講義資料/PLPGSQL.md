# **SQL アドバンスド概念：トランザクション、ストアドプロシージャ、PL/pgSQL**

この講義では、データベースの整合性を保ち、複雑なビジネスロジックを効率的に実装するための重要な 3 つの概念、**トランザクション**、**ストアドプロシージャ**、そして**PL/pgSQL**について解説します。

## **セクション 1: PL/pgSQL の基本構文**

PL/pgSQL は、SQL に手続き型言語の要素（変数、制御構造、エラーハンドリングなど）を追加した言語です。

### **基本ブロック構造**

すべての PL/pgSQL コードは、以下のブロック構造で構成されます。

\[ \<\<ブロック名\>\> \]  
\[ DECLARE  
 宣言文 \]  
BEGIN  
 実行文  
END \[ ブロック名 \];

- DECLARE: 変数やカーソルを宣言するブロックです。必須ではありません。
- BEGIN ... END: 実際の処理を書く部分です。

### **変数の宣言と代入**

変数は DECLARE セクションで宣言します。

DECLARE  
 \-- 単純な変数宣言  
 v_customer_count INTEGER := 0; \-- デフォルト値を設定

    \-- テーブルのカラムの型を参照する
    v\_customer\_id customers.customer\_id%TYPE;

    \-- テーブルの行全体の型を参照する
    rec\_customer customers%ROWTYPE;

- **:=**: 代入演算子です。
- **%TYPE**: カラムのデータ型を動的に参照します。基となるテーブルの型が変更されても、コードの修正が不要になるため、メンテナンス性が向上します。
- **%ROWTYPE**: テーブルの行全体の構造（すべてのカラムとデータ型）を参照します。

### **条件分岐 (IF-THEN-ELSIF-ELSE)**

IF 文を使って条件に応じて処理を分岐させます。

IF condition THEN  
 statements  
ELSIF another_condition THEN  
 statements  
ELSE  
 statements  
END IF;

**使用例:**

IF v_amount \> 1000 THEN  
 v_status := 'High Value';  
ELSIF v_amount \> 500 THEN  
 v_status := 'Medium Value';  
ELSE  
 v_status := 'Low Value';  
END IF;

### **ループ処理 (LOOP, WHILE, FOR)**

**1\. LOOP 文**

無限ループを定義し、EXIT 文で脱出します。

LOOP  
 \-- 何らかの処理  
 EXIT WHEN condition; \-- 条件を満たしたらループを抜ける  
END LOOP;

**2\. WHILE 文**

条件が真である間、処理を繰り返します。

WHILE condition LOOP  
 \-- 繰り返し処理  
END LOOP;

**3\. FOR 文（推奨）**

- **数値の反復**: 決まった回数だけ繰り返します。  
  FOR i IN 1..10 LOOP  
   RAISE NOTICE '繰り返し回数: %', i;  
  END LOOP;

- **クエリ結果の反復**: **実務で最もよく使われるループ形式**です。SELECT 文の結果を一行ずつ処理します。  
  FOR rec IN SELECT product_name, price FROM products WHERE price \> 500 LOOP  
   RAISE NOTICE '高額商品: % (価格: %)', rec.product_name, rec.price;  
  END LOOP;

### **エラーハンドリングのパターン**

BEGIN ... EXCEPTION ... END ブロックでエラーを捕捉し、処理することができます。

- **構文**  
  BEGIN  
   \-- 処理  
  EXCEPTION  
   WHEN 例外名 THEN  
   \-- 例外処理  
   WHEN others THEN  
   \-- その他の例外処理  
  END;

- **個別の例外を捕捉する**:
  - unique_violation: ユニーク制約違反
  - foreign_key_violation: 外部キー制約違反
  - division_by_zero: 0 による除算

BEGIN  
 INSERT INTO customers (customer_id, customer_name) VALUES (101, 'Test User');

EXCEPTION  
 WHEN unique_violation THEN  
 RAISE NOTICE 'この顧客 ID はすでに存在します。';

    WHEN others THEN
        RAISE NOTICE '予期せぬエラーが発生しました: %', SQLERRM;

END;

## **セクション 2: ストアドプロシージャとストアドファンクション**

### **PostgreSQL におけるストアドプロシージャとストアドファンクション**

PostgreSQL では、\*\*ストアドファンクション (FUNCTION)\*\*が古くから存在し、\*\*ストアドプロシージャ (PROCEDURE)\*\*は PostgreSQL 11 で導入されました。この歴史的背景から、両者の使い分けが重要になります。

| 特徴                     | ストアドプロシージャ (PROCEDURE)                                                                     | ストアドファンクション (FUNCTION)                                              |
| :----------------------- | :--------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------- |
| **主な用途**             | **トランザクション制御**を伴う手続き的処理。\<br\>例: 複数テーブルへの一連の更新、複雑なバッチ処理。 | **値を返す**ことが目的の処理。\<br\>例: 計算、データ加工、ビューのように利用。 |
| **戻り値**               | 戻り値を持たない、または OUT パラメータで値を返す。                                                  | 必ず 1 つの値を返す（RETURNS TABLE で表を返すことも可能）。                    |
| **トランザクション制御** | 内部で COMMIT や ROLLBACK を実行可能。                                                               | できない（呼び出し元のトランザクションの一部として実行される）。               |
| **呼び出し方法**         | CALL procedure_name(...); のみ。                                                                     | SELECT function_name(...)や PERFORM function_name(...)で呼び出し。             |

**使い分けの指針**

- **プロシージャ**：データベース内で一連の不可分な処理（INSERT/UPDATE/DELETE を含む）を実行し、その途中でトランザクションを確定させたい場合に選びます。
- **関数**：何らかの計算結果や加工されたデータを単一の結果として返したい場合に選びます。SELECT 文の中で利用できるため、ビューのように活用できます。

### **実践例と正しい呼び出し方**

customer_id を受け取り、その顧客の合計注文金額を計算して返すストアドファンクションと、それを使ってトランザクションを制御するストアドプロシージャの例です。

**1\. 合計金額を計算するストアドファンクション**

\-- PostgreSQL の例: 合計金額を計算する関数  
CREATE OR REPLACE FUNCTION calculate_customer_total_fn(  
 p_customer_id INT  
)  
RETURNS DECIMAL  
LANGUAGE plpgsql  
AS $$  
DECLARE  
 v_total_amount DECIMAL(10,2); \-- 合計金額を格納する変数を宣言  
BEGIN  
 \-- 顧客の注文明細から合計金額を計算し、変数に代入  
 SELECT SUM(oi.quantity \* p.price)  
 INTO v_total_amount  
 FROM Orders o  
 JOIN OrderItems oi ON o.order_id \= oi.order_id  
 JOIN Products p ON oi.product_id \= p.product_id  
 WHERE o.customer_id \= p_customer_id;

    \-- 計算結果を関数の戻り値として返す
    RETURN v\_total\_amount;

END;

$$
;

\-- 関数の実行方法 (SELECT文で呼び出す)
SELECT calculate\_customer\_total\_fn(102);

**2\. トランザクションを制御するストアドプロシージャ**

\-- PostgreSQL の例: 注文を確定するプロシージャ
CREATE OR REPLACE PROCEDURE confirm\_order(
    p\_order\_id INT
)
LANGUAGE plpgsql
AS
$$

BEGIN  
 \-- 注文ステータスの更新  
 UPDATE Orders SET status \= 'Confirmed' WHERE order_id \= p_order_id;

    \-- 何らかのチェック
    IF NOT FOUND THEN
        RAISE EXCEPTION '注文ID % は見つかりませんでした。', p\_order\_id;
    END IF;

    \-- 全ての処理が成功すればコミット
    COMMIT;

END;

$$
;

\-- プロシージャの実行方法 (CALL文で呼び出す)
CALL confirm\_order(1007);

## **セクション 3: トランザクションの深い理解**

### **ACID特性の再訪**

トランザクションは、以下の4つの重要な特性（**ACID特性**）によって、信頼性の高いデータベース操作を保証します。

1. **Atomicity (原子性)**
   * 「すべてか、無か（All or Nothing）」の原則です。
   * トランザクション内の複数の操作は、ひとまとまりの不可分な作業単位として扱われます。
   * 例: 銀行の口座振替では、UPDATE文が成功した後にシステムがクラッシュしても、ROLLBACKによって最初の状態に戻されるか、両方のUPDATEが成功（COMMIT）します。途中の状態は決して残りません。
2. **Consistency (一貫性)**
   * トランザクションが成功すれば、データベースは常に一貫性のある状態を保ちます。
   * データベースの制約（NOT NULL、FOREIGN KEYなど）やルールを壊すような操作は許可されません。
   * 例: 口座振替のトランザクションが完了した後、Aさんの残高 \+ Bさんの残高の合計値は、トランザクション開始前と変わらないことを保証します。
3. **Isolation (独立性)**
   * 複数のトランザクションが同時に実行されても、互いに干渉することはありません。
   * それぞれのトランザクションは、まるで他のトランザクションが存在しないかのように振る舞います。
   * この特性がなければ、同時実行による不整合なデータの読み取り（ダーティリードなど）が発生します。この問題への対策は、後述の「トランザクション分離レベル」で詳しく解説します。
4. **Durability (永続性)**
   * トランザクションが一度コミットされると、その変更は永続的にデータベースに保存されます。
   * 変更内容は、システム故障、電源断、データベース再起動などの障害が発生しても失われることはありません。

### **トランザクション分離レベルとPostgreSQLの挙動**

データベースが複数のトランザクションをどのように隔離するかを定義する設定です。分離レベルが低いほど同時実行性が高まりますが、不整合なデータ読み取りのリスクが高まります。

以下に、トランザクションの同時実行によって発生する3つの典型的な問題と、それを防ぐ分離レベル、そしてPostgreSQLの実際の挙動を解説します。

| 分離レベル | ダーティリード | ノンリピータブルリード | ファントムリード | PostgreSQLの注意点 |
| :---- | :---- | :---- | :---- | :---- |
| **READ UNCOMMITTED** | **発生しない** | 発生する | 発生する | 内部的にはREAD COMMITTEDとして動作するため、ダーティリードは発生しない。 |
| **READ COMMITTED** | 発生しない | 発生する | 発生する | PostgreSQLのデフォルト設定。 |
| **REPEATABLE READ** | 発生しない | 発生しない | **発生しない** | 標準SQLの定義より厳格で、スナップショット隔離（Snapshot Isolation）を採用しているためファントムリードも防ぐ。 |
| **SERIALIZABLE** | 発生しない | 発生しない | 発生しない | 厳密な直列実行を保証する最も高い分離レベル。同時実行性は最も低い。 |

### **実務的なトランザクション制御**

* BEGIN or START TRANSACTION：トランザクションを開始します。
* COMMIT：トランザクション内のすべての変更を確定し、永続化します。
* ROLLBACK：トランザクション内のすべての変更を元に戻します。
* **SAVEPOINTとROLLBACK TO**:
  * SAVEPOINT savepoint\_name;：トランザクションの途中に一時的な保存ポイントを設定します。
  * ROLLBACK TO savepoint\_name;：トランザクション全体ではなく、特定の保存ポイントまで部分的にロールバックすることができます。これは、大きなトランザクションで部分的なエラーが発生した場合に便利です。
* **デッドロックの回避**:
  * デッドロックは、複数のトランザクションが互いに相手がロックしているリソースを待っている状態です。
  * **典型的な原因**: ロックを取得する順序がトランザクション間で異なること。
  * **回避策**: 複数のリソースを更新する際には、常に**同じ順序**でロックを取得するようにルールを定めます。例: ordersテーブル、order\_itemsテーブルの順にロックを取得するなど。
* **長時間トランザクションのリスク**:
  * 長時間にわたるトランザクションは、データベースのパフォーマンスに悪影響を与えます。
  * **MVCCの非効率化**: PostgreSQLでは、古いデータ（タプル）の削除はVACUUMによって行われますが、長時間トランザクションが存在すると、そのトランザクションが参照している古いタプルは削除できません。これにより、ディスク容量の無駄や性能低下につながります。
  * **ロックの保持**: トランザクションが終了するまで、取得したロックが解放されません。これにより、他のトランザクションが待機させられ、システム全体の並行性が低下します。

## **セクション 4: トリガーと実務で役立つベストプラクティス**

### **トリガーの利用と注意点**

トリガーはデータの整合性維持に強力ですが、濫用するとデバッグやパフォーマンスが難しくなります。

* **BEFOREトリガー vs AFTERトリガー**
  * **BEFORE**: **書き込み前**に行を加工したい時に使います。NEW変数を変更し、NEWまたはNULL（削除）を返す必要があります。
  * **AFTER**: データベースに**変更が確定した後**の「副作用的な処理」に使います。監査ログの記録や、関連するテーブルの更新など。OLDやNEW変数は参照できますが、戻り値は無視されます。
* **トリガーの利用に向く場面**
  * **監査ログ**: 変更履歴を自動的に別テーブルに記録する。
  * **データ整合性の強制**: 複雑なチェック制約や、関連テーブルの自動更新を行う。
  * **BEFORE UPDATE トリガーの例**:

CREATE OR REPLACE FUNCTION set\_updated\_at()
RETURNS TRIGGER AS
$$

BEGIN  
 NEW.updated_at \= NOW(); \-- 新しい行の updated_at カラムを現在時刻に設定  
 RETURN NEW;  
END;

$$
LANGUAGE plpgsql;

CREATE TRIGGER trg\_set\_updated\_at BEFORE UPDATE ON products
FOR EACH ROW EXECUTE FUNCTION set\_updated\_at();

### **パフォーマンスチューニング**

* **不要なCOMMIT/ROLLBACKの乱用を避ける**: プロシージャやファンクション内部で頻繁にトランザクションを確定すると、オーバーヘッドが増え、パフォーマンスが低下します。可能な限り、呼び出し元のアプリケーション側でトランザクションを制御し、データベース内では最小限に留めます。
* **ループよりもセットベース処理を優先**: FOR...INループは簡潔ですが、大量のデータ処理には向いていません。セットベースのUPDATE ... FROM ...やINSERT ... SELECT ...のような単一のSQL文は、データベースが並列処理やオプティマイザの最適化を適用しやすいため、圧倒的に高速です。

### **運用・保守の注意点**

* **ブラックボックス化を防ぐ**:
  * ストアドプロシージャやトリガーは、一度デプロイされるとアプリケーションからは見えにくくなり、「ブラックボックス化」しやすいという欠点があります。
  * 開発者や運用者が後から理解できるように、**詳細なコメント**をコード内に残すこと、および**監査ログ**を組み込むことが不可欠です。
* **デプロイ時の依存関係を意識する**:
  * FUNCTIONやPROCEDUREが参照しているテーブルやカラムの名前が変更されると、それらが壊れる可能性があります。
  * %TYPEや%ROWTYPEを使用することで、この問題をある程度回避できますが、デプロイ時には必ず依存関係を確認し、テストを行う必要があります。

これらの内容を習得することで、データベースのパフォーマンスと堅牢性を両立させたアプリケーション開発が可能になります。
$$
