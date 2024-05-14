---
title: "macOS上のVSCodeで(たぶん)一番シンプルにgccを使ってコンパイル・実行する方法"
description: "Clangなんてコンパイラ、知らない！"
date: 2024-04-01T16:59:27+09:00
draft: false
tags: [blog, programming]
categories: [blog]
---

## はじめに

C言語の開発環境を整えたくなることありますよね。

しかしmacOSでは、デフォルトの状態で`gcc`や`g++`がClangというコンパイラへのリンクとなっており、gcc用に書かれたコードをコンパイルしようとするとライブラリが足りずにコンパイルできないなんてことになります。

gccを入れシンボリックリンクを置いてClangからコマンドの宛先を奪い取っても良いんですが、これではシンプルと呼べないので今回はVSCodeの拡張機能だけでサクッとコンパイル・実行できるようにしていきます。

## 前提

今回は以下のような状態であることを前提とします。

- Apple SiliconのMac
  - macOS 14 Sonoma
- VSCode
  - Code Runner
- Homebrew(arm64)
  - gcc

VSCodeに拡張機能の[Code Runner](https://marketplace.visualstudio.com/items?itemName=formulahendry.code-runner)を入れておきます。

[Homebrew](https://brew.sh/ja/)はRosettaを使わない状態で入れ、[gcc](https://formulae.brew.sh/formula/gcc)をインストールしておきます。

## 設定

前提のところまで環境ができたら拡張機能の設定でいい感じ™にコンパイル・実行できるようにしていきます。

はじめに、Code Runnerの設定アイコンかVSCodeの設定から`Code-runner: Executor Map`の項目を出し、`settings.json`を開きます。

すると、以下のようなjsonの構造があると思うので、

```json
"code-runner.executorMap": {
},
```

ここに以下のようにgccのパスを含んだコマンドを入れます。

```json
"c": "cd $dir && /opt/homebrew/bin/gcc-14 $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",
"cpp": "cd $dir && /opt/homebrew/bin/g++-14 $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",
```

私の環境ではこうなっています。

```json
"code-runner.executorMap": {
    "c": "cd $dir && /opt/homebrew/bin/gcc-14 $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",
    "cpp": "cd $dir && /opt/homebrew/bin/g++-14 $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",
    "python": "python3 -u"
},
```

`/opt/homebrew/bin/gcc-14`はインストールしたgccのパスで、その他の部分はCode Runnerのデフォルトの設定になっています。

この部分はCode Runnerのコマンドをカスタマイズする方法などで調べると書き方が出てくると思うので自由に使いやすいように変更すると良いと思います。

最後に`Command + S`で保存するのをお忘れなく。

## コンパイル・実行

設定が終わればあとは実行するだけです。

画面右上の実行ボタンから`Run Code`をクリックまたは`Ctrl + Option + N`でgccを使ったコンパイル・実行ができるはずです。

## おわりに

今回はmacOS上のVSCodeを使った環境でgccでコンパイル・実行する方法でした。

この方法であればデフォルトのClangの部分を一切変更せずにVSCode上ではHomebrewで入れたgccを使って開発を進めることができます。

ネットで調べるとパスを通すだとかシンボリックリンクを配置するだとか初心者に優しくない方法が上位に出てきますが、これなら簡単で環境を壊すこともないのでおすすめできると思います。

VSCode以外でもgccを使いたい場合は気合で`/opt/homebrew/bin/gcc-14`を打つかパス通してなんとかするかしてください。

それでは、また次回。
