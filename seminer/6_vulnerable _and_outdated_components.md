---
marp: true
footer: "2023/04/11"

---

# 6. 脆弱で古くなったコンポ―ネント

# A06:2021 – Vulnerable and Outdated Components

---

# 今回やること

* 脆弱で古くなったコンポーネント(Vulnerable and Outdated Components)とは
* アプリケーションの構成
* 想定されえる被害
* 脆弱性の事例
* OWASP juice shopによる実践
* 一般的な原因
* 一般的な対応策

---

# 脆弱で古くなったコンポーネント(Vulnerable and Outdated Components)とは

* ウェブアプリケーションやシステムの一部であるライブラリ、フレームワーク、その他のソフトウェアモジュールが、既知のセキュリティ脆弱性を持っていたり、バージョンが古くなっていたりすることを指す
* Avg Incidence Rate（平均発生率）が8.77%と他と比較して高め(他のtop 10のものは大体2～5％ぐらい)

[A06_2021-Vulnerable_and_Outdated_Components](https://owasp.org/Top10/A06_2021-Vulnerable_and_Outdated_Components/)

---

# アプリケーション・ソフトウェアの構成

* アプリケーションやソフトウェアは多数のコンポーネントから構成されている
  * オープンソースライブラリ
  * フレームワーク
  * API
  * etc
* 特に近年はOSSが充実し、自分自身が直接実装する部分よりもライブラリを用いて実装する部分の方が多い
  * アプリケーション固有のドメインロジック以外はほとんど共通化して利用できるため
* 大きなライブラリはまた別のライブラリを利用(依存)している

---

# 想定される被害

被害の内容はコンポーネントの持つ脆弱性の種類によって様々です。

* 情報漏洩
* プライバシー侵害
* システムへの不正アクセス
* データ改ざんや破壊

例えばdom操作における脆弱性を持つライブラリを使用していた場合には[DOM_Based_XSS](https://owasp.org/www-community/attacks/DOM_Based_XSS)といったようなインジェクションが起こりえる(この脆弱性はOSSやサードパッケージのものに起因する)

---

# 脆弱性の事例

多くの事例がニュースとなっている。ここでは以下の二つの事例をピックアップして紹介する。

* Equifaxデータ侵害(2017年)
* Heartbleedバグ (2014年)

---

# 脆弱性の事例1 Equifaxデータ侵害(2017年)
**概要**
クレジット報告機関であるEquifaxでは、古いApache Strutsの脆弱性を利用した攻撃により、約1億4700万人分の個人情報が盗まれた。([A03:2021 - Cryptographic Failures（暗号化の失敗）](https://owasp.org/Top10/A02_2021-Cryptographic_Failures/)とも関連してる)
**原因**
OSSのwebフレームワークであるApache Struts2に存在した脆弱性([CVE-2017-5638](https://nvd.nist.gov/vuln/detail/cve-2017-5638))が原因。この脆弱性は、攻撃者がリモートで任意のコードを実行できるもの。
修正パッチがリリースされていたにもかかわらず、適用を行っていたかったことにより脆弱性を付かれた。

**参考**
[piyolog](https://piyolog.hatenadiary.jp/entry/20170307/1488907259)
[Apache Struts2の脆弱性対策(IPA)](https://www.ipa.go.jp/security/vuln/oss/struts2_list.html)

---

# 脆弱性の事例2 Heartbleedバグ (2014年)
**概要**
Heartbleedバグは、2014年に発覚したOpenSSLの重大な脆弱性。([SSLとは](https://ds-b.jp/media/pages/200/))
この脆弱性は、攻撃者が暗号化された通信を傍受し、秘密鍵やユーザー情報などの機密データにアクセスすることを可能になり、多くのウェブサイトやオンラインサービスに影響を与え、ユーザーのプライバシーやセキュリティを脅かす状況が発生。

**原因**
Heartbleedバグは、OpenSSLのTLS/DTLSハートビート拡張機能に関連する脆弱性([CVE-2014-0160](https://nvd.nist.gov/vuln/detail/cve-2014-0160))
悪用すればシステムのシステムのメモリ上の情報を任意に閲覧することができてしまうとされる

**参考**
[piyolog](https://piyolog.hatenadiary.jp/entry/20140410/1397139257)
[OpenSSLの脆弱性と対策](https://siteguard.jp-secure.com/blog/open-ssl)

---

# その他の事例

* [jQuery File Uploadの脆弱性](https://jp.tenable.com/plugins/nessus/118310)
  * ファイルアップロードの脆弱性
  * 攻撃者は任意のファイルをサーバーにアップロードし、リモートコード実行を行うことができた
* [シェルショック脆弱性(2014年)](https://ja.wikipedia.org/wiki/2014%E5%B9%B4%E3%82%B7%E3%82%A7%E3%83%AB%E3%82%B7%E3%83%A7%E3%83%83%E3%82%AF%E8%84%86%E5%BC%B1%E6%80%A7)
  * Bashシェルに存在する脆弱性（[CVE-2014-6271](https://nvd.nist.gov/vuln/detail/cve-2014-6271)）が悪用され、攻撃者がリモートで任意のコードを実行できるリスクがあった。
* [WannaCryランサムウェア (2017年)](https://www.kaspersky.co.jp/resource-center/threats/ransomware-wannacry)
  * 古いWindowsシステムのSMBv1プロトコルに存在する脆弱性（MS17-010）を悪用し、ランサムウェアが広まった。

最新のセキュリティニュースを見ても多くの事例が共有されており、日々パッチ更新の呼び掛けがされている

---

# OWASP juice shopによる実践

* Legacy Typosquatting
  * 旧versionのジュースショップであったtyposquattingの事例を特定し、フォームで報告する
    * [typosquattingとは](https://office110.jp/security/knowledge/cyber-attack/typosquatting)
      * ありがちな入力ミスを利用し、正規の名前を少し変えたような、悪意のあるwebサイトやライブラリを用意すること
  * 利用ライブラリの情報を知るためにSensitive Data Exposureの課題であるAccess a developer's forgotten backup fileを事前に解く


---

# Access a developer's forgotten backup file

アプリケーションの設定ファイルをダウンロードする。今回はnode.jsのアプリなので、対象はpackage.jsonファイル。

1. http://localhost:3000/ftp/ にアクセスすると、ftpディレクトリが見れる
2. `package.json.bak`があることが分かる。ダウンロードしたいが、直接`http://localhost:3000/ftp/package.json.bak`にアクセスる宇都403エラー(.mdと.pdfしかダウンロードが許可されていない)
3. ヌルバイト攻撃によってアクセスする。`http://localhost:3000/ftp/package.json.bak%2500.md`でアクセスするとダウンロードが出来る(%2500は%00をurlエンコードしたもの)

---

# Legacy Typosquatting

4. 先ほどダウンロードしたjsonファイルの中で`dependencies`のvalueのリストが利用ライブラリの一覧。この中で、`eplogue-js`がTyposquattingのものにあたる(上から順に調べる or ツールを利用)
5. Customer Feedbackで`eplogue-js`を報告する

[package.json](https://github.com/juice-shop/juice-shop/blob/master/ftp/package.json.bak)
[epilogue-js](https://github.com/bkimminich/epilogue-js)

---

# 一般的な原因

* 未更新のコンポーネント: 古いバージョンのライブラリやフレームワークを使用している場合、新たに発見された脆弱性に対して脆弱になる。
* 不適切なコンポーネント選択: セキュリティが十分に検討されていないコンポーネントを選択すると、リスクが高まる。
* 依存関係の管理不備: アプリケーションの依存関係が適切に管理されていない場合、脆弱性が見逃される可能性がある。
* セキュリティ情報の不足: 開発者が最新のセキュリティ情報にアクセスできない場合、適切な対策が取れない可能性がある。

---

# 一般的な対応策1

* 依存関係の監査: アプリケーションの依存関係を定期的に監査し、脆弱性のあるコンポーネントを特定する。
* コンポーネントの更新: 脆弱性が報告されたコンポーネントや古いバージョンのコンポーネントを最新の安全なバージョンに更新する。
* セキュリティ情報の追跡: 使用しているコンポーネントのセキュリティ情報を追跡し、問題が発覚した場合に迅速に対策を講じる。

---

# 一般的な対応策2

* セキュアなコンポーネントの選択: セキュリティが検討されたコンポーネントを選択し、開発プロセスに組み込む。
* パッチ管理とデプロイメント: パッチやアップデートを迅速かつ効果的にデプロイするプロセスを整備する。
* ベストプラクティスとツールの活用: 自動化された脆弱性スキャンツールや依存関係管理ツールを活用し、開発プロセスに組み込むことでセキュリティを向上させる。

---

# 原因と対応策のまとめ

* 原因
  * 

* 対応策としては
  * 
  * パッチやアップデートの自動化

---

# ライブラリ管理



---

# 脆弱性診断ツール

この脆弱性は検知と対応の自動化が比較的しやすい

* 対処法としては

[Vuls]()


---

# githubを利用した対策



[]()

---