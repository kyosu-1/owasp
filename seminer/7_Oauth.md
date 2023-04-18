---
marp: true
footer: "2023/04/18"

---

# 認証・認可に関する話

---

# やること

* 認証・認可とは
* Basic認証
* セッションストア方式
* トークン方式
* モノリスからマイクロサービスへ
* Oauth2.0
* Open ID Connect

---

# 認証・認可とは

* 認証(Authentication)
  * **通信の相手が誰（何）であるかを確認すること**
  * 知識、所有、生体認証といった三要素のいずれかまたは複合して利用
* 認可(Authorization)
  * **とある特定の条件に対して、リソースアクセスの権限を与えること**
  * **それを持っている**ことによって**何か（リソースアクセス）**が**許可**される

---

# キーワード

* Basic認証
* セッションストア方式
* トークン方式
* Oauth2.0
* Open ID Connect
* SSO(Single Sign On)

---

# Basic認証とは

* HTTPで定義される認証方式の一つで、もっとも単純な実装方式
* Authorizationヘッダに`[ID]:[PASSWORD]`の形でBase64エンコードしたものを入れてリクエストを送る

---
# Basic認証のフロー

![](https://camo.qiitausercontent.com/5dd9f79328db0f49df2f8a8d2618d8dd296ca634/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3931323938322f31363663336236352d303461312d386263382d656165662d6133653432323234616466372e706e67)

[参考](https://qiita.com/hitomik/items/38befaf0d740c4b07363)

---

# Basic認証の特徴

* ステートレスな認証方式
  * 一度認証したとしてもその認証状態を維持しない
* ECサイトなどの一般的なwebサービスで利用されることはないが、ページの限定公開を簡易的に行うことに使われることがある
* Basic認証は平文でのパスワード送信を伴うので、HTTP通信では安全ではない(最低限HTTPS化することが望ましい)
  * [RFC7617](https://datatracker.ietf.org/doc/html/rfc7617)より

---

# セッションストア方式

* SessionIDに紐づくユーザー情報をサーバー側で持つ
* ログインが成功したユーザーが、SessionIDとそれに紐づくデータを作成しRDBやKVSに保存
* 

---

# セッションストア方式のフロー

![](https://storage.googleapis.com/zenn-user-upload/cbe792e9c14c-20221015.png)

[参考](https://zenn.dev/tanaka_takeru/articles/3fe82159a045f7)

---

# セッションストア方式の特徴

* 
* 
* 


---

# トークン方式

* サーバーが認証情報を含むトークン（例：JWT）を生成し、クライアントに送信します。
* クライアントはこのトークンを保存し、以降のリクエストでトークンをサーバーに送信することで、ユーザーが認証済みであることを証明します。
* トークンは、有効期限や署名などの情報を含んでおり、サーバーはトークンを検証して認証状態を確認

---

# トークン方式のフロー

![](https://storage.googleapis.com/zenn-user-upload/dcc08ae4b06c-20221015.png)

[参考](https://zenn.dev/tanaka_takeru/articles/3fe82159a045f7)

---

# トークン方式とセッション方式の比較

* 最大の違いはサーバー側で状態を持つかどうか
  * セッション方式はステートフルで、トークン方式はステートレス
* セッション方式
  * pros
* 

---

# モノリスアプリケーションにおける認証・認可

* モノリスアプリケーションにおいてはリソースを提供するサーバーでセッションやトークンを発行
* 認可は基本的に認証に基づく
  * ユーザーが直接ブラウザ上から操作することのみを想定されたモノリスアプリケーションにおいては大体これで十分
  * 一方でマイクロサービスだと...

(セッション方式とトークン方式はマイクロサービス)

---

# モノリスからマイクロサービスへ

* 近年のwebアプリケーションにおいてはほとんどがAPIを通じた複数のサービス間通信を含むもの
  * ここでのマイクロサービスはgoogleのような巨大なものだけでなく、slackアプリやYoutube API等と一部連携するようなサービスも含む
* マイクロサービスにおいては
  * サービス間通信の

---

# モノリスからマイクロサービスへ
![](https://ncdc.co.jp/wp-content/uploads/2020/02/%E3%83%9E%E3%82%A4%E3%82%AF%E3%83%AD%E3%82%B5%E3%83%BC%E3%83%92%E3%82%99%E3%82%B91.jpeg.webp)
[参考](https://ncdc.co.jp/columns/6503/)

---

# マイクロサービスにおける認証・認可の課題


* **あるサービス(アプリケーション)から別の外部サービスを適切な認可により呼び出したい**
  * Oauth2.0
* **複数のアプリケーション間で同じ情報を用いてログインを行うことをしたい**
  * Open ID Connect, SSO

---

# Oauth2.0とOpen ID Connect




---

参考資料まとめ

* [セッションベース認証とトークンベース認証の違い](https://zenn.dev/tanaka_takeru/articles/3fe82159a045f7)
* [RFC6749](https://datatracker.ietf.org/doc/html/rfc6749)
* [MDN HTTP Authentication](https://developer.mozilla.org/ja/docs/Web/HTTP/Authentication)
* [OAuth 2.0 全フローの図解と動画](https://qiita.com/TakahikoKawasaki/items/200951e5b5929f840a1f)
* [一番分かりやすいOauthの仕組み](https://qiita.com/TakahikoKawasaki/items/e37caf50776e00e733be)
* [Authentication_Cheat_Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)