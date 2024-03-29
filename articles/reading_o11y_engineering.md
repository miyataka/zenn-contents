---
title: "オブザーバビリティ・エンジニアリング🐺を読んで考えたことメモ"
emoji: "🎈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

## そもそもオブザーバビリティ（observability, o11y, 可観測性）ってなんなのよ (what)

この記事タイトルでみにきている読者には不要かもしれないのですが，自分の理解のためにも再度確認しておくと

- 本書の定義
    - 数学的定義
        - > 制御理論では，オブザーバビリティとは，外部出力の知識からシステムのの内部状態をどれだけうまく推測できるかの尺度として定義されています．
        - なるほどわからん
    - ソフトウェアシステムにおける定義
        - > 簡単に言うと，（中略），システムがどのような状態になったとしても，それがどんなに斬新で奇妙なものであっても，どれだけ理解し説明できるかを示す尺度です．
            また，そのような斬新で奇妙な状態に対しても，事前にデバッグの必要性を定義したり予測することなく，
            システムの状態データのあらゆるディメンションやそれらの組み合わせについてアドホックに調査し，よりデバッグが可能であるようにする必要があります．
            **新しいコードをデプロイする必要がなく**，どんな斬新で奇妙な状態でも理解できるなら，オブザーバビリティがあると言えます．

- 自分の理解
    - 「外から観測できたものからどれだけ内部を理解できるかの尺度」であると理解した．
    - 「ある」という言い回しが多用されるが，二値ではなく，グラデーションであることには注意すべきだと感じる．
    - A事象に対しては，十分に理解でき，別のB事象の理解には不十分であるということはありうるし，通常の状態．
    - ただ，ある閾値を超えると質的転換が起こる，ということを指しているのかもしれない．ここはわからない．
    - 「新しいコードをデプロイする必要がない」ということが強調されているが，十分によいとそうなるというだけで，よりよいオブザーバビリティのためにデプロイすることが否定されているわけではない．


## オブザーバビリティを獲得するとなにが嬉しいのか (why)

背景として，従来の手法ではなぜ不十分であるのかを確認する


### （従来の手法である）モニタリングではなぜ不十分か

これまでメトリクスによるモニタリングを前提として運用が行われてきたが，これでは不十分であると本書は主張する
- モニタリングとは，メトリクス（ある数値，またはそれらを集計したもの）を利用した監視手法のことである
- モニタリングという手法がこれまで前提としてきたものが変化している．以下にいくつか上げる．
    - アプリケーションは，モノリスな構成から多くのサービスの組み合わせに変化した
    - クラウド，外部サービスの利用などにより，完全に自分の管理下にあるインフラは少なくなった（低レベルのメトリクスは取得できなくなっている）
    - インフラストラクチャは静的ではなく，非常に動的であり，キャパシティが弾力的に増減するようになった
    - etc
- 前提が崩れた結果として，従来のモニタリングでは障害や異常に対して不十分になった
    - 分散化したシステムでは，どれかが遅くなると，全てが遅くなる（そのように観測される）
    - リクエストが様々なシステムを経由して処理されるため，それらを繋ぎ合わせることができなければ，ほぼデバッグが不可能になる
    - > ソフトウェアエンジニアは従来のデバッガーでコードをステップスルーする能力を失いました．その一方で，ツールはこの激変にまだ対応できていません．
- 注意すべきポイントとして，**モニタリングだけでは「不十分」なのであり，「不要」なわけではない**．モニタリングとオブザーバビリティは組み合わせるべきという理解．


### オブザーバビリティを獲得するとどうなる？

- オブザーバビリティを獲得したシステムは，与えられたリクエストの周りのコンテキストをできるだけ多く保存し，斬新な障害モードにつながった環境や状況を再構築できるようになる
- 雑に言い換えると，十分なコンテキスト情報があれば，なぜ・どのように問題事象が起きた（起こっている）のかを事後的に探索・分析できるようになる


## オブザーバビリティの実現のためには何が必要か

高いカーディナリティとディメンション，それを活用するためのツールが必要．
- 高いカーディナリティ
    - カーディナリティ (cardinality) はデータの一意性の程度のこと．低いと重複した値が多く含まれていることを意味し，高いと一意である値の割合が多いことを意味する．
    - 膨大なイベントを分類あるいは特定し，紐づけるのに必要
- 多くのディメンション
    - データ内のキーの数を意味する
    - オブザーバービリティのあるシステムでは，テレメトリーデータは任意の幅広い構造を持つイベントとして生成される．何百，何千のキーバリューのペアを持っており，また持っているべきである
- 上記二つを活用して，探索をサポートするツール
    - イベントやログを記録するだけではなく，それを探索できる必要がある．
    - 多くの場合は外部ツール，サービスを利用することになる．


## オブザーバビリティの実現のためにはどんなエンジニア的なアクションが必要か

- 任意の幅の構造化イベントの収集を実現する
    - `任意の幅`としているのは，あらゆるコンテキストをkey-value形式で含める必要があるため
- イベントをトレースにつなぐ
    - OpenTelemetryはここに活用できる


## オブザーバビリティの応用

ここでは詳細は省くが，以下についてのかなりの言及がある．具体的な記述でフェーズによってはとても役立つだろうと思われる．
- SLOへの活用
- ソフトウェアサプライチェーンへの活用
- コスト最適化のためのサンプリング戦略
- オブザーバビリティ成熟度モデル (Obverbability Maturity Model, OMM) という考え方の提示がある


## 現時点での個人的まとめ

- 自分にとっては読む価値はあった
    - 特にトレーシングの有用性は日々実感するところでもあるし，発展した先のイメージをつかむことができた
    - 日々接しているシステムがマイクロサービスとまでは言わないくらいの分散システムなので，複雑性が高まったバージョンを想像すれば必要性・重要性はかなり理解できる
- オブザーバビリティは現代的なシステムが必要とする特性であり，健全性の指標になりうる
- （当然ながら）オブザーバビリティはすべてを解決するわけではなく，すべてのケースで有用なわけではない
    - そもそもモノリスなシステムではモニタリングで十分である
    - モニタリングを置き換えるわけではない，新しいアーキテクチャには別の手法が必要だということである
    - 膨大かつ詳細なコンテキスト，ログを追加的に取得するわけなので，コストがかかることは念頭におく必要がある（とはいえメリットが上回るケース）
- OpenTelemetryでSaaSやソリューションを切り替えられる統一的なインターフェースができたことは喜ばしい


## 注意すべきポイント

著者らの所属について本書の中でも繰り返し強調されていて，（かつそれ自体はかなり誠実で好印象です）
著者らは特定のo11y serviceを提供している企業に所属していて，具体例としてよく出てきます
