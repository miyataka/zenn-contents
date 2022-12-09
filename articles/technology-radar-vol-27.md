---
title: "テクノロジーレーダーをざっと読んだので紹介"
emoji: "🎈"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["thoughtworks"]
published: false
---

これは[カンム Advent Calendar 2022](https://adventar.org/calendars/7789)の9日目の記事です．
前回は[Yoshinori Okadaさんの東京から滋賀移住して1年経って思うこと](https://note.com/okkamon/n/n54e8a343d9f3)でした．琵琶湖の写真がよすぎました．
今日は一転，技術雑談的な記事です


## テクノロジーレーダーとは
- ThoughtWorksが毎年だしている新しめのテクノロジーへの評価一覧のようなもの
- 12/7時点での最新は vol.27 です．年に数回更新されます．
    - [こちら](https://www.thoughtworks.com/radar)からダウンロードできます
- ThoughtWorksは「リファクタリング」という本で有名なマーティンファウラー先生が所属しています

## 読むとなにが嬉しいの？
- どんな新しい技術がでてきているのかをざっくり追うことができ、しかもそれはある程度信頼のおけるレビューコメントつきである．
    - とはいえ、英語です。日本語を常用する我々（誰？）には言語的にも心理的にもハードル高めなので、一緒に読んでいきましょう。独断で気になったものについてとりあげていきます
    - 意訳多めです．間違ってるぜ！という点を見つけたら教えてくださると助かります．
    - 門外漢な部分にもコメントしているので，誤解だよ！って場合にも教えてもらえると助かります

## テクノロジーレーダーの読み方
quadrantsと呼ばれる4つの分野について分類されています．
以下のようになっています．ちなみに個人的には読む時にはあまり気にしていません．

- 4象限 (quadrants)
    - テクニック (Techniques)
    - プラットフォーム (Platform)
    - ツール (Tools)
    - 言語とフレームワーク (Languages and Frameworks)

もう一点，リングと呼ばれる各テクノロジーに対するThoughtWorks社の評価/スタンスといったものを簡潔に表したものです
- リング (ring)
    - 採用 (Adopt)
    - トライアル (Trial)
    - 評価 (Assess)
    - 待ち (Hold)

![](/images/techradar-at-a-glance.png)
画像はPDFより引用

## 気になったもの
ここからは気になったものを取り上げていきます．必ずしもポジティブとは限りません．

### Techniques
#### Path-to-production mapping
- Adopt
- ホワイトボードなどを用いながら，チームでプロダクト（やその機能）が実際に本番環境にデプロイされるまでのステップをより詳細に深堀ったり，問題点を把握したりするワークショップのようなアクティビティのこと．
- https://caroli.org/en/path-to-production/
- https://en.wikipedia.org/wiki/Value-stream_mapping
- 所感
    - すぐにでもやれそうなプラクティスが最初にでてきて嬉しい．技術的な目新しさがあるわけではないが，実際できてるかと言われるとそうではないので，やってみるとよさそう．
#### [Threat modeling](https://owasp.org/www-community/Threat_Modeling)
- Adopt
- システムとその環境をセキュリティ観点から評価・整理するためのプロセス
- 所感
    - 結構体系だった整理がされていて，セキュリティを考える際にかなり活用できそうな印象をうけた
- https://wiki.owasp.org/index.php/Category:Threat_Modeling
- https://www.microsoft.com/en-us/securityengineering/sdl/threatmodeling

#### [SBOM](https://www.nist.gov/itl/executive-order-14028-improving-nations-cybersecurity/software-security-supply-chains-software-1)
- Trial
- 脆弱性を管理するために，チームが依存しているライブラリを`machine-readable`な形で管理しましょうというもの
- 所感
    - こちらもセキュリティ文脈やPCIDSS文脈でよく聞く単語がランクインしていた．

#### SPA by default
- Hold
- Single Page Applicationをデフォルトの選択肢とすること
- 所感
    - トレードオフをちゃんと分析してからにしようね，と言っていた．そうだね...
#### Superficial cloud native
- Hold
- 直訳すると，表層的なクラウドネイティブ
- cloud-nativeを設計としてではなく，クラウドベンダー固有のサービスを選択することだと勘違いするんじゃない，という指摘
- 所感
    - それは本当にそう


### Platform
Platformカテゴリは，商用製品も多く，気軽に試せるものではないが，いざ導入するときに，選択肢を頭にいれておけるといいよね，くらい．
ではみていきます．

#### [Backstage](https://backstage.io/)
- Adopt
- https://github.com/backstage/backstage
- 所感
    - Spotifyが開発しているDeveloperPortalらしい，現在のチーム規模ではやりすぎ感あるものの，便利なものがあるものですね．
    - こういうのがでてくるあたり，ThoughtWorksは大企業を相手に仕事をやっているんだなぁと
#### [Colima](https://github.com/abiosoft/colima)
- Trial
- docker desktopを代替するOSS
- 所感
    - 個人的には現時点でdockerを代替するとは思っていない．
    - そもそもdockerはパラダイムシフトを大幅に加速した企業としてもっと儲けてくれてよいと思っている．
#### [eBPF](https://ebpf.io/)
- Trial
- extended Berkeley Packet Filter
- linuxからsocketのIOに対して，hookを仕込めるようなイメージ
- 所感
    - ロゴがかわいい！これは人気でる！
    - 個人的にネットワーク周りの技術に興味があり，知っていたので新情報はなかったが，とりあげられていると嬉しい
#### [Bun](https://github.com/oven-sh/bun)
- Assess
- 新しめのZigで書かれたJavaScript runtime
- 所感
    - Zig！今年一時期盛り上がりましたね．`better C` となるんだろうか

#### [Dragonfly](https://github.com/dragonflydb/dragonfly)
- Assess
- redis/memcached互換という意味不明なことを実現していて，かつパフォーマンスが恐ろしくよいインメモリDBらしい
- 所感
    - 前評判通りなら，いずれ置き換えていくのだろうが，本番投入時にはクラスタ構成とれるのか，とかが論点になりそう
#### [OrioleDB](https://github.com/orioledb/orioledb/)
- Assess
- PostgreSQLの辛いところを解決するぜ！というPostgresの新しいストレージエンジン.
- 所感
    - Posgresユーザーなので，よくなってくれると嬉しい


### Tools
#### [Great Expectations](https://docs.greatexpectations.io/docs/)
- Adopt
- データパイプラインを流れるデータのqualityチェックをするツール
- 所感
    - 継続的データクオリティチェックといったところか，たしかによさそう
#### [k6](https://k6.io/)
- Adopt
- jsで簡単にかける負荷テストツール
- 所感
    - これはよさそう．負荷テスト時はもちろん，CIプラットフォームの対応も結構進んでおり，
#### [spectral](https://stoplight.io/open-source/spectral/)
- Trial
- OpenAPIなどのjson/yamlに対して，lintできるツール．
- 所感
    - すごく便利そう．
#### [Clasp](https://github.com/google/clasp)
- GASがtypescriptで書けるぞ！！CLIですぐさまuploadできるぞ！というもの
- 所感
    - どうしても個人の権限に紐づいてしまうので，そもそもGASを避けたいが， 人類はスプレッドシートからは逃れられないんや...という気持ち
#### Online services for formatting or parsing code
- Hold
- 容易にセキュリティやプライバシーの問題を引き起こすのでやめとけ，とのこと
- 所感
    - そうだね
    - タダより高いものはないというやつぽい

### Languages and Frameworks
#### [io-ts](https://gcanti.github.io/io-ts/)
- Adopt
- TypeScriptにおける外部とのデータのやりとり部分を楽にするもの，という理解
- 所感
    - 全然知らんやつだった．OpenAPI/GraphQLあたりとの棲み分けがわかっていない
#### [NestJS](https://nestjs.com/)
- Adopt
- Opinionatedなnodejsのframeworkとのこと
- 所感
    - `Node Overload` JSでなんでもやろうとするな，というHoldがでていたころから方針転換させるほどなのか，すごそう
    - 門外漢ながら，nodejs/Deno/Bun というjavascript runtime戦国時代が来るかもしれないときに，乗っかっていいのか感がすこしある
#### [React Query](https://react-query-v3.tanstack.com/)
- Adopt
- axios/Fetch/GraphQLなどにも対応済みのdata-fetchingライブラリ. v3でstableになったとある
- 所感
    - 普通によさそう
    - [作者](https://github.com/tannerlinsley)ヤバない？chartjsとかの超有名ライブラリ作者でもあった
#### [Astro](https://astro.build/)
- Assess
- > It’s hard to believe, but in 2022, the developer community continues to pump out interesting new frameworks for building web applications.
- 信じがたいことに，2022年になっても開発者コミュニティはWebアプリケーションを作るための新しくて興味深いフレームワークを生み出している
- 所感
    - 煽りか？と思ったらちゃんと褒め言葉でした
#### [BentoML](https://github.com/bentoml/BentoML)
- Assess
- MLモデルにとってのDockerらしい
- 所感
    - 機械学習領域でのDockerのようなもの，という言い方が気になる
    - なぜDockerではいけないのかを知りたい
#### [Synthetic Data Vault](https://github.com/sdv-dev/SDV)
- Assess
- 過去に本番データをテスト環境で使うことについて，Holdとしていたことに気づいてほうとなった
- 本番データをそのまま使う代わりに，似た特徴を持つデータを生成してくれる，とのこと
- https://github.com/sdv-dev/SDV
#### [Carbon](https://github.com/carbon-language/carbon-lang)
- Hold
- C++後継としてGoogleがOSSにした言語
- GoやRustを使うべきとのこと
- 所感
    - 明確にダメと言われていてちょっと面白かった

## 総評
- Adoptだけじゃなく，Holdが意外と面白い
- 自分とは縁遠くてここではとりあげていないが，ThoughtWorks社的にはデータレイク/データウェアハウスやETLあたりのツール，プラットフォームの仕事が多いんだな，というのが見えて面白い
- あとやっぱりJava/Kotlin大好きやんけ，みたいな感想
- とりあげられているOSSもできてから1年ほどのものがほとんどで，（当然ながら）よく調査されているな，と思った
- 普通に`path to production mapping`, `spectral`, `k6`あたりは業務で使いたい．収穫だった．
- ここでとりあげていないものがかなりある（本家は100以上ある）ので，この記事を読んで気になったものがあった人は，直接みにいくことをおすすめします！PDFはGoogle翻訳でファイルごと翻訳もできますし．
