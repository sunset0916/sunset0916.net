---
title: "HugoにAdventarのカレンダーを埋め込む"
description: "ショートコードをもっと活用していきたい"
date: 2025-12-13T19:00:00+09:00
draft: false
tags: [blog]
categories: [blog]
summary: "この記事はしょぼねこ Advent Calendar 2025 13日目の記事です"
---

{{< alert "circle-info" >}}
この記事は[しょぼねこ Advent Calendar 2025](https://adventar.org/calendars/12131) 13日目の記事です。
{{< /alert >}}

{{< adventar url="https://adventar.org/calendars/12131" >}}

## はじめに

はじめましての方ははじめまして、そうでない方はこんにちは。サンセット([@sunset@mi.sunset0916.net](https://mi.sunset0916.net/@sunset))です。

自宅鯖や変なスマホ収集、プログラミング、お絵かきなどをしたりしなかったりする美少女 **[要出典]** です。

さて、noteにはAdventarの埋め込みがあるのですが、他のサービスや個人サイトではなかなか見かけないことに最近気付きました。

軽く調べたところ、埋め込みURL自体は存在しているみたいなのですが、note向けに作成されたもので一般向けでは無いようです。

せっかく埋め込み用ページがあるのなら、自分のサイトにも埋め込んでやろうというのが今回のネタです。

## 仕様を探る

AdventarのURLは以下のような構造になっています。

```plaintext
https://adventar.org/calendars/{カレンダーのID}
```

noteで表示されているHTMLを取り出すとこんな感じ。

```html
<iframe
    allowtransparency="true"
    class="fude-iframe-oembed-widget__iframe"
    frameborder="0"
    scrolling="no"
    src="https://adventar.org/calendars/{カレンダーのID}/embed"
    loading="lazy">
</iframe>
```

重要なのは`src`の部分で、`/embed`が付いているだけの簡単なURLとなっています。

任意のカレンダーに`/embed`を付けてアクセスするとわかりやすいのですが、URLに少し追記するだけで埋め込み用ページを表示することができます。

この仕様であれば、少し弄るだけで様々なサイトに移植できそうですね。

## Shortcodeとは

このブログはHugoというSSG(静的サイトジェネレーター)で作られています。

HugoとはMarkdownとテーマを使用してサイトを作成するGo製のSSGです。

HugoにてMarkdownからHTMLを作るにあたって、テーマやユーザー自身で独自の要素を定義して使用することができます。

これがShortcodeです。

Hugoには組み込みのShortcodeも多くあり、Twitter(X)の投稿の埋め込みなどが代表的です。

例えば、以下のような投稿があったとします。

```plaintext
https://x.com/sunset09160306/status/1484190310041546752
```

これをShortcodeで埋め込むとこうなります。

```plaintext
{{</* x user="sunset09160306" id="1484190310041546752" */>}}
```

実際に出力されたものがこちら。

{{< x user="sunset09160306" id="1484190310041546752" >}}

このような感じで、URLを入れれば簡単にカレンダーを埋め込めるShortcodeを作ります。

## 作った

早速完成品を出します。

```html
<div style="position: relative; width: 100%; margin-top: 2em;">
  <iframe src="{{ .Get "url" }}/embed" width="620" height="362" frameborder="0" loading="lazy" style="width: 100%;"></iframe>
</div>
```

はい、これだけです。  
iframeで持って来るだけですからね。

こちらを、`/layouts/shortcodes/adventar.html`として保存します。

呼び出し方は簡単で、末尾の`/`が無いURLを入れるだけです。

```plaintext
{{</* adventar url="https://adventar.org/calendars/12131" */>}}
```

そして出力されたものがこちら。

{{< adventar url="https://adventar.org/calendars/12131" >}}

## おわりに

今回はアドカレの埋め込みをHugoに実装しました。

以下のサイトの内容が参考になりました。  
他サービスやSSGへの埋め込みをする際は、こちらも参考にしてみてください。

[https://kivantium.hateblo.jp/entry/2021/12/18/034810](https://kivantium.hateblo.jp/entry/2021/12/18/034810)

明日はろむねこが何か書いてくれるみたいです。

それでは、また次回。
