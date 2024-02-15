---
title: "ドスパラのWinタブ(Windowsなし)で遊んだ話"
description: "Winタブ？よし通れ。ただしWindows、テメーはダメだ"
date: 2023-09-09T16:51:42+09:00
draft: false
tags: [blog, pc]
categories: [blog]
---

## はじめに

Windowsタブレット、良いですよね。  
小さい本体にすべてが詰まっていて、性能は低いけど最低限動いてくれる。  
仕事もコーディング動画視聴も頑張れば出来なくはない。  
遊びがいありそうだし1台くらいは持っててもいいかなぁと。

〜ある日ドスパラ中古にて〜

Atom x5-Z8350/4GB/64GBなWinタブが2200円。  
よし通れ。ただしWindows、テメーはダメだ。

## 購入したもの

今回購入したWindowsタブレットはこちら

| 分類 | 内容 |
| --- | --- |
| 型番 | DG-D08IW2SL |
| CPU | Atom x5-Z8350 |
| メモリ | DDR3L 4GB |
| UEFI | 32bit |
| ストレージ | eMMC 64GB (microSDカードスロットあり) |
| 画面サイズ | 8インチ |
| 画面解像度 | 1280 * 800 |
| USB端子 | micro-B x1 (充電通信兼用) |
| 画面出力 | microHDMI x1 |
| 通信 | IEEE802.11ac, Bluetooth 4.0 |
| OS | なし |

はい。よくある安いWindowsタブレットって感じのスペックですね。  
実用は微妙だけどおもちゃとしてなら丁度いいくらいの。  
あと32bitUEFI、お前だけは絶対に許さない。

OSは無しです。ストレージは完全消去済みでマザボにライセンスもなかったのでWindowsを使う場合はライセンスのみ別途用意が必要です。

付属品完品で新品未使用(開封して消去済み)の状態で税込み2200円でした。  
いいおもちゃになりそうです。

## 入れたOSと結果

32bit UEFI, 64bit CPUな環境で64bit版がインストールできるOSの中で個人的に好意を持っているOSを試しました。  
32bit UEFI非対応のUbuntuはカスなので滅んでいいよ(頑張ればできるらしいが)

### Debian 12

結論から書くと微妙でした。

タッチパネルで使えそうなデスクトップ環境はGNOMEとKDEかなということで両方試しましたがDebian自体が微妙でしたね。  
デスクトップ環境はGNOMEの方がタッチパネル向けでした。

Debianが微妙だった理由は以下の通り。

- バックライトが制御できない
- スリープが正常に動作しない
- オーディオドライバがない(non-freeなリポジトリを有効化すれば入る)
- 画面回転がズレてる(ファイル追加により修正可)
- カメラ使用不可(カーネルビルドすれば使えるかも)

スリープ出来ないんじゃタブレットPCとしておしまいということでDebianは却下となりました。  
でもGNOMEの可能性は感じた。それは次に試すOSの話。

### Fedora Workstation 38

Windows以外だとこれが一番まともかもしれない。

画面回転とカメラの問題はDebianと共通ですが、バックライトが問題なく動いてスリープできるのでタブレットとしては合格ラインかなと。

あとはデフォルトのデスクトップ環境がGNOMEなのでスクリーンキーボードや画面自動回転ボタンなどが用意されててタブレット端末に優しい。

まあこんな感じで今はFedoraに落ち着いています。

## 画面自動回転の問題を修正する

GNOMEやKDEはiio-sensor-proxyというパッケージから得た値で画面を回転させています。  
メジャーな機種は物理的な画面の方向に合わせるための設定が書かれていますが、ドスパラのような日本国内のみに展開するハードは設定がありません。  
デフォルトの設定では90度ズレていたりするため、設定を追加して修正します。

具体的には以下の手順を踏むだけです。

最初にファイルを作成

```shell
sudo nano /usr/lib/udev/hwdb.d/61-sensor-local.hwdb
```

以下を記述

```plaintext
sensor:modalias:acpi:*:dmi:*:svnThirdwaveCorporation:*
 ACCEL_MOUNT_MATRIX=0, -1, 0; -1, 0, 0; 0, 0, 1
```

最後に設定を読み込んで再起動

```shell
sudo systemd-hwdb update
sudo reboot
```

## 今後試したいこと

今後やりたいこととして、カメラを使えるようにしたいと思っています。  
CherryTrail世代のAtom系SoCはLinux用のドライバが少なく、カメラの機能に関してはカーネル側でデフォルトで無効にされています。  
カーネルをビルドすることでいけるかもしれないので一度試そうかと考えています。

あとは他のOSも試しておきたいですね。  
Android-x86とかRemixOSとかChromeOSとか…  
あとは危ないやつだけどTiny10とか。

また試して分かったことがあればブログにするので気長に待ってもらえたらなと。

## おわりに

2200円にしては遊びがいのあるコスパの良いおもちゃでした。  
何か使い道を見つけたいですがiPadもChromebookも持ってるのでおもちゃ以上に昇格することは無いんだろうなと。

試してほしいOSとかあれば連絡ください。  
できれば普通じゃないOSでお願いします。

それでは、また次回。