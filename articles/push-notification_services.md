---
title: "アプリにおけるプッシュ通知の技術選定"
emoji: "🎈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['firebase', 'ios', 'android']
published: true
---

## 前提
- iOS, Android, webへpush通知を送りたい

## 結論
- firebase cloud messaging(FCM) 一択である

## 根拠
- FCMを利用すれば，iOS/Android両方がカバーでき，性能，料金いずれの面でも申し分ない
    - 性能面根拠資料
        - [アップストリーム メッセージの制限](https://firebase.google.com/docs/cloud-messaging/concept-options?hl=ja#upstream_throttling)
        - [トピック メッセージの制限](https://firebase.google.com/docs/cloud-messaging/concept-options?hl=ja#topics_throttling)
        - [ファンアウトのスロットル処理](https://firebase.google.com/docs/cloud-messaging/concept-options?hl=ja#fanout_throttling)
    - 料金面根拠資料
        - [無料(2022/03時点)](https://firebase.google.com/pricing?hl=ja)
- 利用する側はpush通知したい端末がどちらのOSなのかを意識する必要がない
- firebaseが発行するデバイスを一意に特定するidを保持しておくだけでよい
- そもそもAndroidにpush通知を送るためには使わざるを得ない
- 各種SDKが揃っている
    - rubyにはSDKがないが，FCMに限れば[fcmpush](https://github.com/miyataka/fcmpush)がある

## 比較対象
- APNS
    - [APNsとは？設定と実装方法を解説！｜Repro Journal （リプロジャーナル）](https://repro.io/contents/ios-remote-push-notifications-in-a-nutshell/)
- AWS SNS
    - [Amazon SNS とは - Amazon Simple Notification Service](https://docs.aws.amazon.com/ja_jp/sns/latest/dg/welcome.html)

## その他
- ディスクロージャーとして，fcmpushは筆者作成のライブラリであることを付記しておく
    - しかし2022/03時点でrubyもしくはrailsからfcmを扱うライブラリでは一番よいと自信を持って言える
    - 詳細は[fcmpush](https://github.com/miyataka/fcmpush)のREADMEを参照のこと
- スマートフォンの次の端末がきた際の，firebaseはなにか？を考える必要がある
