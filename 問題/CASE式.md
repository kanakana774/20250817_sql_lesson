## **EC サイトデータ CASE 式問題集**

### **問題 1: 商品の価格帯を分類する（シンプル CASE 式）**

- **重要度**: ★★★★
- **問題目的**: シンプル CASE 式を使用して、数値データ（価格）を基に新しいカテゴリを作成する方法を理解する。
- **問題**: products_mst テーブルから、各商品の**価格帯**を以下の基準で分類し、商品名、価格、そして分類された価格帯を表示してください。
  - 20000 円以上: '高価格帯'
  - 5000 円以上 20000 円未満: '中価格帯'
  - 5000 円未満: '低価格帯'
- **答え**:  
  SELECT  
   product_name,  
   price,  
   CASE  
   WHEN price \>= 20000 THEN '高価格帯'  
   WHEN price \>= 5000 THEN '中価格帯'  
   ELSE '低価格帯'  
   END AS price_category  
  FROM  
   products_mst  
  ORDER BY  
   price DESC;

### **問題 2: 顧客の登録時期をセグメント化する（検索 CASE 式と日付関数）**

- **重要度**: ★★★★★
- **問題目的**: 検索 CASE 式と日付関数を組み合わせて、日付データを基に顧客をセグメント化する方法を理解する。
- **問題**: customers_mst テーブルから、各顧客の**登録時期**を以下の基準で分類し、顧客名、登録日、そして分類された登録時期を表示してください。
  - 2023 年 1 月 1 日から 2023 年 3 月 31 日まで: '初期登録顧客'
  - 2023 年 4 月 1 日から 2023 年 6 月 30 日まで: '中期登録顧客'
  - 2023 年 7 月 1 日以降: '最近登録顧客'
  - 上記以外: 'その他'
- **答え**:  
  SELECT  
   customer_name,  
   created_date,  
   CASE  
   WHEN created_date BETWEEN '2023-01-01' AND '2023-03-31' THEN '初期登録顧客'  
   WHEN created_date BETWEEN '2023-04-01' AND '2023-06-30' THEN '中期登録顧客'  
   WHEN created_date \>= '2023-07-01' THEN '最近登録顧客'  
   ELSE 'その他'  
   END AS registration_segment  
  FROM  
   customers_mst  
  ORDER BY  
   created_date ASC;

### **問題 3: 商品カテゴリごとの在庫状況を評価する**

- **重要度**: ★★★★★
- **問題目的**: GROUP BY 句と SUM()、CASE 式を組み合わせ、条件に応じた集計（クロス集計の基礎）を行う方法を理解する。
- **問題**: products_mst テーブルから、**カテゴリごと**に、以下の在庫状況別の商品数を集計してください。
  - stock_quantity が 100 個以上: '豊富'
  - stock_quantity が 10 個以上 100 個未満: '通常'
  - stock_quantity が 10 個未満: '品薄'  
    各カテゴリの豊富、通常、品薄の商品の数を表示し、合計商品数も併せて表示してください。
- **答え**:  
  SELECT  
   category,  
   COUNT(CASE WHEN stock_quantity \>= 100 THEN 1 END) AS stock_abundant,  
   COUNT(CASE WHEN stock_quantity \>= 10 AND stock_quantity \< 100 THEN 1 END) AS stock_normal,  
   COUNT(CASE WHEN stock_quantity \< 10 THEN 1 END) AS stock_low,  
   COUNT(\*) AS total_products_in_category  
  FROM  
   products_mst  
  GROUP BY  
   category  
  ORDER BY  
   category;

### **問題 4: 特定カテゴリの商品を優先して並び替える（ORDER BY 句での CASE 式）**

- **重要度**: ★★★★★
- **問題目的**: ORDER BY 句内で CASE 式を使用し、特定の条件を満たす行を優先的に表示するカスタムソートの方法を理解する。
- **問題**: products_mst テーブルから、商品を以下の優先順位で並び替えて表示してください。
  1. 'Electronics'カテゴリの商品を最優先。
  2. 次に'Books'カテゴリの商品。
  3. それ以外のカテゴリの商品。  
     各グループ内では、価格が高い順に並び替えてください。
- **答え**:  
  SELECT  
   product_name,  
   category,  
   price  
  FROM  
   products_mst  
  ORDER BY  
   CASE category  
   WHEN 'Electronics' THEN 1  
   WHEN 'Books' THEN 2  
   ELSE 3 \-- その他のカテゴリ  
   END,  
   price DESC;

### **問題 5: 特定の顧客のみ注文日を調整して表示する（WHERE 句での CASE 式と注意点）**

- **重要度**: ★★★★★
- **問題目的**: WHERE 句で CASE 式を使用する例を提示し、そのメリットとデメリット、そして推奨される書き方を対比して理解を促す。
- **問題**: orders_trn テーブルから、customer_id が 1（佐藤 太郎）の注文は**2023 年 8 月 10 日以降**のものを表示し、それ以外の顧客の注文は**全ての注文**を表示してください。CASE 式を使ってフィルタリングしてください。
- **答え**:  
  \-- CASE 式を使った方法 (学習用 \- 実務では推奨されないことが多い)  
  SELECT  
   order_id,  
   customer_id,  
   order_date  
  FROM  
   orders_trn  
  WHERE  
   CASE  
   WHEN customer_id \= 1 THEN order_date \>= '2023-08-10' \-- customer_id が 1 の場合は日付で絞り込む  
   ELSE TRUE \-- それ以外の顧客は常に TRUE（全ての注文を表示）  
   END  
  ORDER BY  
   customer_id, order_date;

- **推奨されるバリエーション（実務向け）**:  
  \-- 論理演算子を使った推奨される方法  
  SELECT  
   order_id,  
   customer_id,  
   order_date  
  FROM  
   orders_trn  
  WHERE  
   (customer_id \= 1 AND order_date \>= '2023-08-10') \-- customer_id が 1 で、かつ日付条件を満たす  
   OR  
   (customer_id \<\> 1\) \-- または customer_id が 1 ではない  
  ORDER BY  
   customer_id, order_date;

- **補足**: WHERE 句での CASE 式は、複雑な条件を簡潔に記述できる場合がありますが、パフォーマンス面や可読性で問題が生じることがあります。通常は AND や OR などの論理演算子を組み合わせた方法が推奨されます。この問題を通じて、両者の違いとそれぞれの状況での使い分けを議論してください。

### **問題 6: 商品メモの有無と在庫状況に応じた評価（NULL と複数条件）**

- **重要度**: ★★★★★
- **問題目的**: CASE 式内で IS NULL と複数の条件を組み合わせ、柔軟なデータ分類を行う。ELSE 句での NULL の扱いを再確認する。
- **問題**: products_mst テーブルから、各商品の**詳細状況**を以下の基準で評価し、商品名、在庫数、メモ、そして評価を表示してください。
  - メモが NULL ではなく、かつ在庫数が 50 個以上: '詳細充実・在庫あり'
  - メモが NULL ではなく、かつ在庫数が 50 個未満: '詳細充実・在庫少'
  - メモが NULL で、かつ在庫数が 0 個より大きい: '詳細未入力・販売中'
  - 上記以外（例: メモが NULL で、在庫数が 0 の場合など）: 'その他'
- **答え**:  
  SELECT  
   product_name,  
   stock_quantity,  
   memo,  
   CASE  
   WHEN memo IS NOT NULL AND stock_quantity \>= 50 THEN '詳細充実・在庫あり'  
   WHEN memo IS NOT NULL AND stock_quantity \< 50 THEN '詳細充実・在庫少'  
   WHEN memo IS NULL AND stock_quantity \> 0 THEN '詳細未入力・販売中'  
   ELSE 'その他'  
   END AS detail_and_stock_status  
  FROM  
   products_mst  
  ORDER BY  
   detail_and_stock_status, product_name;

これらの CASE 式の問題は、データの変換、集計、並び替え、そしてフィルタリングといった様々な場面で CASE 式がいかに強力であるかを学ぶのに役立つはずです。特に実務では、レポート作成や複雑なビジネスロジックの実装に不可欠な知識となります。

次に、ALTER TABLE や DROP TABLE に関する問題、またはテーブル間の結合（JOIN）に焦点を当てた問題に進みますか？ご希望のテーマをお知らせください！
