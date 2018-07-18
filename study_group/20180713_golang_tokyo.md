# golang.tokyo #16

[connpass](https://golangtokyo.connpass.com/event/92225/)

## WebAssemblyとGoの対応状況について by tenntenn

* WebAssembly
  * ネイティブに近い早さ(JSより早い)
  * ブラウザで実行できるバイナリ形式の言語
* AutoCADが使ってるらしい
* 言語の対応状況
  * C/C++
  * Rust
  * Lua
  * とか
  * Go もあるよ
* Go の対応状況
  * 1.11 でリリースされる
  * ゴルーチンとチャネルには対応
  * JS の API 呼び出し
    * syscall/jsを用いる
    * 雰囲気的にはリフレクションパッケージに似てる
* wasm を試す
  * Go の最新コード取ってきてビルド
  * GOOS と GOARCH を指定してビルドする
  * GOOS = js, GOARCH = wasm
* DOM の操作とイベント
  * おおよその Go の標準パッケージは使える
  * 静的解析する Go パッケージも使える
* HTTP アクセス
  * net/http パッケージがうごく
  * fetch が用いられる

## GopherJS vs Wasm (仮) by hajimehoshi

* Go で Web アプリを書きたい
  * GopherJS
  * GOARCH=wasm
* GopherJS
  * 非公式
  * Go -> JS に変換する
  * 読みやすい出力
  * 速度重視ではない
  * JS に変換するくせに Goroutine などほぼ全機能サポート
  * 細かいバグが結構ある
  * メイン作者がwasmの人と同じ
  * wasm のほうに注力していてあんまり活動が活発じゃない
* wasm
  * 公式!
  * フロントエンドが共通 SSAまで同じ
  * 変なバグがない
  * syscall/js パッケージを使う
  * Goroutine など全部サポート
  * wasm 自体にはGCやスレッドがない
  * スタックをヒープにおいてエミュレート
  * 1.11でサポート予定
  * まだAPIや機能が変化し続けてる
* 似て異なるAPI
  * GopherJS
    * *js.Object
  * wasm
    * js.Value
    * 関数はjs.Value
* 速度比較
  * GopherJS
    * 55-60fps
  * wasm
    * 15-20fps
  * wasmが遅い
  * 単精度浮動小数点数ベンチマーク
    * それでもwasmはJSより倍ぐらいおそい……
    * C++のwasmは早い
  * 倍精度浮動小数点数にしても同じだった……
* 考察
  * ChromeのJSが早すぎる or wasmが遅い
  * Goの最適化不足
  * もっとベンチマークしたい
* バイナリサイズ
  * GopherJSでも500KBくらいある
  * GoのwasmはMB普通にいく
  * C++のwasmが25Kでやばい
  * wasm-optによる最適化は必須

[GopherJS vs GOARCH=wasm](https://docs.google.com/presentation/d/e/2PACX-1vQLOcSY-SpdWedMT48QFZ8f9T_XojfqUOCgMg4jqIz8cJjFIJhHm98gHKVyMaboqGpsXCfedplT-lmp/pub?start=false&loop=false&delayms=3000&slide=id.p)

## Goで社内向け管理画面を楽に作る方法

[Goで社内向け管理画面を楽に作る方法](https://speakerdeck.com/yudppp/godeshe-nei-xiang-keguan-li-hua-mian-wole-nizuo-rufang-fa)

## Goのスライス容量拡張量がどのように決まるのか追った

[Goのスライス容量拡張量がどのように決まるのか追った / 180713 LT](https://speakerdeck.com/kaznishi/180713-lt)

## Go言語の正規表現に後読みを実装した話

[Go言語の正規表現に後読みを実装した話](https://slides.com/makenowjust/regexp-lookbehind-in-golang)
