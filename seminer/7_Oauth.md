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

# 3つの代表的な認証

まずは代表的な以下の3つの方式に基づいた認証のフローを見ていく(認可はひとまず置いておく)

* Basic認証
* セッションストア方式
* トークン方式

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

* ステートレスなHTTP通信をステートフルにして、認証を維持させる方式

[RFC6265: HTTP State Management Mechanism](https://www.rfc-editor.org/rfc/rfc6265)

---

# セッションストア方式のフロー

![](https://storage.googleapis.com/zenn-user-upload/cbe792e9c14c-20221015.png)

[参考](https://zenn.dev/tanaka_takeru/articles/3fe82159a045f7)

---

# セッションストア方式のフロー

1. ログインによるユーザーの認証を経て、サーバー側でSessionIDを発行しCookieにセットする
  * セッションIDは一意であり、短期間で有効期限が切れる
2. SessionIDとそれに紐づくユーザー情報をサーバー側で持つ(RDBやKVSに保存)
3. 再度リクエストを投げた際にはリクエスト(Cookie)に含まれるセッションIDを確認

---

# セッションストア方式の特徴

* ステートフルで、クライアント(ユーザー)の状態を管理できる
* 脆弱性の観点
  * セッションIDが盗まれた場合、不正なアクセスが行われる可能性がある
    * 推測困難なものを生成する必要がある
  * [安全なウェブサイトの作り方 - 1.4 セッション管理の不備](https://www.ipa.go.jp/security/vuln/websecurity/session-management.html)


---

# トークン方式

* トークン（デジタルな証明書）を用いてユーザーの身元を確認し、セキュアなアクセスを提供する認証方法
  * トークンとしてはJWTが代表的
    * [RFC7519: JSON Web Token (JWT)](https://www.rfc-editor.org/rfc/rfc7519)
    * [jwt-auth](https://developer.mamezou-tech.com/blogs/2022/12/08/jwt-auth/)

---

# トークン方式のフロー

![](https://storage.googleapis.com/zenn-user-upload/dcc08ae4b06c-20221015.png)

[参考](https://zenn.dev/tanaka_takeru/articles/3fe82159a045f7)

---

# トークン方式のフロー

1. サーバーが認証情報を含むトークン（例：JWT）を生成し、クライアントに送信する
2. クライアントはこのトークンを保存し、以降のリクエストでトークンをサーバーに送信することで、ユーザーが認証済みであることを証明。
3. トークンは、有効期限や署名などの情報を含んでおり、サーバーはトークンを検証して認証状態を確認

---

# トークン方式とセッション方式の認証の比較

* 最大の違いはサーバー側で状態を持つかどうか
  * セッション方式はステートフルで、トークン方式はステートレス
  * この違いからトークン方式はセッション方式と比較して分散アプリケーションに強い
    * トークン方式は状態を持たないため、各サーバーで独立して検証を行える
      * セッション方式はセッションストアからデータを確認する必要あり
      * (スケールアウトするようなストアを利用すれば緩和)
  * 一方でトークンはサーバー側で情報を持っていないため、トークンは無効化することは難しい
    * セッションはログアウトによりサーバー側でストアから無効化できるが、トークンはあくまでクライアント側で棄却するのみ

[セッションベース認証とトークンベース認証の違い](https://zenn.dev/tanaka_takeru/articles/3fe82159a045f7)

---

# モノリスアプリケーションにおける認証・認可

* モノリスアプリケーションにおいてはリソースを提供するサーバーでセッションやトークンを発行
* 認可は基本的に認証に基づく
  * ユーザーが直接ブラウザ上から操作することのみを想定されたモノリスアプリケーションにおいては大体これで十分
  * 一方でマイクロサービスだと...

---

# モノリスからマイクロサービスへ

* 近年のwebアプリケーションにおいてはほとんどがAPIを通じた複数のサービス間通信を含むもの
  * ここでのマイクロサービスはgoogleのような巨大なものだけでなく、slackアプリやYoutube API等と一部連携するようなサービスも含む
* 認証・認可に関してもマイクロサービスに適したものを考える必要がある

---

# モノリスからマイクロサービスへ
![](https://ncdc.co.jp/wp-content/uploads/2020/02/%E3%83%9E%E3%82%A4%E3%82%AF%E3%83%AD%E3%82%B5%E3%83%BC%E3%83%92%E3%82%99%E3%82%B91.jpeg.webp)
[参考](https://ncdc.co.jp/columns/6503/)

---

# マイクロサービスにおける認証・認可の課題

マイクロサービスにおける認証・認可の設計にはモノリスなアプリケーションと比較して様々な課題がある。

* セキュリティの一貫性: 異なるマイクロサービス間で認証・認可の仕組みが統一されていない場合、セキュリティの一貫性が損なわれ、脆弱性が生じる可能性がある。
* セッション管理の複雑さ: マイクロサービス間でセッション情報を共有する必要がある場合、セッション管理が複雑になり、データの同期や共有の課題が生じる。
* サービス間の認可ポリシーの維持: マイクロサービスが増えることで、それぞれのサービス間の認可ポリシーを維持・管理することが難しくなる。認可ロジックの変更が必要になった際に、一貫性のある更新が困難になることがある。

---

# マイクロサービスの認証・認可に挑む

* 認可：**あるサービス(アプリケーション)から別の外部サービスを適切な認可により呼び出したい**
  * → Oauth2.0
* 認証：**複数のアプリケーション間で一貫したログイン情報を維持したい**
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
* [マイクロサービスの認証・認可とJWT](https://speakerdeck.com/oracle4engineer/authentication-and-authorization-in-microservices-and-jwt)
* [Microservices における認証と認可の設計パターン](https://please-sleep.cou929.nu/microservices-auth-design.html)