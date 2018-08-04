# 夏の AWS Fargate & Amazon ECS/EKS 祭り

[夏の AWS Fargate & Amazon ECS/EKS 祭り](https://pages.awscloud.com/AWSFargateAmazonECSEKSmatsuri20180803-jp.html)

## AWS Fargate & Amazon ECS/EKS 最新情報

### コントロールプレーン / データプレーン

* コントロールプレーン
  * コンテナの管理をする場所
  * ECS / EKS

* データプレーン
  * 実際にコンテが稼働する場所
  * Fargate / EC2

### ECS でのフィードバック

* ECS の中でもっとも多かったフィードバック
  * アプリの開発に集中したい
  * EC2 の管理したくない

### Fargate

#### リソース消費モデル

* インスタンス管理不要
* タスクネイティブAPI
  * ECS と互換
* リソースベースの価格

#### CONSTRYCTS

* Tack Definition
  * アプリケーションとコンテナの定義
  * Image URL, CPU と Memory
* Task
  * コンテナの実行単位
* Service
  * ロングランニングアプリ用スケジューラ

#### Fargate でのコンテナ実行

* ECS と同一スキーマの Tack Definition
* ECS API を用いた起動

#### VPC INTEFRATION

* AWS VPC Networking Mode
  * 各タスクがネットワークインたフェースを持つ
* Task とやり取りする全てのネットワークトラフィックに Task ENI が利用される
  * アプリケーションがアウトバウンドのインターネットアクセス不要でも Image の Pull や log の創出に使用

#### ロードバランシング

* ALB と NLB をサポート
  * クラシックLB は非対応

#### パーミッションのタイプ

* Cluster Permissions
  * 誰がタスクを実行参照できるか
* Application Permissions
* Housekeeping Permissions

#### モニタリング

* CWL
* CW Events

#### Fargate or EC2

* Fargate では難しいもの
  * windows
  * GPU
  * docker exec など
  * Spot や RI

これ以外は Fargate で OK

#### Fargate or Lambda

* Lambda を使う方がいい場合
  * イベントドリブン
  * ms単位のコンピュート
  * ランタイム管理をしたくない
  * 分散バッチコンピューティング

それ以外は Fargate

#### コンテナの Auto Scaling

* Target Tracking (ECS)
  * メトリクスに対してターゲットの値を設定するだけ
  * 楽
* Fargate
  * Service のスケールに応じて自然にコンテナが起動終了する
* EC2
  * EC2 本体のスケールも考慮が必要

### Amazon EKS

* k8s のコントロールプレーンをマネージドにする
  * マスタを 3AZ にまたがって自分で配置みたいなことが必要なくなる
* マスタや Etcd が抽象化される

#### API

* クラスタを作る API
* クラスタを表示する API
* クラスタを list する API
* クラスタを削除する API

#### ワーカーノード

* 自分のEC2インスタンスを使用する必要がある
  * オートスケールとかも自分で設定する
* k8s 用の AMI がある
* GitHub に AMI ビルドするスクリプトがあったりする

#### ネットワーキング

* VPC CNI plugin
  * k8s と VPC 間をブリッジ
  * AWS 上の k8s であれば利用可能
* v1.1 リリース
  * Pod に対する Desable Sucrce Address Transration をサポート
  * IP アドレスを事前に割り当てた warm pool を作成
  * EKS ノードのスケール時のレイテンシ軽減
  * ENI tragging

#### エンドポイントの認証

* IAM でやる

#### Fargate の EKS サポート

* 予定してます

#### おまけ

* dev day は 8月末に色々公開される
* [DEV DAY 2018 (2018年10月29日～11月2日)](https://pages.awscloud.com/DevDaySavetheDate20181029-1102-jp.html)

***

## コンテナ化したアプリケーションを AWS で動かすためのベストプラクティス

### なぜコンテナなのか

* パッケージング
* 配布
* イミュータブルインフラストラクチャ

#### コンテナのユースケース

* マイクロサービスアーキテクチャ
  * 多数のマイクロサービスを同じように管理
* 非同期ジョブ実行
  * ジョブのリクエストに応じた柔軟なスケール
* CI/CD
  * 開発 ~ テストまで一貫したイメージを使用

#### コンテナを利用した開発に必要な技術要素

* アプリのステートレス化
  * ステートが必要なものはコンテナの外に置く
* レジストリ
  * コンテナのイメージの置き場所
  * 高い可用性, スケーラビリティが求められる
  * ECR
* コントロールプレーン / データプレーン
  * コントロールプレーン(ECS/EKS)
  * データプレーン(Fargate/EC2)
* CI/CD パイプライン

### ECR

* フルマネージド
* IAM による認証管理
* ライフサイクルポリシーでイメージの自動クリーンアップ
  * 何日以前や何個以上は自動削除といったルールを組み合わせて使用可能

### ECS

* Task
  * Task Definition の情報から起動
  * 1つ以上のコンテナを実行しているリソース
  * CPU とメモリの上限を指定する
* Service
  * Task の数を希望数に保つ
  * Task Definition を新しくすると B/G デプロイ
  * ELB と連携することも可能
  * メトリクスに応じて Task 数の Auto Scaling も可能
* IAM Role
  * Task 毎に設定可能
  * AWS SDK を利用して入れば自動的に認証情報が得られる
* VPC ネットワークモード
  * Task 毎に ENI を自動割り当て
  * Task 内のコンテナは local host を共有
* Service Discovery by Rout 53 (New!)

### CI/CD パイプライン

* 誰がやっても同じようにデプロイできる
* アプリ毎に統一された手法を利用する
* イメージがどうやって作られどこで使われているかを把握

#### AWS CodePipeline

* 継続的デリバリサービス
* リリースプロセスをモデル化
* コードが変更されるたびにビルド, テスト, デプロイ
* サードパーティツールや AWS との統合
* マニュアル承認のサポート
* ECS/Fargate へのデプロイサポート

#### AWS CodeBuild

* フルマネージド
* 継続的なスケールと自動複数ビルドの実行
* 実行環境は Docker image
* 利用した分だけの課金
* CodePipeline や Jenkins との統合が可能
* S3 キャッシュ機能
* ローカルの CodeBuild 環境を提供, デバッグが容易に
* VPC 内のリソースにアクセス可能
* VPC エンドポイントをサポート

#### AWS CodeCommit

* Git リソース管理
* スタンダードな Git tool を使用可能
* バックエンドが S3 なので スケーラビリティ, 可用性, 堅牢なストレージ
* カスタマ特有のキーを使用した暗号化
* SNS/Lambda を呼び出せる
* PullRequest 対応
* ブランチごとの権限管理が可能

### まとめ

* コンテナを利用するメリットは多い
* AWS のフルマネージドなサービスを利用してコンテナのCI/CDを容易に実現可能
* AWS Fargate はインスタンスの管理不要で理想的なオートスケールも実現可能でアプリケーション開発に集中できる
* AWS CodePipelineを利用することで Docker image をAWS Fargate にノンコーディングでデプロイ可能

***

## Kintone on EKS

サイボウズ株式会社　運用本部　野島 裕輔 様

### なぜ EKS の上に構築することにしたのか

* 手順は差分的
  * 現在の状態と理想の状態の差分
* 手順をそのまま自動化すると脆いシステムになる
  * 現在の状態が少し変わっただけで動かなくなる
* ロバストな自動化のためには「理想状態の収束」という考え方が必要
  * 現在の状態と理想の状態の差分を自動的に抽出し、必要な手順を行う

EKS + CloudFormation でフルスタックで「理想状態への収束」ができる

### 継続的インテグレーション

* 使われていない自動化は壊れていく
  * 壊れたらすぐ気づいて修正したい
* 常に自動化を使う
* Git push するたびに CI で変更を開発環境に適用する
* 1日1回 CI で VPC を削除してゼロから再構築する

対象の環境をソースコードで宣言された状態に収束させる

### Pipeline

* Provisioning
  * CFn で VPC, EKS, RDS等を構築
* Build
  * コンテナイメージをビルド（テストも実行）
* Deploy
  * k8s のマニフェストを用意して kubectl apply で適用
  * コンテナは基本的に Deployment というリソースを通じて作る
* Initialization
  * サービスを管理するサービスを作り、REST API で初期化できるようにした

### ECS / EKS のどちらを使うか

* EKS(k8s) の利点
  * k8s はオンプレでも使える
  * 機能が豊富
  * コンテナオーケストレーションのデファクト
* ECS の利点
  * AWS とのインテグレーションが強い
  * リリース直後の EKS と比べると安定している
  * コントロールプレーンが無料
* なぜ ECS ではなく EKS を選んだのか
  * 自社DC との相互運用性

## AWS FargateでElixirのコンテンツ配信システムを運用してみた

エムスリー株式会社　エンジニアリンググループ　園田 亮平 様

[AWS FargateでElixirのコンテンツ配信システムを本番運用してみた](https://www.m3tech.blog/entry/elixir-aws-fargate)
[AWS FargateでElixirのコンテンツ配信システムを動かしてみた (実装編)](https://www.m3tech.blog/entry/elixir-fargate-impl-1)
[AWS Fargateのデプロイパイプライン(Gitlab > S3 > CodePipeline)を構築してみた](https://www.m3tech.blog/entry/elixir-fargate-impl-2)

### Elastic Beanstalk -> Fargate 化してみて

* デプロイ時間の短縮
  * 20m -> 10m
* プラットフォーム構成変更が楽になった
  * プラットフォーム変更にかかる時間が短縮された
  * 1h -> 2m
  * Docker 化によりローカル環境でプラットフォーム自体のテストができるようになった
* コンテナ化によるオーバヘッドはほとんど見られなかった
* デプロイパイプラインを管理しやすくなった

### 従来のElastic Beanstalk構成と課題

* Fargate 化以前は Elastic Beanstalk の Custom Platform を利用して環境を運用

[このブログに書いてある](https://www.m3tech.blog/entry/elixir-aws-fargate)

* Custom Platform のビルド時間が長い
  * Erlang のビルドに 40分
* サーバ構成を .ebextensions で管理するようになる
  * アプリケーションのデプロイがサーバ構成のエラーで失敗するようになる
* Gitlab CI で利用するIAMユーザの権限がかなり強くなる
  * Gitlab CI  のパラメタが簡単に参照できるのでセキュリティ上よろしくない
* TerraformがCustom Platformに対応していない
  * 自身で Terraformのソースにパッチをあててビルドしたバイナリをリポジトリにコミットして利用する運用

Elastic Beanstalk の問題ではなく Erlang, Gitlab, Terraformといった組み合わせによる課題だった

### Fargate 化後の構成

* ECS 化を行う予定があったが Fatgate が東京にきたので Fargate に移行した

[構成図はブログ](https://www.m3tech.blog/entry/elixir-aws-fargate)

* デプロイパイプラインの構成
  * Gitlab ではソースのアーカイブのみを行い S3 に upload
  * Docker image のビルドや ECS へのデプロイは CodePipeline で行う

#### 構成変更後の改善点

* プラットフォームのビルド時間
  * 1h -> 2m
* IAM の権限
  * s3: PutObjectだけになった
* Terraformのパッチを当てなくなった

#### 構成変更後の課題

* コンテナはシングルプロセスなので監視用のプロセスをどう実装するかは検討が必要
* buildspec.yaml がアプリケーションリポジトリ内にしか置けないので CodeBuild がアプリケーションリポジトリに依存する
* AWS 利用料が少しだけ割高?

## EKS, Fargate, コンテナの運用監視は今までと何が違うのか？よくある課題とその対策

Datadog, Inc.　服部 政洋 様

### Detadog とは

* Saas ベースのインフラ&アプリケーションモニタリング
* OSSのエージェント
* 時系列データ
* 1日あたり1兆のデータポイントとトレースの処理
* 契約社が9000社くらい
* インテリジェントでアクショナブルなアラート

### EKS/Fargate の監視はどうするのか

* コンテナ環境ではデータを取るプロセスに特徴あり
* かといってモニタリングが特別すぎるわけではない

### なぜコンテナを導入するのか

大前提: コンテナ化によるビジネスアジリティの獲得

* 依存関係地獄からの脱却
* 圧倒的なスケーラビリティの獲得
* 超軽量で高速なデプロイ

だが……

* コンテナも所詮VM, 従来通りのサーバ監視を適用?
  * コンテナ自体のメトリックやイベントは全くなくモニタできていないことを深刻な障害発生によって知る
  * 想定をはるかに超えてスケールし監視ツールがスローダウンする

### コンテナの運用は何が違うのか

[8 surprising facts about real docker adoption | Datadog](https://www.datadoghq.com/docker-adoption/)

* Docker 利用者はユーザ全体の 25%, 1年で 40% 増の勢い
  * 1年で5ポイント増加
* Docker が稼働するホストは datadog 全体の 20%
* 50%のユーザががオーケストレーターを利用
* Docker の導入から10ヶ月のコンテナ数の推移
  * 導入した途端に急拡大する
* k8s のシェアがオーケストレーター全体の41%
* オーケストレーターがコンテナのライフライムを平均で半日までに縮小
  * オーケストレーターで管理されていないコンテナは6日
* 全てのコンテナユーザでは1ホストあたり中央値で8コンテナが稼働
  * 25% は17以上のコンテナを稼働させている
* k8s の利用は様々はコンテナイメージの利用も推進
  * コンテナユーザ全体では Nginx, Redis が多い

### コンテナ化によって何が変わるか

* モニタすべきメトリック, イベント, ログは激増する
* 今まで以上に物理的/論理的なレイヤが増える
  * クラスタ, ネームスペース, ポッド, ノード, コンテナ, コード etc……

### スケールを前提としたモニタリング基盤とは

* 従来のサーバ監視: サーバホスト中心
  * スケールすると複雑になる
* スケーラブルなモニタリング: サービス中心
  * タグやラベルによるクエリベースのモニタリング

### メトリックの収集

参考にするといいやつ
[cncf/landscape](https://github.com/cncf/landscape)
