# Cloud Functions ではじめよう、サーバーレスアプリケーション開発

[Cloud Functions ではじめよう、サーバーレスアプリケーション開発](https://cloud.withgoogle.com/next18/tokyo/my-schedule/session/223430)

## Google Cloud Functions

* イベント駆動型
* サーバレスコンピュートプラットフォーム

## サーバレスとは

* 透過的なインフラストラクチャ
* 自動スケーリング
* 未使用CPUサイクル分課金なし

## スケーリングポイントの縮小

* 共有/物理マシン
* 仮想マシン
* コンテナ
* PaaS
* 関数

## Cloud Functions

* トリガーイベントに反応してバックエンドコードの関数を自動的に実行

## 種類

* HTTP関数
  * Webhook もできる
* バックグラウンド関数
  * Cloud Strage
  * Cloud Pub/Sub
  * Firabase

## GA Node6環境

* GAはNode6だけ
  * Image Magick プレインストール
  * /tmp にローカルのエフェメラルディスク
* ベータ
  * Python
  * Node8
* アルファ
  * Go

## HTTP 関数

* HTTPSエンドポイント
* FQDNを提供
* SSL／TLS証明書を自動生成
* 自動スケーリング
* 1秒以下の単位で課金
* リクエストのパラメータを自動でパース

## バックグラウンド関数

* Cloud Strage のイベント
  * ファイナライズ
  * デリート
  * アーカイブ
  * メタデータのアップデート
* 同じトリガーで複数の Function が動く

## ローカルでの開発

* オープンソースのエミュレータがある
* Node だけ

## 監視

* Stackdriver なら自動でキャプチャされる

## 新着情報

* GA
  * 複数リージョンサポート
  * 東京含めて
* 言語サポート
  * Phyton 3.7(ベータ)
  * Java/C#/PHP とかも検討段階
  * クラウドでの自動構築と依存性管理もサポート
* Cloud Functions for Firebase
  * Node8を降るサポート
* Ubuntu ベースイメージ
* セキュリティ管理, VPC, IAM
* 環境変数
* Cloud Scheduler
* スケーリング管理
* Cloud SQL に直接接続
