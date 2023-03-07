---
marp: true
footer: "2023/03/07"

---
# SQLインジェクション with OWASP juice shop

---

# 今回やること

* Injectionとは
* SQLインジェクション
* OWASP juice shopによる実践と解決策

---

# Injectionとは

* 悪意のある攻撃者がウェブアプリケーションに不正なコードを注入することで、システムやデータを危険にさらす攻撃のこと
* 最新版である2021年の[OWASP TOP 10](https://owasp.org/www-project-top-ten/)では`Broken Access Control`や`Cryptographic Failures`に続いて三位に位置付けられている(以前の2017年のものでは一位だった。)
* [A03-2021-Injection](https://owasp.org/Top10/A03_2021-Injection/)

---

# 代表的なInjection

* SQLインジェクション
* XSS: クロスサイトスクリプティング
* OSコマンドインジェクション

今回はこの中でSQLインジェクションに関して取り扱う

---

# SQLインジェクション

* Webアプリケーションにおいてユーザーが指定した値を元にSQL文を組み立てる際に、その値を適切に処理しないままSQL文の一部として利用してしまうことにより、意図しないデータベース操作が可能となる脆弱性、およびその攻撃手法を指す(詳解セキュリティコンテストより)
* 任意のデータの取得や改ざん、削除といった攻撃を行うことができ、サービス提供者や利用者に対して様々な被害を与えうる

---

# SQLインジェクションの流れ

![](https://www.nttpc.co.jp/column/img/entry/security/sql_injection/img_01.jpg)

[引用元](https://www.nttpc.co.jp/column/security/sql_injection.html)

---

# SQLインジェクションの簡単な例

例えば、ユーザーがユーザー名とパスワードを入力するWebアプリケーションで、以下のようなSQL文が使われている場合を考えてみる。

```sql
SELECT * FROM users WHERE username = 'ユーザー名' AND password = 'パスワード';
```

ユーザー名とパスワードは、ユーザーが入力した値がそのまましようされるとする。このとき、ユーザーが以下のような値をusernameとして入力した場合、SQLインジェクション攻撃を行うことができる。

```
' OR 1 = 1 --
```

以下のようなSQLが発行され、全ユーザーが取得される

```sql
SELECT * FROM users WHERE username = '' OR 1 = 1 --' AND password = '';
```

---

# OWASP juice shopによる実践

ログイン画面におけるSQLインジェクションを実践する

* adminでログイン
* 他人のアカウントでログイン

(その場でデモを見せる)

---

# ソースコードから見るログインの脆弱性

* [ログイン処理をしているコード](https://github.com/juice-shop/juice-shop/blob/master/routes/login.ts)
* `/rest/user/login`のパスに対して`POST`メソッドでemailとパスワードをリクエストボディで送信している

---

# ソースコードから見るログインの脆弱性

- [実際のソースコードにおけるログイン認証を行うSQL文](https://github.com/juice-shop/juice-shop/blob/master/routes/login.ts#L36)


```ts
models.sequelize.query(
  `SELECT * FROM Users WHERE email = '${req.body.email || ''}'
 AND password = '${security.hash(req.body.password || '')}' 
 AND deletedAt IS NULL`, { model: UserModel, plain: true })
```

* Usersテーブルからemailとハッシュ化したパスワードの両方が一致するレコードがあるかどうかを確かめている
* 途中でemailやpassowordの変数をいれているが、SELECTからIS NULLのところまでは結局は単なる文字列

---

# ソースコードから見るログインの脆弱性(adminでログイン)

```ts
models.sequelize.query(
  `SELECT * FROM Users WHERE email = '${req.body.email || ''}'
 AND password = '${security.hash(req.body.password || '')}' 
 AND deletedAt IS NULL`, { model: UserModel, plain: true })
```

* ここで、`req.body.email`の値を`' OR TRUE--`とすると...
  * `SELECT * FROM USERS WHERE email = '' OR TRUE -- || ...`
  * `email = '' OR TRUE`は常に真
  * `--`により以降はコメントアウトと認識され、直前まででSQL文は解釈される
  * よって、このSQL文ではUsersテーブルの全レコードを取得するものとなる
* plainオプションをtrueに指定しているので、最終的な戻り値は最後のレコードの値を含む単一のJavaScriptオブジェクト
  * 今回はadminユーザーでログインすることができたので、該当したレコードがadminと思われる
---

# ソースコードから見るログインの脆弱性(benderでログイン)

```ts
models.sequelize.query(
  `SELECT * FROM Users WHERE email = '${req.body.email || ''}'
 AND password = '${security.hash(req.body.password || '')}' 
 AND deletedAt IS NULL`, { model: UserModel, plain: true })
```

* `req.body.email`の値を`bender@juice-sh.op'--`とすれば
  * 先ほどと同様に`--`以降はコメントアウトとして見なされる
  * よってパスワードが適当でもログインされる
* 任意のユーザーに対してメールアドレスが分かってしまえば、パスワードを知らずともログインできる

---

# 想定される被害

* adminユーザーとしてログインされることで、強い権限をもって不正な操作が行われる
* メールアドレスだけで不正にログインがされる
  * 登録した個人情報が漏れる
  * 勝手に商品を大量に注文される
* より一般的には
  * サービスの根幹であるデータベースを不正に操作されるため、あらゆる危険がある
    * データの削除、個人情報の流出etc

---

# SQLの構造と想定されるクエリ

|キーワード(予約語)|	SELECT FROM WHERE|
|---|---|
|演算子|=|
|識別子|name *|
|リテラル|'yamada'|

* 正しいSQLクエリを発行したい
  * リクエストボディとして送信したemailとパスワードはそれぞれ、WHEREの比較条件におけるそれぞれのフィールドの単なる文字列(リテラル)として扱いたい
* リテラル以外をユーザー側で弄れるような構造は基本的に危険

---

# 解決策1 入力データの検証とフィルタリング

* 一般的にウェブアプリケーションは、入力されたデータを検証し、期待される形式や範囲内の値かどうかを確認することで不正な値な入力値を受け付けないようにする
* 今回の入力(リクエストボディ)でいうと
  * 例えばemailはRFCに準拠するもの以外は弾くなど(3rdパーティのライブラリが大体ある)
    * ただしRFCに準拠しないものもあるので、一概に弾くのが良いとは限らない(ここはアプリの要件次第)

---

# 解決策2 プリペアードステートメントを利用する

* プリペアードステートメントとは
  * データベースに対して事前にSQL文のテンプレートを準備し、実行時にバインド変数に値をセットして実行する機能
  * バインド変数は、実行時に実際の値をセットすることができる
* ユーザーが入力した値をそのままSQL文に埋め込まないため、SQLインジェクション攻撃を防止することが可能

go言語での例

```go
// プリペアドステートメントを使用してSQL文を構築する
stmt, err := db.Prepare("SELECT * FROM users WHERE username = ? AND password = ?")
if err != nil {
  return err
}
defer stmt.Close()
// バインド変数に値をセットする
rows, err := stmt.Query(username, password)
```

---

# 3. エラーメッセージの制限

* 一般に、ウェブアプリケーションは、エラーメッセージが攻撃者に重要な情報を提供しないようにする必要がある
* 例えばSQLインジェクションの対策としては、エラーメッセージに生のSQLクエリを返さないようにする
  * 今回のログイン画面ではシステムエラーでSQLクエリがそのまま返ってきたことで解析が容易になった
* 入力データの検証とフィルタリングと合わせて考える
  * 想定される不正なリクエストを洗い出し、それに対応するエラーメッセージを返す

---

# SQLインジェクション対策を考える上で大事なこと

* 外部からの入力は基本的に信用できない
* 期待する入力と動作を定義する
  * 不正な入力は評価以前に弾く
    * REST APIであればクエリパラメータ、リクエストボディ等の適切なvalidation
  * 入力に対して期待する動作になるような実装になっているかを確認する
    * プリペアードステートメントの利用

---

# 参考資料

* [SQLインジェクション総”習”編](https://www.slideshare.net/yohgaki/sql-76168380)
* [攻撃から学ぶSQLインジェクション対策](https://qiita.com/risto24/items/0354a902faeb442cddcc)
* [詳解セキュリティコンテスト](https://www.amazon.co.jp/dp/4839973490)