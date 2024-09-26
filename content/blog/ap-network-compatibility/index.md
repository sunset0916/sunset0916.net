---
title: "ネット回線の相性が悪いとPingが通ってもActivityPubの配送は通らない"
description: "なんのためのプロトコルだよ"
date: 2024-09-26T22:02:11+09:00
draft: false
tags: [blog, misskey]
categories: [blog]
summary: "なんのためのプロトコルだよ"
---

## はじめに

アクセスできるのにアクセスできない…

私は自宅でとあるMisskeyサーバーを運営しているのですが、一時期から特定のサーバーに配送ができない問題が発生し、困ったことがありました。

今回はその現象と改善策についてです。

## 問題の内容

どのような問題が起こったのか内容を書いていきます。

- 特定のサーバーに配送ができない
- pingは通る(DNSは解決されている)
- 他サーバー経由で拡散された投稿が特定のサーバーに到達すると一時的に配送が通る
- 疎通が確認されても少し時間があくとすぐ配送が止まる

こんな感じで謎が多い状態でした。

ソフトウェア的な問題があれば配送自体ができないだろうし、相手サーバーにアクセスできないのであればpingは通らないはず。

何が起きているか確認するためにエラーの内容を確認しました。

```plaintext
AbortError: The operation was aborted.
    at abort (file:///home/misskey/misskey/node_modules/.pnpm/node-fetch@3.3.2/node_modules/node-fetch/src/index.js:70:18)
    at EventTarget.abortAndFinalize (file:///home/misskey/misskey/node_modules/.pnpm/node-fetch@3.3.2/node_modules/node-fetch/src/index.js:89:4)
    at [nodejs.internal.kHybridDispatch] (node:internal/event_target:807:20)
    at EventTarget.dispatchEvent (node:internal/event_target:742:26)
    at abortSignal (node:internal/abort_controller:369:10)
    at AbortController.abort (node:internal/abort_controller:391:5)
    at Timeout._onTimeout (file:///home/misskey/misskey/packages/backend/built/core/HttpRequestService.js:118:24)
    at listOnTimeout (node:internal/timers:573:17)
    at process.processTimers (node:internal/timers:514:7)
```

うーんタイムアウト。

ということは通信周りの問題か？

謎は深まるばかりです。

## 試したこと

以下全て試しても改善しなかったものです。

### 自動投稿BOTで配送し続ける

一時的に配送が通るようになったときにしばらく配送をしないと止まることがわかっていたので、配送し続けたらどうかと試しましたが、それでもしばらくしたら止まり、意味はありませんでした。

### DNSサーバーを変えてみる

使用するDNSサーバーを変えてみましたが、そもそもDNSはちゃんと解決されていたので変わりませんでした。

### ソフトウェアを更新してみる

もしかしたらソフトウェアの問題の可能性もあるかもと最新版がリリースされたタイミングでアップデートしましたが、ソフトウェアの問題ではなかったようで変わりませんでした。

### サーバー機を変えてみる

インストールされたOSの問題や、サーバー機本体の問題の可能性も考え、サーバー機そのものも変えてみましたが、関係なかったようで変わりませんでした。

## 解決策

いろいろ試した結果、使用しているネット回線の相性が悪いのではないかという仮説が立ち、それを検証するために外部のサーバーにフォワードプロキシサーバーを構築しました。

使用したサーバーはOCI無料枠のVM.Standard.E2.1.Microで、Squidを使用しています。

配送を外部サーバー経由にした結果、配送が止まることが一切なくなり、これらの問題はあっさり解決してしまいました。

## おわりに

いろいろと頭を悩ませていた問題でしたが、相性という一番納得がいかない結果に終わりました。

なんのためのプロトコルだよと思いつつ、こんなこともあるんだとブログに書き起こした次第です。

今回の問題のおかげでOCIに手を出すこともできたのでポジティブにとらえていこうと思います。

それでは、また次回。
