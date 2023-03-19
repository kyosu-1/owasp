---
marp: true
footer: "2023/03/20"

---

# XSS with OWASP juice shop

---

# 今回やること

* XSSとは
* OWASP juice shopによる実践と対策法

---
# XSSとは

インジェクションの一種で、攻撃者が悪意のあるスクリプトをWebアプリケーション内で実行させることができる脆弱性

[参考](https://owasp.org/www-community/attacks/xss/)


---

# XSSの流れ

![](https://www.ipa.go.jp/files/000083715.png)

[参考](https://www.ipa.go.jp/security/vuln/websecurity-HTML-1_5.html)


---

# XSSの種類

XSSは主に3つのタイプに分類される

* 反射型XSS(Reflectd XSS)
  * ユーザからのリクエストに含まれるスクリプトをそのままレスポンスに出力してしまい発動するXSS。攻撃者は悪意のあるリンクを作成し、犠牲者にクリックさせる。
* 蓄積型XSS(Stored XSS)
  * 攻撃者が入力したスクリプトがWebアプリケーションに保存され、別のユーザがその保存箇所を読み込んだ際に保存されたスクリプトが発動するXSS
* DOM-based XSS
  * JavaScriptを使ってブラウザに表示しているHTMLを操作する場合に、攻撃者の用意した意図しないスクリプトが発動するXSS

---

# OWASP juice shopによる実践

検索におけるXSSの脆弱性を実践

* Bonus Payload
* DOM XSS

---

# Bonus Payload

商品検索で以下を入力する

 <iframe width="100%" height="166" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/771984076&color=%23ff5500&auto_play=true&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true"></iframe>


* iframeタグ
  * 指定したリンク先のページ内容をフレーム表示することができるHTMLタグのこと
  * [参考 iframeタグ](https://gmotech.jp/semlabo/seo/blog/html-ifame-tag/)

---

# DOM XSS

商品検索で以下を入力する

<iframe src="javascript:alert(`xss`)">

---

# 想定される被害

- 本物サイト上に偽のページが表示される
  - 偽情報の流布による混乱
  - フィッシング詐欺による重要情報の漏えい
- ブラウザが保存しているCookieを取得される
  - Cookie にセッションIDが格納されている場合、さらに利用者へのなりすましにつながる
  - Cookie に個人情報等が格納されている場合、その情報が漏えいする

[参考](https://www.ipa.go.jp/security/vuln/websecurity-HTML-1_5.html)


---

# ソースコードから見るXSSの脆弱性

* [該当するコード](https://github.com/juice-shop/juice-shop/blob/master/frontend/src/app/search-result/search-result.component.ts#L144-L165)

---

# ソースコードから見るXSSの脆弱性


```ts
  filterTable () {
    let queryParam: string = this.route.snapshot.queryParams.q
    if (queryParam) {
      queryParam = queryParam.trim()
      this.ngZone.runOutsideAngular(() => { // vuln-code-snippet hide-start
        this.io.socket().emit('verifyLocalXssChallenge', queryParam)
      }) // vuln-code-snippet hide-end
      this.dataSource.filter = queryParam.toLowerCase()
      this.searchValue = this.sanitizer.bypassSecurityTrustHtml(queryParam) // vuln-code-snippet vuln-line localXssChallenge xssBonusChallenge
```

* `this.searchValue = this.sanitizer.bypassSecurityTrustHtml(queryParam)`の部分で検索値を格納しているが、ここがXSSの原因と思われる
  * `bypassSecurityTrustHtml`関数によってユーザー入力がHTMLとして扱われ、動的に挿入された`<script>`タグ等がそのまま認識される。
    * この関数は開発者がそのHTMLコードが信頼できることを確信している場合にのみ使用すべき
    * [参考: angular-safe-html](https://webbibouroku.com/Blog/Article/angular-safe-html)
* XSSの種類としてはDom-based XSS

---

# 対策

* `this.searchValue = this.sanitizer.bypassSecurityTrustHtml(queryParam)`の部分を`this.searchValue = queryParam`とする
  * Angularのデフォルトのサニタイザー(`<script>`タグなどを無害化するもの)が適切に機能し、XSS攻撃を防ぐことができる
  * [参考](https://blog.lacolaco.net/2019/05/trusted-types-and-angular-security/)

---

# 一般的な対策法

1. 入力の検証とサニタイズ：ユーザー入力を適切に検証し、サニタイズすること。例えば、HTMLタグや特殊文字をエスケープすることで、スクリプトが実行されないようにする。
2. Content Security Policy（CSP）：CSPは、Webアプリケーションで実行されるスクリプトのソースを制限するためのセキュリティヘッダ。これにより、攻撃者が悪意のあるスクリプトを注入するのを防ぐことができる。
3. HTTPOnlyクッキー：HTTPOnly属性を持つクッキーは、クライアントサイドのJavaScriptからアクセスできないため、XSS攻撃によるクッキーの盗みが防ぐことができる。
4. セキュアなプログラミングプラクティスの遵守：Webアプリケーションを開発する際に、セキュリティに重点を置いたプログラミングプラクティスに従う。例えば、DOM操作には安全な関数を使用し、ユーザー入力を適切に処理するなど。

---

# 参考

- [Cross_Site_Scripting_Prevention_Cheat_Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [DOM_based_XSS_Prevention_Cheat_Sheet](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html)
- [IPA XSS](https://www.ipa.go.jp/security/vuln/websecurity-HTML-1_5.html)
- [dom-based-xss](https://www.ubsecure.jp/blog/dom-based-xss)

---