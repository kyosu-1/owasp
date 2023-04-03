---
marp: true
footer: "2023/04/03"

---

# 5. 識別と認証の失敗

# A07:2021 – Identification and Authentication Failures

---

# 今回やること

* 識別と認証の失敗(Identification and Authentication Failures)とは
* 想定される被害
* OWASP juice shopによる実践
* 一般的な原因
* 一般的な対応策

---

# 識別と認証の失敗(Identification and Authentication Failures)とは

* owasp top 10における`A07_2021-Identification_and_Authentication_Failures`
* 攻撃者が他のユーザーとして偽装したり、不正なアクセスや権限を取得できる可能性があるようなリスク
* アプリケーションがユーザーを適切に認証および識別できない場合に発生する

[参考: A07_2021-Identification_and_Authentication_Failures](https://owasp.org/Top10/A07_2021-Identification_and_Authentication_Failures/)

---

# 簡単な例

ユーザー認証を行うwebサイトにおいて、パスワードポリシーが4桁の整数値となっているとする。
このとき、攻撃者による高々1万回の総当たりによってパスワードがマッチし、不正なログインが行われる。

---

# 想定される被害

* アカウントの乗っ取り: 認証および識別の失敗が攻撃者によるアカウントの乗っ取りを容易にする。これにより、攻撃者がユーザーのプライバシーや個人情報にアクセスしたり、権限を悪用してシステム内で悪意のある活動を行う可能性がある。
* 機密情報の漏洩: 認証や識別の脆弱性を利用して不正アクセスを行った攻撃者は、機密データや企業情報にアクセスすることができる。これにより、企業の知的財産や顧客データ等が漏洩し、企業の評判やビジネスに重大な影響を与える可能性がある。
* システムへの不正アクセス: 認証および識別の失敗が、攻撃者によるシステムへの不正アクセスを許すことがある。攻撃者はシステム内で権限昇格を試みたり、システムの脆弱性を悪用してさらに深刻な攻撃を仕掛けることができる。

---

# OWASP juice shopによる実践

以下の脆弱性を実践

* Password Strength
  * adminユーザーのパスワードを予測しログインする
* Bjoern's Favorite Pet
  * bjoernのowaspアカウントのパスワードの再設定を秘密の質問に答えることで勝手に行う

---

# Password Strength

1. ログイン画面(`http://localhost:3000/#/login`)に遷移する
2. adminユーザーのメールアドレス(`admin@juice-sh.op`)を入力する。メールアドレスはカスタマーフィードバックのメールアドレスの形等から推測する
3. adminで使われていそうなパスワードを推測する。`admin123`が実際のパスワードで、これを入力することでadminユーザーでログインが出来る

---

# Password Strengthの原因と対応策

* 原因
  * デフォルトパスワードのような、簡単に推測されそうなパスワードを利用している
* 対応策
  * 推測されずらいパスワードを利用する
  * (adminとは別だが)パスワード登録においてvalidationロジックを追加する
    * 最低文字数の設定([最低8文字以上が推奨されている]([NIST Special Publication 800-63B](https://pages.nist.gov/800-63-3/sp800-63b.html)))
    * 極端に推測されやすいパスワード(`admin123`や`1111111`)の禁止
    * etc

---

# Bjoern's Favorite Pet

1. パスワードリセット画面(`http://localhost:3000/#/forgot-password`)に遷移する
2. bjoernのemail(`bjoern@owasp.org`)を入力する
3. `Security Question`として`Name of your favarite pet`と聞かれる。`Zaya`が質問の答えであり、入力を行う。
   * これはweb上での検索 or 総当たりによって分かる。
4. 適当な再設定用のパスワードを入力して再設定を行う

---

# Bjoern's Favoriteの原因と対策

* 原因
  * SNSでの発信や総当たりで特定されそうな質問による認証を行っている
  * [CWE640: Weak Password Recovery Mechanism for Forgotten Password](https://cwe.mitre.org/data/definitions/640.html)
* 対策
  * 一般に「秘密の質問」のようなものやり方はオンライン調査(Facebook、LinkedInなど)の影響を受けやすく、総当たり攻撃を受けやすいので唯一の方法として使用すべきでない。
  * メールでのトークン(パスワード再設定リンク)送信やSMSによるPINコードによる再設定の方法等のセキュアな仕組みにする。
  * [Forgot_Password_Cheat_Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html)

---

# 一般的な原因

* 弱いパスワードポリシー: アプリケーションが簡単に推測されるパスワードを許可したり、定期的なパスワード変更を要求しない場合、攻撃者がパスワードを推測することが容易になる。
* 不適切な認証機能: 例えば、マルチファクター認証（MFA）が実装されていない場合や、認証情報が適切に保護されていない場合、攻撃者が不正アクセスを試みる際に障壁が低くなる。
* セッション管理の欠陥: セッションIDが予測可能であったり、セッションの有効期限が設定されていなかったりすると、攻撃者がセッションをハイジャックすることが容易になる。
* 総当たり攻撃への対策が不十分: アプリケーションが連続した認証試行に対する制限や遅延を設けていない場合、攻撃者が繰り返し試行して認証情報を推測することができる。

---

# 一般的な対応策

* 強力なパスワードポリシーの導入
* マルチファクター認証（MFA）の実装
* 適切なセッション管理
* 総当たり攻撃対策の強化

---

# 強力なパスワードポリシーの導入

* 長さ、複雑さ、更新頻度などの基準を設定し、適切なパスワード管理を実施する
* 例えばパスワードの最小長は8文字以上とすることが望ましい
  * [NIST Special Publication 800-63B](https://pages.nist.gov/800-63-3/sp800-63b.html)

[参考：Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)

---

# マルチファクター認証（MFA）の実装

* 2つ以上の異なる認証要素を組み合わせることで、セキュリティを強化する

[参考：MFA Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html)

---

# 適切なセッション管理

* セッションIDの生成に安全な乱数生成器を使用し、予測不可能なセッションIDを生成する
  * uuid(128ビットの衝突確率が非常に低いID)が利用される場合が多いが、[RFC](https://www.ietf.org/rfc/rfc4122.txt)的にはセキュリティ要件を満たさないらしい
    * 「推測困難」の部分が実装に依存するので
* セッションの有効期限やタイムアウトを設定し、不正なセッション利用を防ぐ

[参考：Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)

---

# 総当たり攻撃対策の強化

* アカウントロックアウト、遅延、CAPTCHAなどの技術を使用して、連続した認証試行に対して制限を設ける
* また、MFA認証は総当たり攻撃に対する強力な防止策となる

[参考：Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)

