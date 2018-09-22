# Kubernetes と Serverless を利用した開発

[Kubernetes と Serverless を利用した開発](https://cloud.withgoogle.com/next18/tokyo/my-schedule/session/235121)

## サーバレス

* 単なる機能の枠を超えるサーバレス
  * FaaS
  * Functionだけじゃない

## Google Cloud のサーバレス

* CloudFunction
* GAE
* Cloud Strage
* Cloud ML
* Cloud PubSub
* BigQuery
* Cloud Dataflow
* Cloud Datastore

デベロッパーがアプリの開発に集中できるように

## サーバレスの特徴

* サーバ不要
* 設定が用意
* イベント駆動
* ベンダロックインなし

## k8s + サーバレス

* k8s をサーバレスにするとポータビリティを実現できる

## GKE serverless add-on

* アドオンとして提供される
* `--addons=Serverless`
* 1ステップのデプロイ
  * GKE にアドオンをインストール
  * 少ない設定/コードでデプロイ
* サーバレスワークロードの実行
  * ソースからURL
  * コンテナのデプロイとイングレスのプロビジョニングを自動実行
* 自動スケーリング
  * ゼロまでスケールダウン
* g.co/serverlessaddon
  * 年内に提供開始予定
  * 今はアーリーアクセス

## Knative

* サーバレスワークロード向けのk8sベースの基盤
* k8s 上のサーバレス

## Knative の開発原則

* 設定が容易
  * k8s 上でネイティブ感覚の開発
  * デベロッパーに適応
* 拡張可能
  * 上層で疎結合
  * 下層でプラガブル
* 有機的
  * 共通部分の体系化
  * 安定したプラットフォームの構築

## Knative のまとめ

* k8s上のサーバレスワークロード用プリミティブ
* ビルド, サービング, イベントから開始
* ワークロードのポータビリティを実現
* 幅広い業界に対応
