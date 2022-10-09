<!--
title:   Markdown（マークダウン）をVSCodeの拡張機能とスニペットで効率良く書く
tags:    Markdown,VSCode,VisualStudioCode,マークダウン,マークダウン記法
id:      1310d3f0aeb24f393b88
private: false
-->
# Markdownをはやく効率よく書くために

前提として、マークダウンを編集するエディタは**VSCode**を使います。

https://azure.microsoft.com/ja-jp/products/visual-studio-code/

Markdownファイルを使って記事を管理している場合、**マークダウンを効率良く書けるかどうかは生産性に直結**します。

VSCodeの拡張機能と基本設定（スニペット）を使ってマークダウンを速やかに編集する方法について書いていきます。

よく使う機能にフォーカスして**その操作をイメージできる一般的なショートカットキーを使う（覚える）のがポイント**です。

# 拡張機能があるならそれを使う

## Markdown All in One

ショートカットや便利なコマンドが有効になる拡張機能です。たくさんの機能がありますが、利用頻度が高いものだけ使います。

https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one

### 太字にするショートカット

テキストを選択した状態で、Macなら `⌘` + `B` 、Windowsなら `Ctrl` + `B` で太字になります。多くのエディタで採用されているショートカットなので、これはよく使います。

![markdown_all_in_one_b.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/35152/69595741-c1dd-82a8-688a-e0544a3f9917.gif)

他にも、見出しレベルの上げ下げをするショートカットもあります。見出しはよく使いますが、`#` の増減だけです。慣れていない人は、ショートカットコマンドを覚えなくても直接テキスト編集すれば十分です。

### リスト入力の補完

リスト入力した状態で改行すると、連続入力ができます。インデントも考慮されます。`Tab` でリストの階層を下げる（`Shift` + `Tab` で上げる）こともできます。直感的に操作できて、とても便利です。

![markdown_all_in_one_list.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/35152/f4545542-5454-c61a-43e6-c686f3246d99.gif)

### テキスト選択状態でURLをコピペしてリンク表記に変換

テキストを選択した状態でクリップボードにコピーされたリンク（URL形式の文字列）を貼り付けます。すると、自動でリンク表記に変換してくれます。これも便利なのでよく使います。

![markdown_all_in_one_url.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/35152/447f4f5f-eb57-a8ba-72fb-6f202386a463.gif)

## Paste Image

画像を簡単に挿入できるようになる拡張機能です。

https://marketplace.visualstudio.com/items?itemName=telesoho.vscode-markdown-paste-image

### 画像保存と画像挿入記述をコピペ作業で実現

Markdownファイルに画像を挿入するのは結構手間がかかります。事前に画像を格納しておいて、その画像ファイルへのパスを代替テキストと共に指定しなければなりません。この面倒な作業をコピペで実現できる拡張機能です。

画像ファイルをコピーした状態で、Macでは `⌘` + `alt` + `V` 、Windowsなら `Ctrl` + `alt` + `V` で、以下の操作を同時にやってくれます。

1. 指定フォルダへの画像保存
2. Markdownファイルへの画像挿入記述

画像を多用するときは、無いとやってられないくらいに重宝します。

![markdown_paste_image.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/35152/694b1289-5925-d24f-411d-3dd849accb6a.gif)

1つ注意点として、Macでコピーする場合、画像を開いていない状態だとFinderから画像ファイルを選択してコピーしても正しく動作しません。この問題についてはGithubに関連するissueがあがっていました。少し面倒ですが、一度画像を開いてプレビュー表示してからコピーする必要があります。

https://github.com/mushanshitiancai/vscode-paste-image/issues/46

## :emojisense:

絵文字入力が楽になる拡張機能です。

https://marketplace.visualstudio.com/items?itemName=bierner.emojisense

### Unicode絵文字の入力補完

`:` を入力すると一般的な絵文字入力で使われるUnicode絵文字が絵文字コード付きで入力補完されます。たとえば「😀」を入力したい場合は `:grinning:` を選びます。

![markdown_emojisense_input.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/35152/fe4d590e-6db3-6236-5079-b21db6204fa8.gif)

さらにMacでは `⌘` + `I` 、Windowsでは `Ctrl` + `I` でUnicode絵文字ピッカーが使えます。少しややこしいですが、絵文字コードで入力したい絵文字を検索して、そのままUnicode絵文字が入力できます。

![markdown_emojisense_picker.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/35152/ec7e23e4-4d2c-197b-c37f-93a0b88677be.gif)

### 絵文字コードのプレビュー表示

他にも、絵文字コードをMarkdown上で入力すると該当の絵文字をプレビュー表示してくれます。また、絵文字コードを入力したい場合は `::` で補完されます。

:::note info
絵文字コードは `:` で囲まれた一意の絵文字を表すコードです。テキストとして入力すると表示されるタイミングで対応する絵文字に変換されます。たとえば `:smile:` は「😄」に変換されて表示されます。注意点として、QiitaやGithubをはじめ絵文字を表示する側が対応している必要があります。
:::

## Insert Date String

現在時刻の文字列をショートカット入力できるようになる拡張機能です。

https://marketplace.visualstudio.com/items?itemName=jsynowiec.vscode-insertdatestring

### 指定形式で日付文字列を入力補完

日付文字列をショートカットで入力できるようになります。対応するショートカットはMacでは `⌘` + `Shift` + `I`、Windowsでは `Ctrl` + `Shift` + `I` です。

:::note warn
注意点として、ショートカットキーが競合するとうまく動かなくなります。1つ前に説明した:emojisense:の絵文字Picker表示のショートカットと競合します。（`⌘` + `Shift` + `I` にも `⌘` + `I` と同じ絵文字Pickerが割り当てられています）こちらを使いたい場合は、VSCodeのショートカットキー設定から:emojisense:のキーバインドを削除またはしてください。
:::

日付のフォーマットは設定で変更可能です。たとえば、ISO 8601のタイムゾーン付きの形式（例：`2022-8-27T12:34:56+09:00`）にできます。

![markdown_insert_date_string.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/35152/34f5b10c-da58-593a-df9b-ffc6ab4ccdcc.gif)

## Copy file name

### ファイル名を右クリックメニューからコピー

右クリックでファイル名のコピーが一発でできるようになる拡張機能です。

https://marketplace.visualstudio.com/items?itemName=nemesv.copy-file-name

VSCodeは標準だと右クリックでパスコピーしかできないので、ファイルを選択してENTERキーを押して「名前の変更」からファイル名をコピーしている人は多いかもしれません。

拡張子あり・なしのいずれのファイル名も一発コピーできるので、地味に便利です。

## Path Intellisense

### ファイル名・パスの入力を補完

ファイル名やファイルパスを予測してオートコンプリートしてくれます。

https://marketplace.visualstudio.com/items?itemName=christian-kohler.path-intellisense

コーディング時と比べると利用機会はそれほど多くないですが、説明のためにファイル名やファイルパスを書くときに重宝します。

# 拡張機能がなければスニペット登録

拡張機能を使えない場合は、VSCodeのスニペットを活用します。VSCodeの基本設定で使える機能です。

VSCodeのプロジェクト設定として `.vscode/markdown.code-snippets` にスニペットを登録をすれば、プロジェクト固有の設定として、スニペット入力ができます。たとえば、テーブル表記をスニペット登録する場合は以下のように設定します。`table` とタイピングすれば、入力候補に表示されて選択できます。

```json
{
  "table": {
    "prefix": "table",
    "body": [
      "| header1 | header2 | header3 |",
      "| ------- | ------- | ------- |",
      "|         |         |         |",
      "|         |         |         |"
    ],
    "description": "テーブル（Markdown形式）"
  }
}
```

一般的でない特殊表記を使う場合、スニペット登録しておけば呼び出してすぐに入力できます。

https://github.com/waicode/blueblog/blob/main/.vscode/markdown.code-snippets

# Markdownの正しい構文を自動チェックする

関連して、markdownlintとhuskyとlist-stagedを組み合わせてマークダウンの構文をコミット時にチェックする方法を以下で書いてます。

https://qiita.com/waicode/items/33311d0a511dc821f53f

# Markdownでブログ記事を管理している実例

実際の技術ブログでマークダウンのブログ記事を効率良く書いて確認する方法を以下にまとめています。こちらも合わせて参考にしてみてください。

https://archt.blue/articles/markdown