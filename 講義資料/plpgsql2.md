### **PL/pgSQL 基礎文法講義**

#### **はじめに**

PL/pgSQL は PostgreSQL に組み込まれた手続き型言語です。SQL だけでは難しい**複雑なロジックをデータベース内部で直接処理する**ために使用します。これにより、クライアントとデータベース間の通信量を減らし、パフォーマンスを向上させることができます。

#### **1\. 無名ブロック (Anonymous Block)**

無名ブロックは、データベースに保存されない、一度限りの PL/pgSQL コードの塊です。主に以下のような目的で使います。

- **簡単なスクリプトの実行**: データのバックアップ、一時的な更新、メンテナンス作業などに使用します。
- **関数のテスト**: 本番環境にデプロイする前に、新しいロジックが期待通りに動作するか手軽に確認できます。

**構文例**:

DO $$  
DECLARE  
 \-- 変数を宣言する場所  
BEGIN  
 \-- 実行するロジック  
END $$ LANGUAGE plpgsql;

#### **2\. 関数とプロシージャ**

PL/pgSQL では、データベースに保存して繰り返し使用する\*\*関数（FUNCTION）**と**プロシージャ（PROCEDURE）\*\*を定義できます。

**関数とプロシージャの根本的な違い**:

| 特徴                 | 関数 (FUNCTION)                                       | プロシージャ (PROCEDURE)                     |
| :------------------- | :---------------------------------------------------- | :------------------------------------------- |
| **目的**             | **値を返すこと**を主目的とする。                      | **値を返さない**。複数の SQL 文の集合体。    |
| **呼び出し**         | SELECT 文の中で呼び出す。CALL も可能。                | CALL 文で呼び出す。SELECT では呼び出せない。 |
| **トランザクション** | 通常、呼び出し元のトランザクション内で実行。          | 独自に COMMIT や ROLLBACK を実行できる。     |
| **戻り値**           | RETURN で単一の値、RETURNS SETOF で結果セットを返す。 | OUT パラメータを介して値を返すことは可能。   |

**構文例**:

\-- 関数 (FUNCTION) の例  
CREATE OR REPLACE FUNCTION get_total_sales()  
RETURNS NUMERIC AS $$  
DECLARE  
 total_sales NUMERIC;  
BEGIN  
 SELECT SUM(price) INTO total_sales FROM sales;  
 RETURN total_sales; \-- 値を返す  
END;

$$
LANGUAGE plpgsql;

\-- プロシージャ (PROCEDURE) の例
CREATE OR REPLACE PROCEDURE log\_message(message TEXT)
AS
$$

BEGIN  
 INSERT INTO log_table (log_text) VALUES (message);  
END;

$$
LANGUAGE plpgsql;

\-- プロシージャの呼び出し
CALL log\_message('プロシージャが呼び出されました。');

#### **3\. 変数とスコープ**

変数は DECLARE ブロックで宣言します。データ型を指定することで、格納するデータの種類を明確にします。

DECLARE ブロックは、実行ブロック (BEGIN...END) の前に置かれ、PL/pgSQLコード内で使用するすべての変数、定数、カーソル、および行型を定義する役割を担います。

**スコープと変数の寿命**:

PL/pgSQLの変数は、それを定義したBEGIN...ENDブロック内で有効です。ブロックが終了すると、そのブロックで宣言された変数は消滅します。**ネストしたブロック内で同じ名前の変数を宣言すると、内側のブロックでは新しい変数が優先され、外側の変数は一時的に隠されます**。これは、コードの意図しない挙動を引き起こす可能性があるため、注意が必要です。

**具体例**:

DO
$$

DECLARE  
 \-- 外側のスコープ  
 counter INTEGER := 10;  
BEGIN  
 RAISE NOTICE '外側のカウンター: %', counter; \-- 10

    BEGIN
        \-- 内側のスコープ
        DECLARE
            counter INTEGER := 20;
        BEGIN
            RAISE NOTICE '内側のカウンター: %', counter; \-- 20
        END;

    END;

    RAISE NOTICE '外側のカウンター（再確認）: %', counter; \-- 10

END;

$$
LANGUAGE plpgsql;

#### **3.1 制御構造**

PL/pgSQLでは、一般的なプログラミング言語と同様に、条件分岐やループを使ってロジックの流れを制御できます。

* **IF-THEN-ELSE**: 条件に基づいて処理を分岐させます。

**例**:

DECLARE
    emp\_salary NUMERIC := 120000;
BEGIN
    IF emp\_salary \> 100000 THEN
        RAISE NOTICE '高給取り';
    ELSEIF emp\_salary \> 50000 THEN
        RAISE NOTICE '平均的な給与';
    ELSE
        RAISE NOTICE '低めの給与';
    END IF;
END;

* **CASE**: 複数の条件分岐をより簡潔に記述します。

**例**:

DECLARE
    employee\_department VARCHAR := 'IT';
BEGIN
    CASE employee\_department
        WHEN 'Sales' THEN
            RAISE NOTICE '営業部門';
        WHEN 'IT' THEN
            RAISE NOTICE 'IT部門';
        ELSE
            RAISE NOTICE 'その他の部門';
    END CASE;
END;

#### **IF文とCASE文の使い分け**

IF文とCASE文はどちらも条件分岐に使いますが、その目的と最適な使い方が異なります。

* **IF文**: **複数の異なる条件**を評価する際に適しています。特に、条件が論理式（\>,\<,AND,ORなど）で表現され、それぞれが独立している場合に真価を発揮します。例として、給与の範囲に応じて異なるメッセージを出力するような場合です。
* **CASE文**: **単一の変数が持つ複数の値**に基づいて処理を分岐させる際に最適です。これにより、コードがより簡潔で読みやすくなります。特に、列挙型のような事前に決められた値（例: 'Sales', 'IT')に沿って処理を分けたい場合に非常に有効です。

CASE文には、単一の値と比較する\*\*シンプルCASE**と、論理式を記述する**検索CASE\*\*の2つの形式があります。機能的に見れば検索CASEはIF-THEN-ELSEIFとほぼ同じですが、**コードの意図を明確にする**という点で使い分けが重要になります。

#### **3.2 ループ処理のバリエーションと使い分け**

PL/pgSQLには、様々な状況に対応できるよう複数のループ構文が用意されています。それぞれの特性を理解し、適切なものを選ぶことがパフォーマンスと可読性の向上につながります。

* LOOP ... END LOOP:
  最も基本的なループで、条件なしで無限に繰り返します。ループを終了させるためには、EXITまたはRETURNを明示的に使用する必要があります。このループは、特定の条件が満たされるまで繰り返すことが決まっている場合に適しています。
* WHILE ... LOOP:
  ループに入る前に条件を評価し、条件が真である限り繰り返します。条件が偽になった時点でループを終了します。ループの反復回数が事前に不明で、特定の条件が満たされるまで処理を続けたい場合に適しています。
* FOR ... IN ... LOOP:
  クエリの結果セットを1行ずつ処理するのに最適な、最も一般的に使われるループです。ループ変数は自動的に宣言され、クエリが返す各行が代入されます。これにより、コードが簡潔で読みやすくなります。大量のデータを反復処理する際には、できるだけこの構文を使うことが推奨されます。
* FOREACH ... IN ARRAY ... LOOP:
  配列の要素を一つずつ処理するのに最適なループです。この構文は、データベースから取得した配列データや、ローカルで宣言した配列を簡単に反復処理する際に非常に役立ちます。

**ループの制御**:

* **EXIT**: ループを完全に終了します。EXIT WHENという形で条件を指定することもできます。
* **CONTINUE**: ループの現在のイテレーションをスキップし、次のイテレーションに進みます。CONTINUE WHENという形で条件を指定することもできます。

#### **3.3 戻り値の扱い方**

関数内で値を返す方法は、返したいデータの種類によって使い分けます。

* RETURN:
  単一の値を返す最も一般的な方法です。
* RETURN NEXT:
  RETURNS SETOFを指定した関数で使用します。この文を実行するたびに、現在処理中の行を結果セットに追加し、関数は処理を継続します。すべての行を処理し終えたら、RETURN（引数なし）で関数を終了させます。
* RETURN QUERY:
  RETURNS SETOFを指定した関数で使用します。SELECT文のクエリ結果全体を一度に返します。RETURN NEXTが1行ずつ結果を構築するのに対し、RETURN QUERYはシンプルかつ効率的に結果を返すため、結果セットを生成するロジックが単純な場合はこちらが推奨されます。

**例**:

\-- RETURN NEXT を使用して結果セットを返す
CREATE OR REPLACE FUNCTION get\_high\_paid\_employees()
RETURNS SETOF employees AS
$$

DECLARE  
 emp_record employees%ROWTYPE;  
BEGIN  
 FOR emp_record IN SELECT \* FROM employees WHERE salary \> 100000 LOOP  
 RETURN NEXT emp_record; \-- 各行を結果セットに追加  
 END LOOP;  
 RETURN; \-- 関数を終了  
END;

$$
LANGUAGE plpgsql;

\-- RETURN QUERY を使用して結果セットを返す
CREATE OR REPLACE FUNCTION get\_sales\_data\_by\_month()
RETURNS TABLE (month TEXT, total\_sales NUMERIC) AS
$$

BEGIN  
 RETURN QUERY  
 SELECT to_char(sale_date, 'YYYY-MM'), SUM(sale_amount)  
 FROM sales  
 GROUP BY to_char(sale_date, 'YYYY-MM')  
 ORDER BY 1;  
END;

$$
LANGUAGE plpgsql;

#### **3.4 カーソル（Cursors）の詳細**

FORループが暗黙的にカーソルを使用する一方、PL/pgSQLでは**明示的なカーソル**を宣言して使用することもできます。これは、より複雑なデータ処理フローや、結果セットを部分的に処理する必要がある場合に役立ちます。

**カーソルのライフサイクル**:

1. **宣言 (DECLARE)**: カーソルを宣言し、それに関連付けるクエリを定義します。
2. **オープン (OPEN)**: カーソルを開き、クエリを実行して結果セットを生成します。
3. **フェッチ (FETCH)**: 結果セットから1行ずつデータを取得し、変数に代入します。FETCH FORWARD n を使うと、一度にn行を取得する**バッチ処理**が可能で、パフォーマンス改善に役立ちます。
4. **クローズ (CLOSE)**: カーソルを閉じ、使用していたリソースを解放します。

**実務での注意点**:

* **トランザクションをまたげない**: 明示的なカーソルは、それをオープンしたトランザクション内でのみ有効です。COMMITやROLLBACKを実行するとカーソルは自動的に閉じられます。ただし、WITH HOLDを指定したセッションカーソルは、トランザクションをまたいで利用できます。
* **ドライバのフェッチサイズ**: JDBCなどのデータベースドライバは、クエリ結果を一度にすべてメモリにロードするのではなく、FETCH FORWARDと同様に、指定された行数ずつフェッチする設定を持つことが多いです。PL/pgSQL内で明示的なカーソルを使う場合と、アプリケーション側で大量データを扱う場合とで、パフォーマンスの考慮点が異なることを理解しておきましょう。

**使い分けのベストプラクティス**:

ほとんどの場合、FOR ... IN ... LOOP が最もシンプルで効率的な選択肢です。**明示的なカーソルは、以下のような特定の状況でのみ使用します。**

* ループ中に**動的な条件**でカーソルを**再定義**する必要がある場合。
* ループの途中で、複数のカーソルを切り替える必要がある場合。
* 結果セット全体ではなく、**特定の行数だけを処理したい**場合。

#### **4\. 例外処理 (Exception Handling)**

本番環境で堅牢なコードを記述するには、予期せぬエラーに備えることが不可欠です。EXCEPTION ブロックを使用することで、エラーが発生しても処理が中断することなく、安全にリカバリできます。

**特定の例外を捕捉する**:

PL/pgSQLでは、一般的なエラー条件（no\_data\_found, unique\_violationなど）や、SQLSTATEコード（5桁の英数字コード）を基にした**より具体的な例外**を捕捉できます。

**例**:

BEGIN
    \-- 存在しない顧客IDを指定
    DECLARE
        customer\_id INT := 99999;
        customer\_name VARCHAR;
    BEGIN
        SELECT name INTO STRICT customer\_name FROM customers WHERE id \= customer\_id;
    EXCEPTION
        WHEN NO\_DATA\_FOUND THEN \-- データが見つからないエラー
            RAISE NOTICE '顧客ID % は見つかりませんでした。', customer\_id;
        WHEN TOO\_MANY\_ROWS THEN \-- 複数行が返されたエラー
            RAISE EXCEPTION 'ID % に複数の顧客が存在します。', customer\_id;
        WHEN unique\_violation THEN \-- 一意性制約違反エラー
            RAISE NOTICE '一意性違反です。';
    END;
END;

**RAISE文とエラーのカスタマイズ**:

RAISE文を使用すると、ユーザー定義の例外を意図的に発生させることができます。

* RAISE NOTICE/INFO/WARNING: ログレベルのメッセージを出力し、処理は継続します。デバッグに非常に役立ちます。
* RAISE EXCEPTION: 処理を中断し、エラーを発生させます。USING句で詳細な情報を付加できます。

**例**:

RAISE EXCEPTION '無効なユーザーIDが指定されました' USING ERRCODE \= '22000', DETAIL \= 'ユーザーIDはNULLにできません。';

**エラーの伝播とトランザクション制御**:

PL/pgSQL関数は、通常、呼び出し元のトランザクション内で実行されます。関数内でエラーが発生すると、デフォルトでは**トランザクション全体がロールバックされます**。

部分的なロールバックが必要な場合は、SAVEPOINTを組み合わせて使用します。これにより、特定の地点まで処理を戻すことが可能になります。これは、バッチ処理で一部のレコードだけが失敗した場合に、残りのレコードの処理を継続したい場合に有効です。

**入れ子になった例外ブロック**:

BEGIN...EXCEPTION...ENDはネストできるため、より詳細なエラーハンドリングが可能です。外側のブロックが全体的なリカバリを担当し、内側のブロックが特定のサブ処理の失敗を捕捉する、といった使い方ができます。

**例**:

BEGIN
    \-- 全体処理の開始
    BEGIN
        \-- 特定のサブ処理
        INSERT INTO a\_table VALUES (1);
    EXCEPTION
        WHEN unique\_violation THEN
            RAISE NOTICE '重複エラーが発生しました。この処理はスキップします。';
    END;

    \-- エラーが発生しても、この後の処理は継続される
    INSERT INTO b\_table VALUES (2);
END;

#### **5\. 動的SQL（Dynamic SQL）**

PL/pgSQL内で、実行時にSQL文を動的に生成・実行する機能です。テーブル名やカラム名が変数によって変わるような柔軟な処理を実装する際に役立ちます。

**動的SQLのセキュリティリスク：SQLインジェクション**

ユーザーからの入力を直接SQL文字列に連結すると、悪意のあるユーザーが意図しないコマンドを挿入し、データベースを破壊する可能性があります。これを**SQLインジェクション**と呼びます。

**安全な動的SQLのベストプラクティス**:

SQLインジェクションを防ぐため、**プレースホルダーを使用する**ことが絶対的なベストプラクティスです。format()関数とEXECUTE ... USINGを適切に使い分けることが重要です。

* **EXECUTE ... USING句**: \*\*リテラル値（検索条件の値など）\*\*をバインドする際に、最も安全で効率的な方法です。これは、値がSQL文に直接埋め込まれるのではなく、パラメータとして渡されるため、SQLインジェクションを完全に防ぎ、実行速度も最適化されます。
* **format()関数**: \*\*識別子（テーブル名やカラム名）\*\*を動的に指定する際に使います。%Iフォーマット指定子を使用することで、自動的に識別子を二重引用符で囲み、安全に挿入します。

**使い分けの例**:

DECLARE
    table\_name TEXT := 'user\_data';
    user\_id INT := 123;
    result\_count INT;
BEGIN
    \-- 識別子（テーブル名）にはformat()と%Iを使用
    \-- 値（user\_id）にはUSING句を使用
    EXECUTE format('SELECT COUNT(\*) FROM %I WHERE id \= $1', table\_name)
    INTO result\_count
    USING user\_id;

    RAISE NOTICE 'テーブル % にID % のユーザーが % 人見つかりました。', table\_name, user\_id, result\_count;
END;

#### **6\. トリガプロシージャ (Trigger Procedures)**

トリガプロシージャは、INSERT、UPDATE、DELETEなどの特定のイベントがテーブルで発生した際に自動的に実行される特別な関数です。データの整合性維持、監査ログの記録、複雑なビジネスルールの適用などに不可欠な機能です。

**トリガの種類**:

* **BEFORE / AFTER**: イベント発生前か後に実行するかを制御します。
  * BEFORE: データの変更前に実行され、NEWレコードを修正することで、挿入・更新される値を変更できます。
  * AFTER: データの変更が完了した後に実行され、監査ログ記録など、別のテーブルへの操作に主に使われます。
* **INSTEAD OF**: ビューに対するINSERT/UPDATE/DELETE操作を、実際のテーブルに対する操作に置き換えます。
* **行レベル・文レベル**:
  * **FOR EACH ROW (行レベル)**: 変更される**各行ごと**にトリガが実行されます。NEWとOLD変数にアクセスできます。
  * **FOR EACH STATEMENT (文レベル)**: INSERTやUPDATE文全体に対して**1回だけ**実行されます。NEWやOLDは利用できませんが、大量のデータ変更時にオーバーヘッドを減らせます。

**実務での注意点：トリガの副作用**:

トリガは強力なツールですが、不用意に使うとパフォーマンスの問題や予期しない挙動を引き起こす可能性があります。特に注意すべきは**トリガの再帰的な発火**です。

トリガ内で同じテーブルをUPDATEするロジックを記述すると、UPDATE操作がトリガを再度発火させ、無限ループに陥る可能性があります。これを防ぐには、以下のような工夫が必要です。

* **条件の追加**: トリガ内でWHEN句やIF文を使って、特定の条件が満たされたときのみ処理を実行する。
* **トリガ無効化**: ALTER TABLE ... DISABLE TRIGGERを使って、一時的にトリガを無効化する。
* **トリガ関数内のチェック**: TG\_OP（操作の種類）、TG\_LEVEL（行レベルか文レベルか）、TG\_WHEN（BEFOREかAFTERか）などの特殊変数を使い、再帰的な呼び出しを防ぐロジックを実装する。

#### **7\. 実務で役立つTips**

* **デバッグ**:
  * RAISE NOTICE: 開発中に変数の値を確認するのに最も手軽な方法です。
  * plpgsql.print\_strict\_params: PostgreSQLのGUC設定（General User Configuration）で、SELECT INTO STRICTが0行や複数行を返したときに、より詳細なデバッグ情報（どのパラメータが問題かなど）をログに出力できます。
* **セキュリティ定義子**:
  * **SECURITY DEFINER**: 関数を定義したユーザーの権限で実行されます。これにより、権限の低いユーザーでも、通常はアクセスできないデータにアクセスする関数を実行できます。
  * **SECURITY INVOKER**: 関数を呼び出したユーザーの権限で実行されます。これがデフォルトの挙動です。

SECURITY DEFINERを使う際は、**SQLインジェクションのリスクが非常に高くなる**ため、動的SQLを絶対に避け、すべての入力値を厳密に検証することが不可欠です。

* **静的解析**:
  * plpgsql\_check 拡張機能を使用すると、PL/pgSQLコードの静的解析を行い、実行前に潜在的なバグ（変数の使用間違いなど）を検出できます。実運用前のコードレビューに組み込むと非常に有用です。
$$
