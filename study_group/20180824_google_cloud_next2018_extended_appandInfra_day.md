# Google Cloud Next 2018 Extended App & Infra Day

[Google Cloud Next 2018 Extended App & Infra Day](https://gcpug-tokyo.connpass.com/event/94417/)

[Youtube](https://www.youtube.com/watch?v=oPlP_Fa3QJA)

## GCP Database の世代交代

[スライド](https://docs.google.com/presentation/d/11Jp1gt-n8LxVgrHBRy1ym6iJMsKww-mit1A00IS-oHk/edit#slide=id.g3cf2532adc_0_136)

### GCP Database の系譜

* Bigtable が原型, 2006年論文公開
* Datastore, 2011年リリース
* Spanner, 2013年論文公開
* Firestore, 2017年リリース

### Bigtable

* 分散型KVS
* ストレージは自動拡張
* クエリ機能はなし, Key で Get のみ
* トランザクション機能なし
* Client -> Node -> Tablet の階層でデータを保存
* 行 Key の辞書順にデータを保存
  * PK の先頭付近の文字が重要
  * TabletA は a ~ e までみたいな
* 行 Key に偏りがあると Tablet に hotspot ができてしまう
  * a のデータだけがめっちゃ偏るとか
* 行 Key は分散するようにする
  * 行 Key の範囲スキャンしかできないので色々考えることがある
  * UUID とかにすると完全ランダムすぎて辛い
* データがない状態だと分散していない状態からスタートするのでじわじわデータを入れる
  * ローンチの瞬間にめっちゃトラフィックが予想される時とかはあらかじめダミーを入れて分散させておく
* Key をどうするか
  * フィールドプロモーション
    * Key にユーザIDとかのクエリするときに利用する値を Key に入れておく
  * ソルティング
    * 行 Key から計算した値を行 Key に入れる
    * Hash
* 新しい機能
  * Key のヒートアップがわかるようになった

### Datastore

* 分散型KVS
* Bigtable のアプリケーション向け版
  * Zone base の Bigtable に対して Region base になってる
* プロパティに対するクエリに対応
* トランザクションに対応
* 課金は RW 回数とストレージ容量
* Datastore -> Megastore -> Bigtable の階層
* Datastore
  * プロパティに対するクエリを行う
* Megastore
  * データセンタ間のレプリケーション, トランザクション
* Datastore は Bigtable 上に Index Table を作成して Scan
  * Index がなかったら Scan ができないので必要な Index がない場合はクエリは失敗する
* Index は非同期で作成
  * put とかは Index の作成を待つことなくレスポンスが返る
    * Index がいっぱいあったりしてもパフォーマンスに影響がない
  * クエリの実行タイミングによっては古い結果が返る
* Entity Group
  * Entity 間で親子関係を組む
  * Key の頭に親の Key を入れることで Bigtable 上で連なって保存される
* Ancestor Query
  * 親を指定して子供を取得するクエリ
  * 親子同時に取得できない
* トランザクション
  * トランザクション開始時に Entity Group のスナップショットをとって Commit 時に最終更新時刻をぶつけ合わせる
  * 25 Entity Group までしか参加できない
* GROUP BY とか JOIN がない

### Spanner

* 分散型 RDB
  * 分散 KVS に RDB としての機能を付属しているような感じ
* Bigtable, Datastore の後継なのでいい感じに問題が解決されてる
* Node 課金
* Clinet -> Node -> Split の階層
* Split の決定方法
  * PK の辞書順
  * Split がでかくなりすぎたり負荷がかかると分散する
* PK が偏ると Split に hotspot ができる
  * 連番とかを PK にすると辛い
  * UUID を PK にしたりする
  * 複数カラムを PK にしたり
  * ハッシュ
  * UUID + UNIUE INDEX
* ダミーとかいれたりするのも Bigtable と同じ
* Index も結局テーブル
  * Index として指定したカラムを PK としてテーブルを作成する
    * PK にすれば勝手に辞書順に並ぶ
  * Index を設定するカラムもなるべく分散させる
    * 分散しないカラムには Shard を使ったりする
* タイムスタンプベースのトランザクション
* 複数マシンでの時刻のズレが小さい方がパフォーマンスが上がるので TureTimeAPI と呼ばれる API を作成している
  * 複数マシンでの時刻のズレを小さくするやつ
* 新しい機能
  * Query Stats

### Firestore

* Native Mode
  * 元々あったやつ
* Datastore Mode
  * 既存の Datasore が全部置き換わる
* Native Mode
  * モバイルバックエンド as a Service のための DB
* Datastore Mode
  * 既存の Datastre
  * ダウンタイムなしで行こう
* Firestore -> Spanner の階層
  * バックエンドが Spanner
* バックエンドが Spanner なので Datastore の下にあった　Megastore と Bigtable とはお別れ
* Bigtable から Spanner に世代交代されてる
  * データが多いほど力を発揮しやすい分散 DB

### Datastore Mode

* 全ての操作が強い整合性に
* トランザクションの 25 制限撤廃
* Entity Group の更新秒間 1 回までが撤廃
* Batch 操作が Atomic に
  * 今まで成功失敗が一緒に返ってきてた
* 新規プロジェクトなら今すぐ使える
* Tokyo Region がまだ
* API とかコンソールの内容は全部既存のと一緒

### Native Mode

* Single Property Index の設定機能リリース
* Array Index サポート
* Inport / Expoert
  * Expoert は Datastore と同じ方法で BigQuery に Load できる
* GCP コンソールに Firestore が登場したけどセキュリティルールの項目がない

***

## アプリ, インフラ, そして運用についての究極の疑問の話

[スライド](https://speakerdeck.com/kumakumakkk/apuri-inhura-sositeyun-yong-nituitefalsejiu-ji-falseyi-wen-falsehua-to-nextfalsefeng-jing-woshao-jie)

### 本当にマイクロサービスは最高なのか

* 運用が複雑に
  * どのコンポーネントがどこと通信してるのか
  * 俯瞰, 監視どうする問題
  * コンポーネント増えすぎでデバッグ, デプロイ大変じゃね？
* 結局のところマイクロサービスでもモノリスでもどっちでも複雑になる

### いろんなツールを使って管理できるように

* 運用管理に必要なものを自動化
  * インフラ
  * ネットワーク
  * 監視, アラート
* ワークフローを自動化しよう
* Docker
  * コンポーネントをパッケージ化
* アプリの実行, 管理
  * k8s
* クラウド
* サービスを管理
  * Istio
  * サービスメッシュ
* Istio 難しい
  * k8s がただでさえ難しい
  * 進化早すぎ
  * 管理するものが増える
    * 最終的な ROI を考えたらマネージドサービスをフル活用 + モノリシックの方がいいのでは？
* ネットワークレイヤ以外の複雑さ
  * CI/CD
    * マイクロサービスの粒度
    * 1メソッド1マイクロサービスぐらいにすると圧倒的デプロイスピードが得られるけどデプロイの頻度がやばすぎる
  * 「近年のソフトウェアの開発サイクルが早すぎてテストをする時間が短すぎる」
  * モニタリング
    * 監視するもの多すぎ
    * ログ多すぎ
    * どうやってデータ見る問題
  * どこにアプリケーション置くかとかオンプレ資産どうする問題
* Cloud Services Platform
  * **Google が描く現状の最適解をパッケージで提供してくれる**
  * マネージドな Istio
  * 色々使って CI/CD ワークフロー作成
  * Stackdriver Service Monitoring で一括してモニタリングできるよ
  * GKE On-Prem でハイブリッド環境作れるよ
* Stackdriver Service Monitoring
  * どことどこが繋がっているのかみたいなものを図で出してくれる
  * ノードの情報
  * グラフ
  * Trace
  * Metrics
  * SLI の監視
  * Error budget のグラフ
* GKE On-Prem
  * オンプレミスと組み合わせたハイブリッドクラウドが作れる
  * コントロールプレーンは GCP 上になる
  * オンプレ環境でクラスタを組む
* これは誰向け？
  * アーキテクチャの複雑性を紐解き
  * 複雑なシステムを自動化し
  * 許容できる範囲を明確に定義し
  * 事業、システム、ヒトに整合性をつけて全体を無理なく
  * 柔軟性がもてるシステムをしたい人向け
  * **全員が相互理解できている状態じゃないと機能しない**
