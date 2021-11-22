---
title: "freezedでJsonSerializableのオプションを使いたいときには"
emoji: "❄️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["freezed", "json", "flutter", "dart", "serialize"]
published: true
---

# 背景
- [freezed](https://github.com/rrousselGit/freezed)を使っている
- dart/flutterの中では`camelCase`, `PascalCase` で命名を統一したい
- しかしAPIとのやりとりでは，jsonのkeyを `snake_case` にする必要があった
- 候補としては以下のようなものを考えた
  - [dioのinterceptorの仕組み](https://pub.dev/packages/dio#interceptors)と[recase](https://pub.dev/packages/recase)の組み合わせ
  - [json_serializable](https://pub.dev/packages/json_serializable)を使う
- そもそも[freezedはJsonSerializableと組み合わせて使うこともできる](https://github.com/rrousselGit/freezed#fromjsontojson)はず
- オプション渡す方法がよくわからない(明示的にドキュメントになっていない)
  - 作者が言及している[関連してそうなissue](https://github.com/rrousselGit/freezed/issues/50)の書き方を真似る
  - できた！！

# 結論
- [freezedではすべてのJsonSerializableのオプションがサポートされたはず](https://github.com/rrousselGit/freezed/issues/49)
- しかし, こう書くとエラー(生成されたファイルを確認すると，重複して生成されてしまっていた)
```dart
@freezed
@JsonSerializable(fieldRename: FieldRename.snake)
class Example with _$Example {
  factory Example() = GeneratedClass;
}
```
- なのでこう書き換えるとうまくいきます
```dart
@freezed
class Example with _$Example {
  @JsonSerializable(fieldRename: FieldRename.snake)
  factory Example() = GeneratedClass;
}
```

# 参考になった方へ
- LIKE押してってください！
