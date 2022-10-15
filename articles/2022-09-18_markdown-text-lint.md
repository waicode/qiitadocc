<!--
title:   VSCodeでマークダウン構文チェックと文章校正してコミット時にも静的解析（lint）する
tags:    Markdown,VSCode,markdownlint,textlint,cspell
id:      33311d0a511dc821f53f
private: false
-->

# 不正な構文や誤字脱字をリポジトリに混入させないために

マークダウンに書く文章はコンパイルもなければ、実際に読まれるときの動的なチェックもできません。

そのため、マークダウンテキストを**事前に解析することは「糸くず（lint）」を取る以上に重要な役割**を占めると考えています。

静的解析をMarkdownにも適用させれば、eslintやstylelintのように**マークダウンの構文チェック**や**文章の校正**ができます。

これらのチェックは、VSCodeの拡張機能を使うことで**問題があればエディタ上で編集しながらリアルタイムで確認**できます。

| VSCodeの拡張機能 | 説明 |
| ---- | ---- |
| [markdownlint](https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint) | VSCodeのエディタ上でマークダウン構造を解析 |
| [vscode-textlint](https://marketplace.visualstudio.com/items?itemName=taichi.vscode-textlint) | VSCodeのエディタ上でテキストを解析 |
| [Code Spell Checker](https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker) | VSCodeのエディタ上で英単語の誤字をチェック |

もちろん、CLIでの確認も可能です。

| ライブラリ | 説明 |
| ---- | ---- |
| [markdownlint](https://github.com/DavidAnson/markdownlint) | マークダウンの構文をチェックする |
| [textlint](https://github.com/textlint/textlint) |設定や辞書に従い文章を校正する |
| [cspell](https://cspell.org/) |英単語の誤字をチェックする |

さらに `husky` と `lint-staged` を使えば、コミット時に変更対象ファイルだけコマンドを使って自動でチェックできます。

| ライブラリ | 説明 |
| ---- | ---- |
| [husky](https://typicode.github.io/husky/#/) | Gitコミット時に任意のコマンドを組み込むことができる |
| [lint-staged](https://github.com/okonet/lint-staged) | Gitのステージに上がっているファイルを対象に処理を実行できる |

VSCodeで確認しながら編集して、コミット時も保険的に確認すれば、**誤った構文のマークダウンがリポジトリに入り込む余地が一切無くなり**ます。

# VSCodeの拡張機能でlintする

## markdownlint

markdownlintの拡張機能をインストールすると、VSCodeでマークダウンの構文がチェックできます。

https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint

書き方に問題がある場合、VSCodeのエディタ上に波線が表示されて警告されます。

![markdown_lint.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/35152/30cbdbcf-d7c1-df8a-1042-822319b2129c.gif)

markdownlintは拡張機能をインストールすればすぐ使えます。

ですが、**デフォルトの構文チェックはかなり厳しめ**に設定されています。

そのため、[markdownlintのルール](https://github.com/DavidAnson/markdownlint/blob/main/doc/Rules.md)を確認して `markdownlint.config` の設定を調整してから使うのが現実的です。

`.vscode/settings.json` にmarkdownlintの設定を書いておけば、プロジェクト固有の設定として有効になります。

以下は調整した設定の一例です。

```jsonc
{
  "markdownlint.config": {
    "line-length": false, // MD013: 1文の最大文字数をはtextlintと競合するため無効化
    "no-duplicate-heading": false, // MD024: 見出し文字列の重複を許容
    "no-trailing-punctuation": false, // MD026: 見出しに.,;:が入ることを許容
    "no-inline-html": false, // MD033: HTML記述を許容
    "no-bare-urls": false // MD034: URLそのままの表記を許容
  }
}
```

## vscode-textlint

vscode-textlintの拡張機能で、VSCodeのエディタ上で文章の校正ができます。

https://marketplace.visualstudio.com/items?itemName=taichi.vscode-textlint

校正の設定は `.textlintrc` で行います。CLIと設定ファイルを共有できるので、詳細は後述します。

## Code Spell Checker

英単語のスペルチェックのためにCode Spell Checkerの拡張機能をインストールします。

https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker

設定ファイルは `.cspell.json` です。こちらも、後述するCLIと同じ設定ファイルが使えます。

# CLIで静的解析（lint）する

## markdownlint-cli2

マークダウン構文をコマンド（CLI）でチェックするなら `markdownlint-cli2` がオススメです。

https://github.com/DavidAnson/markdownlint-cli2

従来から利用されていたmarkdownlintの伝統的なCLIである `markdownlint-cli` でも良いのですが、新しく登場した `markdownlint-cli2` の方が設定ベースで調整しやすく、VSCodeとうまく連動するように設計されています。

### markdownlint-cli2のインストール

`npm` または `yarn` でインストールします。

```sh
npm install --save-dev markdownlint-cli2
```

```sh
yarn add -D markdownlint-cli2
```

### lint実行とスクリプト登録

インストール後、以下のコマンドでlintできます。

```sh
markdownlint-cli2 "**/*.md"
```

実際には `package.json` の `scripts` に登録して使うのが通例です。以下を追記します。

```package.json
"scripts": {
  "lint:markdown": "markdownlint-cli2 \"**/*.md\""
}
```

### markdownlint-cli2の設定ファイル調整

ただし、このままではすべてのマークダウンファイルに対してすべてのルールが対象となるため、設定ファイルで調整します。

プロジェクトルートに `.markdownlint-cli2.jsonc` を作成すれば設定が読み込まれます。

:::note info
`.jsonc` はコメントが書けるJSONフォーマットです。(JSONC = JSON with comments)
:::

:::note info
ちなみに、VSCode設定ファイルのJSONは暗黙的にJSONCで解釈されます。そのため `.vscode/settings.json` をはじめとする `.json` 拡張子の設定ファイルにコメントを書いてもエラーになりません。
:::

設定ファイルは複数のフォーマットに対応していますが、VSCodeと揃えるために `.jsonc` がおすすめです。

`config` の項目には**VSCode拡張機能のmarkdownlintの設定をそのまま記載**できます。

また、解析対象から外したいファイルやフォルダは `ignores` で設定します。

```jsonc:.markdownlint-cli2.jsonc
{
  "config": {
    "line-length": false, // MD013: 1文の最大文字数をはtextlintと競合するため無効化
    "no-duplicate-heading": false, // MD024: 見出し文字列の重複を許容
    "no-trailing-punctuation": false, // MD026: 見出しに.,;:が入ることを許容
    "no-inline-html": false, // MD033: HTML記述を許容
    "no-bare-urls": false // MD034: URLそのままの表記を許容
  },
  "ignores": ["node_modules"]
}
```

https://github.com/waicode/blueblog/blob/main/front/.markdownlint-cli2.jsonc

## textlint

テキストの校正は `textlint` で行います。プリセットと組み合わせて使うことが通例です。

ここでは、ベースとして以下の2つのプリセットを適用する前提で手順を説明していきます。

| プリセット名 | 説明 |
| ---- | ---- |
| [textlint-rule-preset-ja-spacing](https://github.com/textlint-ja/textlint-rule-preset-ja-spacing) | 日本語のスペース有無を決定するtextlintルールプリセット |
| [textlint-rule-preset-ja-technical-writing](https://github.com/textlint-ja/textlint-rule-preset-ja-technical-writing) | 技術文書向けのtextlintルールプリセット |

また、他にも校正辞書やコメントでlintを無効化できるライブラリも一緒にインストールします。

| ライブラリ | 説明 |
| ---- | ---- |
| [textlint-rule-prh](https://github.com/textlint-rule/textlint-rule-prh) | textlintでprh（校正辞書）を使えるようにする |
| [textlint-filter-rule-comments](https://github.com/textlint/textlint-filter-rule-comments) | コメントでtextlintを部分的に無効化できる |

### textlintとプリセットのインストール

`npm` または `yarn` でインストールします。

```sh
npm install --save-dev textlint textlint-rule-preset-ja-spacing textlint-rule-preset-ja-technical-writing
```

```sh
yarn add -D textlint textlint-rule-preset-ja-spacing textlint-rule-preset-ja-technical-writing
```

### lint実行とスクリプト登録

インストール後、以下のコマンドでlintできます。

```sh
textlint "**/*.md"
```

`package.json` の `scripts` に登録する一例です。

```package.json
"scripts": {
  "lint:text": "textlint \"**/*.md\""
}
```

### textlintの設定を調整

プリセットの内容を調整したい場合は `.textlintrc` で設定を上書きします。VSCode拡張機能も、この設定を使います。

以下は調整した設定ファイルの一例です。

```json:.textlintrc
{
  "plugins": {
    "@textlint/markdown": {
      "extensions": [
        ".md"
      ]
    }
  },
  "filters": {
    "comments": true
  },
  "rules": {
    "prh": {
      "rulePaths": [
        "./prh/index.yml"
      ]
    },
    "preset-ja-technical-writing": {
      "sentence-length": {
        "max": 150
      },
      "no-exclamation-question-mark": {
        "allowFullWidthExclamation": true,
        "allowFullWidthQuestion": true
      },
      "ja-no-successive-word": false,
      "ja-no-mixed-period": {
        "allowPeriodMarks": [
          ":",
          "："
        ]
      },
      "no-doubled-joshi": {
        "strict": false,
        "allow": [
          "も",
          "や",
          "か"
        ],
        "separatorCharacters": [
          ",",
          "，",
          "、",
          ".",
          "．",
          "。",
          "?",
          "!",
          "？",
          "！",
          "「",
          "」",
          "\"",
          "”",
          "“"
        ]
      }
    },
    "preset-ja-spacing": {
      "ja-space-around-code": {
        "before": true,
        "after": true
      }
    }
  }
}
```

`./prh/index.yml` は校正辞書のインデックスです。

```yml:/prh/index.yml
imports:
  - ./rules/tech.yml
```

ここから、個別のルール設定をインポートします。カテゴライズして必要な辞書を増やしていくことを想定しています。

以下は「技術用語の固有名詞ルール」の校正辞書を定義した一例です。

```yml:/prh/rules/tech.yml
meta:
  - title: 技術用語の固有名詞ルール
rules:
  - expected: インターフェース
    patterns:
      - インターフェイス
      - インタフェース
      - インタフェイス
    prh: 技術用語
  - expected: ソフトウェア
    pattern: ソフトウェアー
    prh: 技術用語
  - expected: ハードウェア
    pattern: ハードウェアー
    prh: 技術用語
  - expected: デフォルト
    pattern: ディフォルト
    prh: 技術用語

```

また `textlint-filter-rule-comments` を有効化しているので、以下のようにコメントでtextlintの設定を部分的に無効化できます。

```md
<!-- textlint-disable -->

This is ignored text by rule.
Disables all rules between comments

<!-- textlint-enable -->
```

## cspell（Code Spell Checker）

cspellはVSCode拡張機能の「Code Spell Checker」をCLIでも使えるようにしたライブラリです。

https://cspell.org/

### cspellとプリセットのインストール

`npm` または `yarn` でインストールします。

```sh
npm install --save-dev cspell
```

```sh
yarn add -D cspell
```

### lint実行とスクリプト登録

インストール後、以下のコマンドでlintできます。

以下はすべてのファイルをチェックする例です。

```sh
cspell "**"
```

`package.json` の `scripts` に登録する一例です。

```package.json
"scripts": {
  "lint:cspell": "cspell \"**\" ."
}
```

### cspellの設定

設定ファイルは `.cspell.json`  です。VSCode拡張機能もこの設定ファイルをもとにスペルをチェックします。

```json:.cspell.jsonc
{
  "version": "0.2",
  "language": "en",
  // allowCompoundWords - set to true to allow compound words by default
  "allowCompoundWords": true,
  // words - list of words to be always considered correct
  "words": [
    "qiita",
  ],
  // flagWords - list of words to be always considered incorrect
  // "flagWords": [],
  // ignorePaths - a list of globs to specify which files are to be ignored
  "ignorePaths": [
    ".vscode",
    ".devcontainer",
    ".git",
    ".history",
    ".cspell.json",
    "node_modules",
    "package.json",
    "yarn.lock"
  ]
}
```

許可する言葉は `.cspell.json` の `words` に追加します。対象の文字列にカーソルを合わせてVSCodeのクイックフィックスで登録するのが便利です。

![VSCodeのクイックフィックスでcspellのwordsに登録](https://user-images.githubusercontent.com/3455992/194875399-0036c51e-bbd2-4bc4-928b-0790ae622b5e.gif)

# コミット時にステージ上のファイルだけチェックする

## huskyとlint-stagedでプレコミット時に確認

huskyを使うとコミットやプッシュ時に、任意のコマンドを自動で実行できます。これに加えてlint-stagedを使うと、ソースコード全体ではなく `git stage` されたファイルに対してlintできます。

これらを使って、**コミット時にステージ対象のファイルに対してmarkdownlint-cli2のコマンドを実行**します。

### huskyとlint-stagedのインストール

huskyとlint-stagedが入ってない場合、`npm` または `yarn` でインストールします。

```sh
npm install --save-dev husky lint-staged
```

```sh
yarn add -D husky lint-staged
```

### huskyを有効化

続いて、huskyのインストールコマンドを実行します。

```sh
npx husky install
```

```sh
yarn husky install
```

この操作で `.husky` ディレクトリが作成されます。

```sh
.husky
├── _
│   └── husky.sh
└── .gitignore
```

さらに `package.json` のscriptsにhuskyが有効化される設定を入れておきます。

`prepare` に `husky install` を記述します。

```package.json
"scripts": {
  "prepare": "husky install"
}
```

:::note warn
Monorepo（モノレポ）構成を採用している場合は、ルート直下の `package.json` に設定を入れます。
:::

:::note warn
yarn2を採用している場合は `prepare` でなく `postinstall` に記述します。プライベートパッケージと公開パッケージで記述が異なります。詳しくは[huskyの公式ドキュメント](https://typicode.github.io/husky/#/?id=yarn-2)を確認してください。
:::

### huskyのpre-commitファイルを作成

その後、コミット時にチェックするためpre-commitファイルを作成します。

```sh
npx husky add .husky/pre-commit "npm lint-staged"
```

```sh
yarn husky add .husky/pre-commit "yarn lint-staged"
```

以下の `.husky/pre-commit` ファイルが作成されます。

```.husky/pre-commit
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

# npmの場合
npm lint-staged
# yarnの場合
yarn lint-staged
```

Monorepo（モノレポ）の場合は、リポジトリ内のいずれかのプロジェクトで実行したいケースが多いでしょう。その場合、フォルダを移動するコマンドを追記します。

```.husky/pre-commit
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

# frontフォルダへ移動
cd front

# lintを実行
yarn lint-staged
```

https://github.com/waicode/blueblog/blob/main/.husky/pre-commit

これでコミット時に更新したファイルがlintされるようになりました。

### lint-stagedの設定ファイルを作成

最後にlint-stagedの設定ファイルで実行する静的解析（lint）を指定します。

以下は `.md` の拡張子のファイルに対してmarkdownlintを行う設定です。

``` .lintstagedrc.js
module.exports = {
  '*.{md}': ['yarn lint:markdown']
};

```

:::note warn
ここで記載している `yarn lint:markdown` は、前述のmarkdownlint-cli2を設定した際に `package.json` の `scripts` に自分で記述したコマンドです。
:::

lint-stagedの動作は `package.json` に書くこともできますが、別の設定ファイルに書いて `yarn` のコマンドを直接叩くのがオススメです。

設定ファイルは `.lintstagedrc.js` のように[lint-stagedのConfiguration](https://github.com/okonet/lint-staged#configuration)に記載されている設定ファイル名にすれば自動で読み込んでくれます。

### 不正なMarkdownはコミットできなくなる

ここまでの設定できたら、コミット時にmarkdownlintで構文チェックされるようになります。不正な構文が含まれたファイルをコミットした場合、警告メッセージが出てコミットできません。

![markdown_husky.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/35152/c5bbc19f-54cc-c280-c2d7-3487f9464ced.gif)

# Markdownの自動整形（format）はあえてやらない

静的解析（lint）だけでなく、自動整形（format）までやる考え方もあります。

VSCodeと一緒に使うフォーマッターはPrettierが有名です。

https://prettier.io/

自動整形の設定を入れておけば、たとえばファイルの保存時にフォーマッターへ設定した形式に自動的に補正してくれます。

markdownlint-cli2には自動修正モードで実行できるコマンド（`markdownlint-cli2-fix`）があるので、これも同義です。

ソースコード全体の形式を手間なく統一できるので、原則プログラムを動かすためのソースコードには自動整形（フォーマット）を適用させるべきだと考えます。

**しかし、Markdownファイルの自動整形はオススメできません。**

なぜなら、マークダウンはシステムのためでなく「人が読むために書かれること」が多いからです。

マークダウンはその歴史的背景からパーサの仕様が完全には統一されていません。[MDX](https://mdxjs.com/)や[MDC](https://content.nuxtjs.org/guide/writing/mdc/)など拡張構文も日々増えています。

そのため、自動整形を入れると**見た目が意図せず補正されて崩れるリスク**があります。Markdownファイルは静的解析（lint）にとどめ、自動整形（format）まではやらないのが無難です。

# MarkdownをVSCodeで効率良く書くTIPS

VSCodeの拡張機能とスニペットを活用してマークダウンを効率良く書くテクニックを以下でまとめています。無駄な機能は使わず、よく使う機能にフォーカスして使う（覚える）のがポイントです。

https://qiita.com/waicode/items/1310d3f0aeb24f393b88

# Markdownの構文チェックをブログ記事で実践している例

実際の技術ブログでマークダウンのブログ記事を効率良く書いて確認する方法を以下にまとめています。マークダウンの構文チェックについても書いています。合わせて参考にしてみてください。

https://archt.blue/articles/markdown
