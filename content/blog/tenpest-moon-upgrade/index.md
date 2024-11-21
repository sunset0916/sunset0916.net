---
title: "32bitでDockerでMisskey v12なサーバーを64bitでネイティブ動作の最新版までアップグレードした話"
description: "アプデはサボらない方がいいし、わけわからんならDockerは使わないほうがいい"
date: 2024-03-19T12:30:39+09:00
draft: false
tags: [blog, misskey]
categories: [blog]
summary: "誰のサーバーかは言ってないけどモザイク貫通でバレバレらしい"
---

## はじめに

アプデ、面倒ですよね。

Misskeyなどのアップデートなんかは特に、他に使用するパッケージのアップデートまで要求してくるので。

これが大きな問題になったのがMisskey v12からv13になったとき。  
PostgreSQLとNode.jsのアップデートが必要になり、今回の事例のようなアプデをサボる原因になってしまいました。

今回SOSを受けてアプデ作業をした環境はこちら。

- Kona Linux 32bit (32bit機からHDDだけ移動させたらしい)
- Misskey v12.119.2
- Docker Compose
  - Node.js v16.15.1
  - PostgreSQL v12.2
  - Redis v4.0

v12をそのままDockerで動かしたような構成ですね。  
これを一気に更新して以下のようにしました。

- Debian 12 64bit
- Misskey v2024.3.1
- Node.js v20.11.1
- PostgreSQL v15.6
- Redis v7.2.4

いやー最新っていいですね。ここまでやれば当事者も今後はアプデ作業をしてくれることでしょう。

今回はDockerから非Docker環境への移行とアップデートの手順をまとめる回になります。

## 移行対象データの取り出し

まずは旧環境から移行対象のデータを取り出します。

必要なデータは以下のとおり。

- `.config/default.yml`
- `files/`
- データベースのバックアップ

任意ですが、Redisのバックアップを取ると移行後に溜まっていたキューなどを失わずに動かすことができます。  
無くても一部サーバーに一部投稿が届かないだけなので、気にしない方はやらなくても問題なく移行できます。

`default.yml`に関しては最悪無くても気合で思い出して設定すれば大丈夫ですが、ID設定など、途中からデフォルト値が変更されたものの中で移行時に変更してはいけないものがあるため、ある状態で移行するのが望ましいと思います。

データベースのバックアップは2通りで、クラスタ全体のバックアップと特定データベースのバックアップがあります。

サーバーを運用するくらいの人であれば以下のコードブロックを見れば理解してくれることでしょう。

```shell
# とりあえず全部落とす
docker-compose down

# PostgreSQLのコンテナのみ起動する
docker-compose up -d db

# docker psなどで確認したコンテナ名またはIDのシェルに入る
docker exec -it コンテナ名 /bin/bash

# どこでやってもいいけどとりあえずわかりやすく/tmpで作業
cd /tmp

# pg_dumpで移行する場合(一度起動してDBの構造を作ったりする場合)
pg_dump -Fc -U example-misskey-user -d misskey > backup.dmp

# pg_dumpallで移行する場合(データベースの構造そのものを移行する場合)
pg_dumpall -U example-misskey-user > backup.sql

# コンテナのシェルから出る
exit

# docker cpコマンドでコンテナ内のファイルをホストにコピーする
docker cp db:/tmp/backup.xxx ~/

# もう使わないのでコンテナは落とすなりなんなりと
docker-compose down
```

はい、こんな感じです。  
`docker exec`で初めからホストにDBのダンプをすれば良くない？って疑問については、なんかバグるからやめといたほうがいいよって感じで思ってもらえたら。

<details>
<summary>上記についてたぶんこうだろうって考察</summary>

Dockerコンテナから直接ホストOSにファイルを保存するという行為は、場合によっては言語や文字コードが異なる環境のシェルに標準出力として流してしまうことになります。

Misskeyのデータベースには投稿などが含まれますが、この投稿はアルファベットだけでなく日本語や特殊文字なども含まれています。

これを、言語の設定などが違う可能性のあるDockerコンテナから直接自身のホストOSのシェルに流してリダイレクトすると文字化けなどの不具合の原因になる可能性が高いと推測します。

これを回避するには、一度Dockerコンテナ内のシェルでファイルの保存までを行ってから`docker cp`コマンドでファイルを持ってくるのが確実だと考え、これを採用しました。

正しいかわからんのでなんか良い考察や情報があれば教えてほしい。

</details>

## 移行先環境のセットアップ

データが救出できたら次は移行先のセットアップです。

とりあえず最新環境にすればってわけにもいかなさそうなのでまずは移行前のバージョンであるv12が動くようにセットアップしていきます。

最初に入れるのは以下のパッケージ

- Node.js v18.x
- Redis v7.x
- PostgreSQL v15.x
- FFmpeg
- build-essential

Node.js以外は最新Misskeyに対応したものになります。

Node.jsだけ最新でない理由として、v12を動かすための`yarn install`が正常に動いてくれないことなどが挙げられます。  
Node.jsのバージョンは最後、Misskeyを最新版にするタイミングでv20に上げます。

Misskeyのインストールはまあわかると思うのでそのままやってください。

今回ははじめにv12最終盤を入れるので、`git checkout a5a74f4`でバージョン指定をしていきます。  
今後はこのコミットIDを変えて少しずつ安全にバージョンアップをしていきます。

`yarn install`ができたら`files/`と`.config/default.yml`を配置します。

このときの注意点として、docker用と通常の`default.yml`では設定する項目が異なる点があります。  
PostgreSQLとRedisは`localhost`に、その他項目も説明を読みながら変更していきます。

次にデータベースを戻します。

`pg_dumpall`を使った場合はMisskeyの起動前にDBを`psql`でリストアして起動します。

```shell
sudo su postgres
psql -f backup.sql
```

`pg_dump`を使った場合はDBの構造を用意してから`misskey`データベースに対して`pg_restore`でリストアします。

```shell
pg_restore --clean -U example-misskey-user -d misskey backup.dmp
```

これで起動すれば問題なく起動できるはずです。

systemdのサービスのファイルを用意するのも忘れずにね。

## 最新版までアップデートする

まあここはもう普通にアップデートするだけなんで大丈夫なんですけど、いきなり最新まで上げたりするのは怖いので現実的にちょっとサボった人がアプデしたかなってくらいの間隔でアップデートしました。

最初にv13初期バージョンに上げますが、ここでsystemdのサービスのファイルを更新するのをお忘れなく。  
同時にメモリアロケーターの設定を入れておくとメモリリーク対策になるのでおすすめです。

Node.jsのアップデートはv2023.12.0のアプデ直前くらいにすれば問題ないです。(あまり早い段階でアプデすると起動しないかも)

そうこうしているうちに最新版になりましたね。ね？

はいアプデおしまい！終わり！

## おわりに

アプデはサボらない方がいいし、わけわからんならDockerは使わないほうがいい

って思いました。以上！

それでは、また次回。