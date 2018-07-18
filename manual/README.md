# Manual

## 記述方法

記述は原則 Markdown を使用。markdownlint の適用を必須とします。

Visual Studio Code + markdownlint の使用を推奨します。

[markdownlint - Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint)

## GitHub の運用フロー

GitHub Pages を適用するブランチは `master` ブランチです。よって `master` ブランチから `feature` ブランチを作成し、Pull Request 経由でマージする運用とします。

`master` ブランチには protect を付与し、Pull Request の merge にはレビュー者による approve を必要とします。

## ディレクトリ構造

`勉強会`、`AWS` などのカテゴリごとにディレクトリを作成し、その中に Markdown ファイルを格納する構造とします。

Markdown 内で画像などを使用する際はカテゴリのディレクトリ内に `img` ディレクトリを作成し、そこから参照を行うようにしてください。

ディレクトリ構造の例を以下に記載します。

```text
.
├── README.md
├── aws
│   ├── aws.md
│   └── img
│       └── aws.png
├── manual
│   └── README.md
└── study_group
    ├── img
    └── study.md
```

## ディレクトリ / ファイル の命名規則

キャメルケースではなく、原則スネークケースを利用することとします。また、名前には意味のない単語は使用しないでください。

> TODO: prefix / suffix の有無

## フォーマット

Markdown ファイルの文章フォーマットは厳密には定義せずフリーフォーマットとします。 ただし、markdownlint の適用のみ必須です。
