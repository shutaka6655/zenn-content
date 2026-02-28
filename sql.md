# Python × SQLで最初に躓く「SQLの実行順序」を理解する

## ✅ 結論サマリ（2行）
- SQLは手続き型ではなく宣言型である  
- 実際に処理される順序は句（FROM, JOIN, WHERE…）で決まっている  

---

## サンプルテーブル
## サンプルテーブル
この記事で使う例として、以下の `users` テーブルを想定します。

| id | name    | age | city      |
|----|---------|-----|-----------|
| 1  | Alice   | 25  | Tokyo     |
| 2  | Bob     | 19  | Osaka     |
| 3  | Charlie | 30  | Tokyo     |
| 4  | Dave    | 22  | Osaka     |
| 5  | Eve     | 18  | Osaka     |

---

## SQLは宣言型
Pythonの`pandas`に慣れていると、つい「どう処理するか」を順番に書きたくなります。

```python
# pandas的には手続き型
users_df = users_df[users_df['age'] > 20]
users_df = users_df.groupby('city').sum()
```

これをSQLでそのままで書くとどうなる？
たぶん、サブクエリを使ってこう書くのではないでしょうか？

```sql
SELECT 
    city,
    COUNT(*) as count
FROM (
    SELECT * 
    FROM users
    WHERE age > 20                     
)
GROUP BY city
```
あれ？pandasで20歳以上にフィルターをかけようとしたら、1行なのにSQLにしたら3行も増える？
SQL書くのめんどくさいってなりませんか？

## SQLは宣言型
違います。

SQLは 「どう処理するかではなく、何を取得したいか」 を書く宣言型です。
SQLサーバーに完成図を渡すと、サーバー側が最適な実行計画を作って返してくれます。

SQLの実行順序（句の順番）

実際にSQLが処理する順序は以下の通りです：

1. FROM句
2. JOIN句
3. WHERE句
4. GROUP BY句
5. HAVING句
6. SELECT句
7. ORDER BY句
8. LIMIT句

つまり、SELECTで列を選ぶ前に、テーブルの結合や絞り込みは既に行われています。


```sql
SELECT 
    city, 
    COUNT(*) as count
FROM users
WHERE age > 20
GROUP BY city
```


まとめ

SQLは「完成図を宣言する」のが本質
処理順序を理解すると、デバッグや複雑なクエリ作成がラクになる

もちろん、
最初のSQLみたいに、サブクエリ・WITH句を使う場合も多くありますが、
前提は宣言型であり、よりシンプルに書けることを気に留めておきたいです！