<!--
title:   markdownlintを使ってVSCode上で構文チェックしてhuskyとlint-stagedでコミット時にも解析する
tags:    Markdown,VSCode,husky,lint-staged,markdownlint
id:      33311d0a511dc821f53f
private: false
-->
# 不正なMarkdownをリポジトリに混入させないために

静的解析（lint）をMarkdownにも適用させれば、eslintやstylelintのように**マークダウンの構文チェック**ができます。

VSCodeのエディタ上で確認しながら編集して、huskyとlint-stagedを使ってコミット時にも確認すれば、**誤った構文のマークダウンがリポジトリに入り込む余地が一切無くなり**ます。

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
    "line-length": {
      // MD013: 1文の最大文字数を調整（デフォルトは80）
      "line_length": 120
    },
    "no-duplicate-heading": false, // MD024: 見出し文字列の重複を許容
    "no-trailing-punctuation": false, // MD026: 見出しに.,;:が入ることを許容
    "no-inline-html": false, // MD033: HTML記述を許容
    "no-bare-urls": false // MD034: URLそのままの表記を許容
  }
}
```

# 静的解析（lint）でマークダウンの構文をチェックする

markdownlintを使ってコミット時に構文チェックすれば、不正なマークダウンがコミットされることを防ぐことができます。

JavascriptやTypescriptのファイルにeslint、CSSやSCSSのファイルstylelintをあてるように、Markdownファイル（`.md`）に対して**コミット時にmarkdownlintをコマンドで適用してチェック**します。

## markdownlint-cli2

コマンド（CLI）でチェックするなら `markdownlint-cli2` がオススメです。

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

### 設定ファイルの調整

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

```.markdownlint-cli2.jsonc
{
  "config": {
    "line-length": {
      // MD013: 1文の最大文字数を調整（デフォルトは80）
      "line_length": 120
    },
    "no-duplicate-heading": false, // MD024: 見出し文字列の重複を許容
    "no-trailing-punctuation": false, // MD026: 見出しに.,;:が入ることを許容
    "no-inline-html": false, // MD033: HTML記述を許容
    "no-bare-urls": false // MD034: URLそのままの表記を許容
  },
  "ignores": ["node_modules"]
}
```

https://github.com/waicode/blueblog/blob/main/front/.markdownlint-cli2.jsonc

### huskyとlint-stagedでプレコミット時に確認

huskyを使うとコミットやプッシュ時に、任意のコマンドを自動で実行できます。これに加えてlint-stagedを使うと、ソースコード全体ではなく `git stage` されたファイルに対してlintできます。

これらを使って、**コミット時にステージ対象のファイルに対してmarkdownlint-cli2のコマンドを実行**します。

#### huskyとlint-stagedのインストール

huskyとlint-stagedが入ってない場合、`npm` または `yarn` でインストールします。

```sh
npm install --save-dev husky lint-staged
```

```sh
yarn add -D husky lint-staged
```

#### huskyを有効化

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

#### huskyのpre-commitファイルを作成

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

#### lint-stagedの設定ファイルを作成

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

#### 不正なMarkdownはコミットできなくなる

ここまでの設定できたら、コミット時にmarkdownlintで構文チェックされるようになります。不正な構文が含まれたファイルをコミットした場合、警告メッセージが出てコミットできません。

![markdown_husky.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/35152/c5bbc19f-54cc-c280-c2d7-3487f9464ced.gif)

## Markdownの自動整形（format）はあえてやらない

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
