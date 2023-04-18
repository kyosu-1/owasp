---
marp: true
footer: "2023/04/18"

---

# owasp juice shopの認証・認可に関して

---

# アプリのアーキテクチャ

![](https://pwning.owasp-juice.shop/introduction/img/architecture-diagram.png)

1. SPA(シングルページアプリケーション)
2. サーバーはモノリス(単一)
3. Googleアカウントによるログインをサポート(Oauth2.0)

---

# 認証・認可

* jwtトークンによる認証
* 認可の制御はjwtトークンのpayloadのプロパティであるroleを見ている

---

# owasp juice shopの認証


1. ユーザーがログインフォームにメールアドレスとパスワードを入力し、ログインリクエストを送信する。
2. サーバーは入力されたメールアドレスとパスワードを確認し、正しい場合はJWTを生成し、クライアントに返す。
3. クライアントはJWTをローカルストレージに保存し、以降のリクエストでこのトークンをHTTPヘッダのAuthorizationフィールドに設定して送信する。
4. サーバーは、受信したJWTを検証し、正当なトークンである場合はリクエストを処理する。これにより、ユーザーの認証状態が維持される

[login](https://github.com/juice-shop/juice-shop/blob/master/routes/login.ts)
[oauth](https://github.com/juice-shop/juice-shop/blob/master/frontend/src/app/oauth/oauth.component.ts)

---

# owasp juice shopの認可

jwtトークンのpayloadの中にroleのプロパティがあり、そちらで判別してる

[ロール部分のコード](https://github.com/juice-shop/juice-shop/blob/9a0789b5ecb4ee76fe528b1860095e945f6302ac/lib/insecurity.ts#L156)

---

# 脆弱性

* ログインの脆弱性 → SQLインジェクションが起こりうる
* Oauthの脆弱性 → ？

# 

https://github.com/juice-shop/juice-shop/blob/9a0789b5ecb4ee76fe528b1860095e945f6302ac/frontend/src/app/login/login.component.ts