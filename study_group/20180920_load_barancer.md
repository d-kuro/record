# 地球規模でスケール可能なサービスの実現について 〜 Cloud Load Balancing と Istio の利用 〜

* [地球規模でスケール可能なサービスの実現について 〜 Cloud Load Balancing と Istio の利用 〜](https://cloud.withgoogle.com/next18/tokyo/my-schedule/session/223420)

## ネットワーク LB

* TCP, UDP のバランシングを行う
* オリジナルの IP は変更しない
* 内部で maglev が動作
* シングルリージョンに限る
* L7 の無理
* グローバルでバランシングしたかったら DNS で頑張るしかない

## グローバル LB

* 一つのグローバル IP アドレス
* シームレスな拡張
* リージョン間でのフェイルオーバもできる
* 全てソフトウェア化されている

## コンテネイティブな負荷分散

* ネットワークエンドポイントグループ
  * Pod の中に振り分ける
* マルチクラスタ Ingress
  * 一番ユーザに近いところにパケットを流す

## 世界全体のエッジセキュリティ

* TLS everywhere
  * どこでも TLS
* 複数の SSL 証明書
* マネージド SSL 証明書
  * ライフサイクル管理不要
* マネージドカスタム SSL ポリシー
* グローバル LB + Cloud Armor + IAP

## レイテンシや最適化

* Cloud CDN
  * レイテンシが低い
  * AWS よりずっと低い！
* gRPC を使用するバックエンドへHTTP/2 を有効化 LB に対して QUIC を使用
* QUIC
  * UDP の暗号化
  * Chrome で使われている

## CLB の今後

* インフラではなくサービスを管理するサービスメッシュ
* マルチクラウド
* マイクロサービス
* Istio + Envoy

## メルカリのモノリス

* モノリスの巨大化
* 変更による影響範囲の不明確化
* テストの複雑化
* on-boarding コストの増加

## Migration

* モノリスの手前に API Gateway を配置
  * フロントからの全てのトラフィックは API Gateway を通る
* マイクロサービス単位でチームを分けてなるべくマネージドサービスを使う
* CLB
  * TLS 終端
  * DDoS 対策
* API Gateway
  * ルーティング
  * ロードバランシング
  * AuthZ/AuthN

## Furure Work

* GKE On-prem
  * モノリスはオンプレに置いてある
  * オンプレに置いてあるモノリスをマイクロサービス化する
* Istio Service Mesh
  * サービス間のコミュニケーション
* Multi-Cluster Ingress for HA

### サービス間コミュニケーションの課題

* Observability(可観測性)
* Network reliability(ネットワークの信頼性)
  * 適切なリトライ, タイムアウト
  * サーキットブレイク
* Security(セキュリティ)
  * このサービスはこのサービスからのみアクセス可能みたいな制御が必要
* これらの課題を Istio のレイヤで吸収する
