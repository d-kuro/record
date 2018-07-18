# [AWS] Meguro.dev LT大会

[connpass](https://meguro-dev.connpass.com/event/91743/)

[Meguro.dev LT大会 参加レポート AWSサービスを使ってみたLT #meguro_dev](https://dev.classmethod.jp/study_meeting/meguro-dev-lt-report-lt/)

## AppSync を使ったアプリ開発を効率的に進める方法

### AppSync

* GraphQL のマネージドサービス
* DynamoDB / ES / Lambda を統合
* リアルタイム / オフライン への対応
* iOS / Android / JS / React Native
* CFn に対応

### 大変だったところ

* スキーマの共有フロー
  * iOS / Android でファイルが必要
  * .graphql .json
* マネージドコンソールでダウンロード
* CLI でダウンロード

* スキーマが確定することはあまりない
  * 考慮が足りない場合
  * 機能追加
* スキーマ変更が行き当たりばったりになると大変
  * 型の変更とか

* CFn で Appsync の機能を網羅
* スタック更新はやい
* 環境何個も作れる

* Appsync 側のデバッグが認証使うと大変
* Ampfly でクライアント実装する

[Meguro.dev LT大会で「AppSyncを使ったアプリ開発を効率的に進める方法」を話してきました！ #meguro_dev](https://dev.classmethod.jp/cloud/aws/meguro-dev-lt-appsync/)

## REST API に疲れた私が送る AppSync の思うこと

* REST API
  * Swagger
  * 2000行超えがザラ
  * つらい
* AppSync
  * スキーマを作るときにコメント書いておくと入力時に出てくる & 補完が効く
* だめなとこ
  * 名前がわかりづらい
  * AWS 版 Firebase という文脈で語られすぎ
    * その位置は MobileHub
  * スキーマ固定でなくとも動いて欲しい
    * 動的に
  * データソース増やして欲しい
  * sum とか distinct とかできて欲しい

## Cloud9 のモブプロ環境を爆速で構築する

* モブプロで困ったこと
  * URL の共有がだるい
    * Slack
  * 英語, 日本語キーボード
  * ノートPCに機密情報が入っている
* Cloud9
  * 共同編集モード
  * AWS 上に環境構築
    * AWS リソースが使いやすい
  * Tokyo リージョンがまだきてない

[Cloud9-MobPro-Template](https://github.com/rednes/Cloud9-MobPro-Template)

## AmplifyとAppSyncとIoT Buttonで作ったリアルタイム投票システム

* IoTボタン -> Lambda -> AppSync -> DynamoDB
* グラフは S3 に Amplify 置いとく -> あとは AppSync と pub/sub
* GraphQL のスキーマ定義と React 内での AppSync の使いかた(Apollo) に時間使った

## amplify を勉強してみました

[study-aws-amplify](https://github.com/oracle2k/study-aws-amplify)

## cookpadTV のコメント配信における AppSync の導入事例

* Firebase Realtime databases
  * コメント配信に利用
  * イベント通知を目的としてるので Firebase にはデータを留めない
  * 非同期で永続化する
* 課題
  * Firebase をストレージとして利用していないのでデータの一覧性がない
  * データが永続化されるまでにラグが大きい
  * 急にレスポンスタイムが劣化するけどログがない
  * サポートがない
* AppSync
  * データソースを Dynamo にすることでクエリできる
  * ストリームで永続化がスムーズ
  * サポートが使える
* いい点
  * データソースが Dynamo
  * 新しいテーブル簡単
  * キャパシティの調整が可能
  * クエリの設計が可能
  * 他 AWS サービスとの連携が簡単
* 悪い点
  * ネイティブ SDK がまだ不安定
    * iOS / Android
  * サーバサイドの連携が難しい
  * GraphQL API が SDK でサポートされていない
  * SigV4 の認証を自前でリクエストに組み込む必要がある
    * Lambda を組み込むことで解決？

[cookpadTV のコメント配信における AppSync の導入事例](https://speakerdeck.com/osadake212/cookpadtv-falsekomentopei-xin-niokeru-appsync-falsedao-ru-shi-li)

## Alexaスキル開発。Node.jsからTypescript

* Alexa スキルの開発
  * Lambda を使う
  * Node
  * Python
  * Java, C#, Go
* Node が一番ライブラリが多い
* Node の難点
  * ランライムのバージョン
    * 6.x がまだまだおおい
    * 8.x が Lambda で使えるようになったけど
  * 動的型付が……
* TypeScript での Alexa スキル用 Lambda 開発
  * 必要なパッケージの導入
  * 実装(ask-sdkを使う)
  * デプロイ
    * ask lambda or
    * ask deploy
* TypeScriptnのいいとこ
  * 型使える！
  * async / await 使える

## GraphQL を使いこなすためのNoSQL 設計パターン 改め AppSync を使いこなすための DynamoDB 設計パターン

* DynamoDB の特徴
  * 管理不要で信頼性が高い
  * プロビジョンスループット
  * ストレージの容量制限がない
* DynamoDB を使い場合の勘所
  * Scan させないようにパーティションキーをうまく使う
  * CQRS パターンを使ってテーブルを分散させる
  * 関連するデータをまとめる
    * できるだけ少ないテーブルを維持する
    * どんどん非正規化
  * GSI を利用する
    * セカンダリインデックス
  * DynamoDB は最もよく使われる重要なクエリをできるだけ早く問い合わせするためにスキーマを設計
    * ビジネスユースケース分析
    * クエリのアクセスパターン分析
    * テーブル定義
  * パーティションキーに対してイコールでマッチするアイテムを取得する
  * Scan
    * テーブルを総なめする
    * できるだけ Scan しない
    * キャパシティかかる
  * 意味のない ID をパーティションキーに使わない
  * CQES
    * write と read の責任を分解する
    * 複雑化を避ける
    * w / r をそれぞれスケールできる
  * DynamoDB stream で外に出して read するためのテーブルにつっこむなど
  * AppSync を使うと write は Dynamo / read は ES などにわけることができる
  * DynamoDB の場合は必要なクエリがわかるまでスキーマの設計を開始しない
    * 重要なクエリ
    * 多いクエリ
    * を早く返すように設計する

[AppSyncを使いこなすためのDynamoDB設計パターン](https://speakerdeck.com/ktsukago/appsyncwoshi-ikonasutamefalsedynamodbshe-ji-patan)

## 投票システムの課題

* Lambda の同時実行数制限
* Massaing パターン

[AWS IoT にみる サーバーレスアーキテクチャ Messaging パターン実装例 – Lambda 同時実行数を制御する](https://dev.classmethod.jp/server-side/serverless/messaging-pattern/)

## Alexa スキルのテスト

* Alexaシュミレーター
  * コンソールに入ってる
  * E2E テストができる
  * 実装ビルドデプロイテストのサイクルが大変
  * ビルドとデプロイに時間がかかる
  * エラーの箇所がわからない
* テストをしやすい実装
  * Axexaプロトコルの処理とビジネスロジックを分離する
    * ビジネスロジックはテストしやすい
* Lambda のローカル実行
  * Serverless invoke local
  * デバッグしやすい
  * 対話モデルはテストできない
* Jest
  * JS のテストフレームワーク
* Virtual-Alexa
  * Alexa のテストフレームワーク
  * 発話モデルを取り込んでテストする

[serverless-aws-alexa-ts](https://github.com/HeRoMo/serverless-aws-alexa-ts)
