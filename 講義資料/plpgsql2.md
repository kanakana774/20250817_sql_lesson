# **PL/pgSQL 実践講義：現場で役立つ使い方**

## **1\. PL/pgSQL の基本**

PL/pgSQL（Procedural Language/PostgreSQL）は、PostgreSQL で複雑な処理を行うための手続き型言語です。SQL だけでは難しい、条件分岐や繰り返し処理をデータベースサーバー上で実行できます。これにより、ネットワーク通信のオーバーヘッドを減らし、パフォーマンスを向上させることができます。

### **文法の構成要素**

PL/pgSQL コードは、主に以下の 3 つのセクションから構成されます。

1. **DECLARE**: 変数、カーソル、およびローカルな関数やプロシージャを宣言するセクションです。
2. **BEGIN**: 実際のビジネスロジックや処理を記述するセクションです。
3. **END**: ブロックの終わりを示します。

これらのブロック全体は、DO $$ ... $$; という匿名ブロックの形で実行するか、ストアドファンクションやストアドプロシージャとして定義して利用します。

### **変数とデータ型**

PL/pgSQL では、一時的に値を格納するための変数を宣言して使用します。

変数の宣言構文:  
変数名 データ型 \[:= 初期値\];  
:= は代入演算子です。宣言時に初期値を設定しない場合は、デフォルトで NULL が代入されます。

**例：基本的な変数宣言**

DO $$  
DECLARE  
 \-- ユーザー ID を格納する整数型の変数  
 user_id INT := 123;  
 \-- ユーザー名を格納するテキスト型の変数  
 user_name TEXT := '山田 太郎';  
 \-- アカウント作成日を格納する日付型の変数  
 created_at DATE := '2023-01-15';  
BEGIN  
 \-- 変数を利用してメッセージを出力  
 RAISE NOTICE 'ユーザー ID: %, 名前: %, 作成日: %', user_id, user_name, created_at;  
END $$;

**実践的なヒント:**

- **%TYPE**: テーブルの列と同じデータ型を変数に割り当てることができます。これにより、テーブルのスキーマが変更されてもコードを修正する必要がなく、メンテナンス性が向上します。

この%TYPE は、以下のように**既存のテーブルの列をピンポイントで参照**します。

**具体例：users テーブルの列を参照**

\-- 事前に存在するテーブル  
CREATE TABLE users (  
 id SERIAL PRIMARY KEY,  
 user_name TEXT NOT NULL,  
 email VARCHAR(255) UNIQUE  
);  
\-- PL/pgSQL コード  
DO $$  
DECLARE  
 \-- users テーブルの id 列と同じデータ型（INTEGER）を持つ変数  
 user_id users.id%TYPE;  
 \-- users テーブルの user_name 列と同じデータ型（TEXT）を持つ変数  
 full_name users.user_name%TYPE;  
BEGIN  
 user_id := 5;  
 full_name := '鈴木 一郎';  
 RAISE NOTICE 'ユーザー ID: %, 名前: %', user_id, full_name;  
END $$;

- **%ROWTYPE**: テーブルの行全体を一つの変数として扱うことができます。

**具体例：users テーブルの行を参照**

\-- users テーブルが存在すると仮定  
DO $$  
DECLARE  
 \-- users テーブルの行全体（id, user_name, email）を格納する変数  
 users_row users%ROWTYPE;  
BEGIN  
 \-- SELECT 文で取得した行を変数に代入  
 SELECT id, user_name, email INTO users_row FROM users WHERE id \= 1;  
 RAISE NOTICE 'ユーザー情報: ID: %, 名前: %, メール: %', users_row.id, users_row.user_name, users_row.email;  
END $$;

## **2\. 制御構造**

### **条件分岐（IF-THEN-ELSE）**

特定の条件に基づいて処理を切り替えることができます。例えば、在庫数に応じて対応を変えるようなロジックに利用します。

構文:  
IF 条件式 THEN ... ELSIF 条件式 THEN ... ELSE ... END IF;  
DO $$  
DECLARE  
 product_stock INT := 8;  
BEGIN  
 IF product_stock \> 10 THEN  
 RAISE NOTICE '在庫は十分にあります。';  
 ELSIF product_stock \> 0 AND product_stock \<= 10 THEN  
 RAISE NOTICE '在庫が少なくなっています。発注を検討してください。';  
 ELSE  
 RAISE NOTICE '在庫切れです。販売を停止します。';  
 END IF;  
END $$;

### **繰り返し処理（LOOP）**

データのリストを順に処理したり、特定の回数だけ処理を繰り返したりする場合に便利です。

構文:  
LOOP ... END LOOP; (基本的なループ)  
WHILE 条件式 LOOP ... END LOOP; (条件を満たす間ループ)  
FOR 変数 IN 開始値..終了値 LOOP ... END LOOP; (指定範囲をループ)  
DO $$  
DECLARE  
 i INT := 1;  
BEGIN  
 \-- WHILE ループの例: 条件が真である間繰り返す  
 WHILE i \<= 3 LOOP  
 RAISE NOTICE 'ループ実行中 (while): %', i;  
 i := i \+ 1;  
 END LOOP;

    \-- FOR ループの例: 指定した範囲を繰り返す
    FOR j IN 1..3 LOOP
        RAISE NOTICE 'ループ実行中 (for): %', j;
    END LOOP;

END $$;

## **3\. ストアドプロシージャとストアドファンクション**

データベースサーバーに保存して、後から呼び出せるようにした一連の処理のことです。これらを使うことで、アプリケーションとデータベースの間の通信回数を減らし、処理を効率化できます。

定義構文:  
CREATE \[OR REPLACE\] FUNCTION/PROCEDURE 関数/プロシージャ名(...) \[RETURNS ...\] AS $$ ... $$ LANGUAGE plpgsql;

### **ストアドファンクション**

- **戻り値が必須**です。RETURNS 句で戻り値の型を指定します。
- SELECT 文から呼び出すことが可能です。
- データの取得や計算など、**副作用のない**値を返す目的で使われます。

**具体例：消費税込みの金額を計算するファンクション**

CREATE OR REPLACE FUNCTION calculate_tax_inclusive_price(  
 price_before_tax NUMERIC  
)  
RETURNS NUMERIC AS $$  
BEGIN  
 RETURN price_before_tax \* 1.1; \-- 10%の消費税を計算  
END;

$$
LANGUAGE plpgsql;

\-- 呼び出し例
SELECT calculate\_tax\_inclusive\_price(1000);
\-- 出力: 1100.0

**ストアドファンクションの実際のケース**

* **複雑な計算ロジックの再利用**: SELECT文の中で、複雑な計算を何度も行う場合に便利です。例えば、商品価格に割引率と送料を適用した最終価格を計算するロジックをファンクションとして定義すれば、複数のクエリから呼び出すことができます。
* **データの整形・変換**: JSONデータから特定のフィールドを抽出したり、日付や時刻のフォーマットを変換したりする場合に使われます。これにより、SQLクエリがシンプルになり、可読性が向上します。

### **ストアドプロシージャ**

* **戻り値は任意**です。RETURNS句は使用しません。
* CALL文で呼び出します。
* **複数のCOMMITやROLLBACKを含むトランザクション制御が可能**です。
* 複雑なデータ更新や、一連のバッチ処理など、**データベースの状態を変更する**目的で使われます。

#### **トランザクション管理について**

**トランザクション**は、一連のデータベース操作を一つの論理的な単位として扱うための仕組みです。すべての操作が成功した場合は全体を\*\*コミット（COMMIT）**し、一つでも失敗した場合は全体を**ロールバック（ROLLBACK）\*\*して変更を元に戻します。これにより、データベースのデータの整合性が保たれます。

PL/pgSQLでは、ストアドプロシージャ内でのみトランザクション制御が可能です。これは、プロシージャが一連の処理をまとめて実行するのに適しているためです。一方、ストアドファンクションは、原則として単一の操作の一部として扱われるため、直接的なトランザクション制御はできません。

**具体的なトランザクション制御の例:**

CREATE OR REPLACE PROCEDURE transfer\_funds(
    sender\_account\_id INT,
    receiver\_account\_id INT,
    amount NUMERIC
)
LANGUAGE plpgsql AS
$$

BEGIN  
 \-- 送金元の残高を減らす  
 UPDATE accounts SET balance \= balance \- amount WHERE id \= sender_account_id;

    \-- 送金先の残高を増やす
    UPDATE accounts SET balance \= balance \+ amount WHERE id \= receiver\_account\_id;

    \-- 両方の操作が成功した場合、ここでコミット
    COMMIT;

EXCEPTION  
 \-- いずれかの操作でエラーが発生した場合  
 WHEN OTHERS THEN  
 \-- 全ての変更を元に戻す（ロールバック）  
 ROLLBACK;  
 RAISE EXCEPTION '送金処理中にエラーが発生しました。';  
END;

$$
;

**ストアドプロシージャの実際のケース**

* **複合的なデータ更新処理**: ECサイトの注文処理（在庫数の減少、注文履歴の追加、顧客のポイント更新など）のように、複数のテーブルに対する操作をまとめて実行し、**データの整合性を保ちたい**場合に最適です。
* **定期的なバッチ処理**: 月末の締め処理、日次のデータ集計、ログのアーカイブなど、定時に実行される**重たい処理**をデータベースサーバー側で効率的に実行したい場合に利用します。

### **使い分けのまとめ**

* **ファンクション:** 特定の値を計算・取得する、**副作用のない**処理。
* **プロシージャ:** 複数のUPDATEやINSERTを組み合わせるなど、**データベースの状態を変更する**一連の処理。

## **4\. エラーハンドリング（EXCEPTION）**

予期せぬエラー（例：存在しないIDを指定された、データ型が一致しないなど）が発生した際に、プログラムが中断せずに適切な処理を行えるようにします。

構文:
BEGIN ... EXCEPTION WHEN エラー条件 THEN ... END;
DO
$$

DECLARE  
 product_id_to_find INT := 999; \-- 存在しない商品 ID  
 product_name TEXT;  
BEGIN  
 \-- 存在しない商品 ID でデータを取得しようとする  
 SELECT name INTO product_name FROM products WHERE id \= product_id_to_find;

    \-- \`SELECT\`文で結果が見つからなかった場合に\`NOT FOUND\`となる
    IF NOT FOUND THEN
        RAISE NOTICE '商品ID %は存在しませんでした。', product\_id\_to\_find;
    END IF;

EXCEPTION  
 \-- 予期せぬ全てのエラーを捕捉  
 WHEN OTHERS THEN  
 RAISE NOTICE '予期せぬエラーが発生しました。';  
 \-- エラーの詳細メッセージを出力  
 RAISE NOTICE 'エラー内容: %', SQLERRM;  
END $$;

**エラーハンドリングの実際のケース**

- **データの重複挿入を回避**: 既に存在するユーザー ID やメールアドレスを挿入しようとした場合、UNIQUE 制約違反が発生します。EXCEPTION ブロックでこれを捕捉し、「このユーザーは既に存在します」といった分かりやすいメッセージを返すことで、アプリケーション側のエラー処理を簡潔にできます。
- **ビジネスロジックの違反を処理**: 在庫数が 0 未満になるような注文をキャンセルしたり、送金額が残高を超える場合にエラーを発生させたりするなど、ビジネス上のルールに違反する操作を防ぐために使われます。

## **まとめ**

PL/pgSQL は、SQL の柔軟性を高め、データベースサーバー上で複雑な処理を効率的に実行するための強力なツールです。

- **変数や制御構造**を使い、SQL だけでは実現できないロジックを実装する。
- **ストアドファンクション**で、計算やデータ取得を再利用可能な部品として定義する。
- **ストアドプロシージャ**で、複数のデータ操作を含む一連の業務処理を確実に行う。
- **エラーハンドリング**を実装し、堅牢なシステムを構築する。

まずは簡単なファンクションやプロシージャを作成することから始め、徐々に複雑なロジックに挑戦してみてください。

### **PL/pgSQL 入門：SQL エンジニアのための講義資料**

この資料は、**SQL**の知識はあるものの、**PL/pgSQL**（PostgreSQL の手続き型言語）に馴染みのない方向けのものです。提供された問題を題材に、PL/pgSQL の基本的な概念と、SQL との違いを理解することを目的とします。

### **SQL と PL/pgSQL の違い**

SQL は「何をしたいか」を記述する**宣言型言語**です。例えば UPDATE 文は「この条件に合致するレコードを、この値に更新せよ」という結果を宣言しています。

一方、PL/pgSQL は「どのように処理するか」を記述する**手続き型言語**です。変数の宣言、条件分岐（IF 文）、ループ、例外処理など、より複雑なロジックを段階的に記述できます。ストアドプロシージャは、この PL/pgSQL を使ってデータベースサーバー上で実行される一連の処理をひとまとめにしたものです。

### **問題 1: 会員ランク更新プロシージャの解説**

この問題は、引数を受け取って UPDATE 文を実行するという、比較的単純な処理です。

**コードの解説**

- CREATE OR REPLACE PROCEDURE: プロシージャを新しく作成するか、既存のものを置き換える構文です。
- p_customer_id INT, p_new_membership_id INT: プロシージャが受け取る引数（パラメータ）です。変数名の先頭に p\_ をつけるのは、パラメータであることを示す一般的な命名規則です。
- LANGUAGE plpgsql AS $$ ... $$;: このプロシージャの言語が PL/pgSQL であることを宣言しています。$$ は、文字列リテラルを囲むための**ドル記号クォート**と呼ばれる特殊な記号で、シングルクォート ' をエスケープせずに済みます。
- DECLARE ... BEGIN ... END;: **DECLARE** はローカル変数の宣言、**BEGIN ... END** はメインロジックを記述するブロックです。
- UPDATE Customers SET membership_id \= p_new_membership_id WHERE customer_id \= p_customer_id;: 通常の SQL 文です。引数である p_customer_id を直接使用している点に注目してください。
- RAISE NOTICE '顧客 % の会員ランクを更新しました。', p_customer_id;: **RAISE NOTICE** はユーザーにメッセージを出力します。% の部分に後続の引数 p_customer_id の値が埋め込まれます。

### **問題 2: レビュー追加プロシージャの解説**

この問題では、**条件分岐**と**例外処理**という、PL/pgSQL ならではの機能が使われています。

**コードの解説**

- DECLARE v_cnt INT;: v_cnt という名前の**ローカル変数**を整数型で宣言しています。
- SELECT COUNT(\*) INTO v_cnt FROM ...;: SQL の結果をローカル変数に格納する構文です。INTO を使うことで、SELECT 文の実行結果を v_cnt に代入できます。
- IF v_cnt \= 0 THEN ... END IF;: **IF 文**による条件分岐です。v_cnt が 0 であれば、THEN から END IF までのブロックが実行されます。
- RAISE EXCEPTION '商品未購入のため、レビューできません。';: **RAISE EXCEPTION** はエラーを発生させて、プロシージャの実行を中断します。

### **問題 3: 注文作成プロシージャの解説**

この問題では、さらに複雑なロジックが組み込まれています。

**コードの解説**

- SELECT stock_quantity INTO v_stock FROM Inventory ...;: 問題 2 と同様に、SELECT 文の結果を v_stock 変数に代入しています。
- IF v_stock IS NULL OR v_stock \< p_quantity THEN ... END IF;: **在庫不足のチェック**です。v_stock が NULL の場合や、購入数量 p_quantity より少ない場合に例外を発生させます。
- INSERT ... RETURNING order_id INTO v_order_id;: INSERT 文の後に RETURNING 句を使うことで、挿入された行の特定のカラム（この場合は order_id）の値を取得し、INTO でローカル変数 v_order_id に格納しています。

### **まとめ**

- **PL/pgSQL は手続き型言語**であり、SQL では実現できない複雑なロジックを記述できます。
- **DECLARE**: ローカル変数を宣言します。
- **INTO**: SELECT 文の結果を変数に格納します。
- **RAISE NOTICE**: ユーザーにメッセージを出力します。
- **RAISE EXCEPTION**: 例外を発生させて処理を中断します。
- **IF ... THEN ... END IF**: 条件分岐を実行します。
- **RETURNING ... INTO**: INSERT や UPDATE で変更された行の値を返します。

これらの概念を理解することで、PL/pgSQL を使ったストアドプロシージャの作成・デバッグがスムーズに行えるようになるはずです。
