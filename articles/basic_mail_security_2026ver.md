---
title: "俺の考えた一番わかりやすいSPF/DKIM/DMARC解説（N番煎じ）"
emoji: "🎈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["spf", "dkim", "dmarc", "mail", "security"]
published: true
publication_name: robustengine
---

## そもそもなぜメール認証が必要なのか

メールは受信者側にとって「送信元が本物かどうか」を判断する仕組みがほとんどありません
その結果、以下のような **なりすまし・フィッシング・詐欺** が横行しています

- example.com を名乗った偽メールが届く
- 送信元を偽装してわざと存在しない宛先に送り，バウンスメールを大量に送りつける（Backscatter）
- 自社ドメインがスパム扱いになる

このような被害を防ぐために、**送信元の正当性や内容の整合性を検証する仕組み** が必要になりました
そこで登場するのが **SPF / DKIM / DMARC** です

## 登場人物（技術の役割）

| 仕組み       | 検証する対象                | 何を保証するか         |             |
| --------- | -------------------- | ------------- | ----------- |
| SPF   | 送信元の IP / エンベロープFrom(MAIL FROM) | このサーバーから送ってOK | https://datatracker.ietf.org/doc/html/rfc7208 <br/> https://tex2e.github.io/rfc-translater/html/rfc7208.html|
| DKIM  | メールへの電子署名            | 内容が改ざんされてないこと   |https://datatracker.ietf.org/doc/html/rfc6376 <br/> https://tex2e.github.io/rfc-translater/html/rfc6376.html |
| DMARC | SPF/DKIM + From の整合性 | ドメイン名の真正性     | https://datatracker.ietf.org/doc/html/rfc7489 <br/> https://tex2e.github.io/rfc-translater/html/rfc7489.html |


## 全体の流れ（受信時の挙動）

1. 受信サーバーが送信元 IP をチェック（SPF）
2. メールに署名があれば検証（DKIM）
3. From と SPF / DKIM の結果を突き合わせる（DMARC）
4. DMARC ポリシーに従って合否を判断・処理


## 各技術の仕組みと注意点

### SPF（Sender Policy Framework）

- やっていること:
    - DNS の TXT レコードに「うちのドメインはどの IP から送るよ」と宣言する仕組み。
- 例

```
example.com TXT "v=spf1 include:_spf.google.com ~all"
example.com. TXT "v=spf1 ip4:203.0.113.10 -all"
example.com. TXT "v=spf1 ip4:203.0.113.10 ip4:198.51.100.0/24 -all"

# さらなる具体例は以下
```

:::details 設定例

### IPv4 を許可

```
example.com. TXT "v=spf1 ip4:203.0.113.10 -all"
```

* `203.0.113.10` からの送信を許可
* それ以外は **すべて拒否**

### IPv4 複数

```
example.com. TXT "v=spf1 ip4:203.0.113.10 ip4:198.51.100.0/24 -all"
```


### IPv6 を許可

```
example.com. TXT "v=spf1 ip6:2001:db8::/32 -all"
```


### IPv4 + IPv6 混在

```
example.com. TXT "v=spf1 ip4:203.0.113.10 ip6:2001:db8::/32 -all"
```

---

### include（外部メールサービスを使う場合）

#### Google Workspace

```
example.com. TXT "v=spf1 include:_spf.google.com -all"
```

#### Amazon SES

```
example.com. TXT "v=spf1 include:amazonses.com -all"
```

#### SendGrid

```
example.com. TXT "v=spf1 include:sendgrid.net -all"
```

**include = 相手ドメインのSPFを丸ごと信頼**


### a（AレコードのIPを許可）

```
example.com. TXT "v=spf1 a -all"
```

* `example.com` の A/AAAA レコードのIPからの送信を許可


### mx（MXレコードのIPを許可）

```
example.com. TXT "v=spf1 mx -all"
```

* `example.com` の **メール受信サーバーIP** からの送信を許可

---

## IP + ドメイン混在

```
example.com. TXT "v=spf1 ip4:203.0.113.10 include:_spf.google.com include:amazonses.com -all"
```

* 自社サーバー
* Google Workspace
* Amazon SES
  をすべて許可


:::

- ポイント
    - 受信サーバーは *ENVELOPE FROM*（Return-Path）のドメインを参照して SPF 判定
    - 設定ミス（正規送信元を列挙漏れ）すると正規送信なのに fail になることがある
    - TXTレコードで設定する．SPFレコードは一つしか効果がない．複数レコード設定すると無視される．


### DKIM（DomainKeys Identified Mail）

- やっていること
    - 送信ドメインがメールに秘密鍵で **電子署名** を付け、受信側が DNS 公開鍵で検証する仕組み。
- メール本文やヘッダが途中で改ざんされていないことを保証
- 署名に使われる DKIM ドメイン（d=）が Header-From と一致しているかが DMARC で重要

#### DNSレコードの例

```
# TXTの場合
selector1._domainkey.example.com. TXT "v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEExamplePublicKeyHere"

# CNAMEの場合
abcdefg1234567._domainkey.example.com. CNAME abcdefg1234567.dkim.amazonses.com
```

#### メールヘッダの例
```
DKIM-Signature:
 v=1;
 a=rsa-sha256;
 c=relaxed/relaxed;
 d=example.com;
 s=selector1;
 h=from:to:subject:date;
 bh=abc123Base64Hash;
 b=def456Base64Signature
```

- メールヘッダにある，DKIM-Signatureを受信サーバーは確認する
- `d=`と`s=`をみて，`selector1._domainkey.example.com` のDNSレコードを取得し，公開鍵で署名を検証


### DMARC（Domain-based Message Authentication, Reporting & Conformance）

- SPF / DKIM の結果を踏まえ、**From のドメイン整合性（alignment）** もチェックして最終判断する仕組み
- SPF や DKIM が **pass** でも、From と一致していなければ **DMARC は pass にならない**
- p=none / quarantine / reject のポリシーを受信側に指示できる
- レポートによって失敗状況を可視化できる
- 例
```
_dmarc.example.com. TXT "v=DMARC1; p=none; rua=mailto:dmarc-rua@example.com; ruf=mailto:dmarc-ruf@example.com; fo=1"

_dmarc.example.com. TXT "v=DMARC1; p=quarantine; pct=25; rua=mailto:dmarc-rua@example.com; ruf=mailto:dmarc-ruf@example.com"

_dmarc.example.com. TXT "v=DMARC1; p=reject; rua=mailto:dmarc-rua@example.com; ruf=mailto:dmarc-ruf@example.com; adkim=r; aspf=r"
```

- ruf は「失敗時の詳細レポート」なので、`p=none` かつ `fo` 未指定だと送られないケースが多い
- alignment には strict(s) / relaxed(r) があり、 relaxed の場合は「サブドメイン一致」でも PASS になる
    - `adkim=`, `aspf=` とした箇所


## 仕組みを式で表すと

```
( SPF=PASS AND SPFドメイン=Fromドメイン )
 OR
( DKIM=PASS AND DKIM署名のドメイン=Fromドメイン )
 → DMARC=PASS
```

## 困ったときの確認手順
0. (optional) もしDMARCレポートが受け取れていない場合，DMARCレコードのrua, rufを指定して受け取れるようにする
1. 自分自身にメールを送ってみる
    - 送信元サービスなどが複数ある場合，その全てから送信する
2. Gmailの場合，「原文を表示」してみる
3. SPF, DKIM, DMARCの`PASS` / `FAIL` を確認する
    - 実務的にはこのタイミングで表（送信元サービス，SPF, DKIM, DMARC）に整理するとよい
4. SPFが`FAIL`の場合，SPFレコードの設定を更新する必要がある
5. DKIMが`FAIL`の場合，DKIMレコードの設定を更新する必要がある
6. DMARCが`FAIL`の場合，DMARCレコードの設定 or SPFかDKIMとのalignmentを一致させる必要がある
    - SPFのENVELOPE-FROM/MAIL-FROM と，Header-FROMの一致
    - DKIMのメールヘッダ`d=`（DNSドメイン）と，Header-Fromの一致
7. DMARC含めて全てPASSするようになったら，DMARCのpolicy(`p=`)を段階的に引き上げる



# 参考文献

- https://zenn.dev/a24k/articles/20220605-dmarc-postmark "ご家庭用の DMARC 運用を考える"
- https://zenn.dev/segawa/articles/15ddf86c68214b "SPF/DKIM/DMARCを3行で分かったかのようなつもりになる"
- https://zenn.dev/hori/scraps/2a3e02e7ce6a99 "SPF, DKIM, DMARC設定について、わからないことを調べる"
- https://zenn.dev/tamasan/articles/8da97f551c765e "独自ドメインでメールするために、DKIM/SPF/DMARCのお勉強"
