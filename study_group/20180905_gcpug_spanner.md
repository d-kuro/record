# GCPUG Tokyo Spanner Day September 2018

* [connpass](https://gcpug-tokyo.connpass.com/event/97825/)

## Google Cloud Spanner Deep Dive

* [スライド](https://docs.google.com/presentation/d/1XKaOrex3WS8xZ0TsjsTQKBgxjVSGxkyqdcIqG66zd64/edit#slide=id.p)

### Spanner とは

* Googleが開発した分散型データベース
  * OSS じゃない
* KVS と RDB が合体したやつっぽい
  * 従来の RDB と互換もってないので MySQL 感覚で使うと即死する
  * 開発当初は KVS
  * **KVS に SQL 投げる感じのデータベース**
* GCP に登場したのは 2017 年
* Spanner 使ってる世界最大のプロダクトはおそらくポケモンGO

### Spanner の特徴

* 水平にスケールするのが得意
  * マシンタイプなしで Node の数だけ指定する
  * Node の数は好きなタイミングで指定
* ストレージは無制限
  * 従量課金
  * 1 Node 2 TB というガイドラインはある
* 強い整合性

### Spanner Component

* Instance
  * リージョン
  * ノードの数
  * を指定する
* DB
  * IAM
  * マシンリソースは DB 全体で共有
* Table
* Index

### Node の数

* 全てが R/W レプリカ
  * Read only レプリカは存在しない
* ダウンタイムなしで増減
  * **自動でやってくれない**
* プロダクション環境は 3 Node 以上が推奨
* セッション数 Node * 10000

### リージョン

* リージョンはシングル/マルチを選択できる
* マルチリージョンは Write ができるリージョンが限定される
* マルチは高い
* 1時間 $9
* 基本はシングルリージョンでみんな運用してる

### アーキテクチャ

Root Server

* Node
  * Split
  * Split  
* Node
  * Split
  * Split  

### Split

* Spanner は PK の辞書順にデータを保存
  * **PK の先頭付近の文字が重要**

### Split の分割

* 1 つの Split が大きくなりすぎたり、負荷が上がると Split が分割する
  * 1 Split 2 GB くらいらしい

### Split の再配置

* Node の増減時は Split の割り当てが再配置される
* 1 ~ 2 sec くらいでおわる
  * パフォーマンスに影響出るのはもう少しかかる
  * 20 時にスパイクが来るとかわかってる時は 1時間前くらいに Node を増やすと良い

### Hotspot 問題

* PK に偏りがあると Split に hotspot ができてしまう
  * 辞書順なので特定のものに偏りができるとやばい
  * Spanner 側も分割とか頑張ってくれるけどどうしよもならないことがある

### PK 戦略

* **Spanner を使う上で最も重要**
* PK は絶対分散させる
* 公式のベストプラクティスは UUID
  * お手軽にランダムにばらける
  * 順番がばらばらになれば他のでもいい
  * 先頭が乱数になる PK とか
* バッドパターン
  * シーケンシャルナンバー
  * タイムスタンプ
  * 大 + 中 + 小分類 + ID (いずれかに hotspot があると辛い)
* 一番パフォーマンス出るのは Split がいっぱいあって均等にアクセスが分散するとき
* 一番パフォーマンスが出ないのはデータが空で Split が 1 の時
  * 空から始めるならじわじわデータを入れて行くのがよい
  * 最初からスパイクが想定される時は事前にダミーを雑に突っ込んでおくのがよい
* PK にしたい値から、順序がランダムになる値を生成する
  * ハッシュ
  * 逆順 (ID + 小 + 中 + 大分類)
  * Shard

### 複合キー

* 複数カラムを PK にできる
  * 先頭のカラムが分散する必要
  * 途中で ID に含めるカラムを増やせない
* ハッシュ
  * Hash (カラム1 + カラム2)
* UUID + UNIQE INDEX
  * UNIQE INDEX の先頭が分散してる必要がある
  * UUID でクエリしないと遅い

### Index

* **Index もテーブル**
* Index として設定したカラムを PK としたテーブルを作成する
  * PK にすれば勝手に辞書順に並ぶ
* Index を設定するカラムもなるべく分散させる
  *分散しないカラムには Shard をつかったりする

### 単調増加する値の Index

* timestamp を保存する場合必ず既存より大きい値になる
* Hotspot になる
* Shard を使いましょう
* クエリ時は ShardId BETWEEN 0 AND 9 を指定することで、全 Split に対してクエリを実行する

### PK を分散させればhotspotは起きないのか

* PK に UUID を使っていても hotspot は発生する可能性はある
  * データが少なくて Split が少ない
  * 同じ PK へのアクセス頻度が多い

### Spanner の気持ちになる

* 自分のアプリケーションはどうやって DB にアクセスするかを考えて設計する
* Spanner は銀の弾丸ではない
* 金の力で Node 増やしても解決にはならない
  * 結局アクセス偏ったらダメ
* Write の頻度、件数
  * エンドユーザが write するなら早く
  * バッチとかなら別に少し遅くてもいい
* どうやって read するのか
  * PK で取得
  * 最新の 10 件
  * フィルターかけたり
  * 設計方法が変わる
* write と read どっちが早い方が良いか
  * write がバッチ、read がユーザなら read が早くなる方を優先する
  * ShardId BETWEEN 0 AND 9 みたいなのはフルスキャンでめっちゃおっそい
  * 先頭10件欲しいなら各シャードの10件とって、その100件の中でならべかえるみたいな工夫をする
  * write が少ないなら index でもいい
  * Spanner に対する理解が必要

### クエリ

* Index を指定して欲しいクエリを実行するなら FORCE_INDEX を指定する
* Spanner のオプティマイザはあんまり賢くない
  * アホの子
  * どうみても index 使うだろみたいなのでも使ってくれない
* 統計とかもとったりしてない
* Spanner に寄り添ってあげる

### ページネーション

* Spanner では難しい
* Cursor が無い
* offset だと取得した後 offset 分捨てる
  * 後ろにあると辛い
* 最初のクエリの最後の値を WHERE に入れてクエリを投げる必要がある
* ページネーションするときは Index はユニークであるべき
* CreatedAt とかを Index にするときは PK を組み合わせたりする
* 4 ページ目みたいな取り方は超辛い

### JOIN HINT

* APPLY_JOIN
  * 値をぶつけあわせて探す
  * Index があるときは基本これ
* HASH_JOIN
  * Hash table を作成して探す
  * データ量が多いと hash table 作るのに時間がかかる
  * Index 使えないやつとかで Join するときに使う
* LOOP_JOIN
  * 全探索して探す
  * 2重ループ
* 種類を省略したらだいたい APPLY_JOIN になってる気がする

### INTERLEAVE

* 親子関係があるテーブルを同じ場所に保存しておく機能
  * 受注と受注明細みたいな
* ほとんどのケースで JOIN して一緒に使うテーブルを作るときに使う
* 実質 JOIN されてる

### サブクエリによる順序指定

```sql
JOIN ...
:
WHERE Price > 10000
ORDER BY Price DESC
LIMIT 100
```

JOINしてから100件に絞ってる

```sql
FROM
  (SELECT ...
   FROM ...
   WHERE Price > 10000
   ORDER BY Price DESC
   Limit 100) O
JOIN User1K U ON O.UserId = U.UserId
ORDER BY Price DESC
```

100件取得した後、JOINされる

### 順序を指定したい気持ちになるやつ

* JOIN
* フィルター
* ソート
* フィルターの重みが異なるやつ
  * 正規表現
  * Index があるものへのフィルター
* クエリの実行方法は BigQuery に似てる

### クエリの実行プラン

* [スライド](https://docs.google.com/presentation/d/1XKaOrex3WS8xZ0TsjsTQKBgxjVSGxkyqdcIqG66zd64/edit#slide=id.g40b4598186_6_359)がわかりやすい
* MySQL の気持ちは捨てよう

### Timestamp Bounds

* パフォーマンスやバックアップのためにタイムスタンプや x 秒以内みたいな指定をしてデータを取れる
* マルチリージョン
  * 最新の結果がいらないなら指定するとパフォーマンスが上がる
* バックアップ
  * 特定のタイムスタンプ時点のデータを取得できる

### Unit test

* エミュレータが無いので Unit Test が超辛い
* Unit Test 用の spanner を立ち上げたりしてる
* CircleCI からアクセスする時は US に置いたりした
* DB の作成、テーブルの作成は Spanner は速くない
* DB を作ったときに時間かかったりするのでDB を先に作っておいて空いてる DB を割り当てみたいな仕組みが欲しい
* DB 作ったり消したりすると 裏で大量に Split ができて新しく DB を作れなくなったりする
  * Node に 2 TB 入らない問題
  * 消しても後ろにまだ残ってる
  * Node ごとに作り直したりも有効

### データのコンパクション

* Spanner は暇なときにデータのコンパクションをしてくれる
* データサイズがちょくちょく減る

## GCPUG Shared Spanner Release

* Spanner は世界で GCP にしかない
* 試すにはGCP で Spanner を立ち上げるしか無い
* 毎月 7 万くらい
* Spanner を適当に試せる場として提供する
  * [Link](http://spanner.gcpug.jp/)

## インターリーブ実践入門

* [スライド](https://docs.google.com/presentation/d/1O78-6XKQH737lGySwAEX_LNLVlGmGmZlFWGMhbItPJ4/edit)

### インターリーブとは

* 親子関係を定義
* データの局所性を担保
* パフォーマンスの安定、向上が期待できる
  * 子レコードを参照する際にリモート通信が発生しない
  * 親子は同じ Split に入る
* 子の挿入先が親テーブルになる
* インターリーブインデックス
  * インデックスへのアクセスも同じ Split で完結する
  * write するときも同じ Split にあるので速くなる

### 制限

* 後からルートテーブルに変更不可
* 複数の親はダメ
* 最大6階層
* 親に紐づくレコードサイズが最大 4GB
  * 1 Split の制限

### インターリーブで性能改善させてみる

* インターリーブされてないと別の Node にいることがあるのでやっぱ遅くなる
* 性能改善が見込める
* 親に紐づく形で参照するケースには超有用

### インターリーブで性能劣化させてみる

* 親を横断して子を参照しに行くケースで性能劣化する？
* わざとフルスキャンする SQL を実行
* データ量に応じて差があるけど性能劣化はする
  * 20 万レコードのテーブルだと 70 倍の時間がかかった

### まとめ

* インターリーブは正しく使うとめっちゃ強力、速い
* 参照のされ方次第では性能劣化する
* インターリーブは**不可逆**なので注意して設計する
* **実際に性能検証すること**
  * Spanner は Node 課金なので中でいくら DB を作ろうが自由
  * 検証するだけならあんまりお金はかからない
