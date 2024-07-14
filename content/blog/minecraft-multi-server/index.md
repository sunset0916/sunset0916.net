---
title: "Cloudflareのドメインと1台の自宅サーバーで複数のMinecraftサーバーをホストする"
description: "欲は常に新しい技術や知識を得るきっかけになる…はず。"
date: 2024-07-14T22:15:57+09:00
draft: false
tags: [blog]
categories: [blog]
---

## はじめに

自宅サーバーの登竜門とも言えるMinecraft。

サーバーを構築するだけなら簡単ですが、細かい要望に応えようとすると設定が難しいのもMinecraftサーバーの特徴。

例えば、2つのサーバーをポートを変えずにドメインの違いだけで分けたいとか。

今回はタイトルのような感じのMinecraftサーバーを構築する手順です。

## 要件

手順に移る前に今回構築したいサーバーの要件をまとめます。

- OSは安定のDebian12
- サバイバル用サーバーと試作クリエイティブ用サーバーの2つを運用したい
- 開放するポートは25565のみで、ドメインごとにサーバーを割り振りたい
- 動的IPの回線でCloudflareのDNSを使用したドメインを使いたい
- Minecraftサーバーの起動・終了はsystemdを使いたい
- Minecraftサーバーのコンソールはちゃんと触りたい

なんとまあ贅沢な望みだという感じですが、ちゃんと実現できます。

人によってどこまで必要か変わってくると思うので部分ごとに解説していきます。

## systemdでMinecraftサーバーを管理し、screenでコンソールにアクセスする

systemdで運用する関係でコンソールへのアクセスはscreenを使うという方法です。

Minecraftサーバーの建て方はわかってる前提で書くのでわからない人は勉強してから来てくださいね。

ということでsystemdの設定ファイルから。`{}`で囲まれた部分は各自の環境に合わせて変えてください。

```shell
[Unit]
Description={Minecraft Server Description}
After=network-online.target

[Service]
Type=forking
User={userName}
Group={groupName}
WorkingDirectory={/path/to/server}
ExecStart=screen -S {screenName} -d -m java -Xmx2G -jar {/path/to/server/server.jar} --nogui
ExecStop=screen -S {screenName} -X quit
Restart=always

[Install]
WantedBy = multi-user.target
```

これを2つ分作ってMinecraft側でポートを変えれば2台をたてるのはすぐできますね。

Javaの起動設定は任意で変えましょう。

コンソールを触るときは

```shell
screen -r {screenName}
```

で入り、出るときは`Ctrl + A -> D`で抜けます。

## 2つのMinecraft鯖に通信を割り振る

[itzg/mc-router](https://github.com/itzg/mc-router)というものを使用してドメインごとにサーバーを割り振ります。

これは`git clone`して`docker-compose.yml`書いて起動するだけなので設定ファイルだけ。

こちらも`{}`で囲まれた部分は環境に合わせて変えて下さい。

```yml
version: "3.8"

services:
  router:
    image: itzg/mc-router:latest
    environment:
      API_BINDING: ":25564"
    ports:
        - 25565:25565
        - 25564:25564
    restart: always
    command: --mapping={mc1.example.com}={server1-ip:port},{mc2.example.com}={server2-ip:port}
```

お気づきの方もいるかもしれませんが、これサーバーが同じでも別でもどこにあっても通信割り振りできます。お得ですね。

mc-routerの動作確認は

```shell
curl -H Accept:application/json localhost:25564/routes
```

でできます。

ここまででローカルからはhostsを書けばドメインごとに通信割り振りがされるサーバーが完成しました。

hostsでmc-routerに対してドメインを2つ向けて動作確認してみましょう。

## CloudflareのDNSを使うドメインを動的IPで使用する

外部ツールを使ったりなどいくつか方法はありますが、今回はシェルスクリプトでAPIを叩いて更新する方法を使います。

必要な情報は以下の5つです。

- Cloudflareアカウントのメールアドレス
- アカウントのAPIトークン
- ドメインのゾーンID
- ドメイン1のレコードID
- ドメイン2のレコードID

これらはWebダッシュボードまたはcurlで取得することができます。

これを以下のシェルスクリプトに入れてcronで自動実行すれば自動更新は完成です。

毎度のことですが`{}`で囲われた部分は正規表現を除いて(ry

```shell
#!/bin/bash

# 更新するドメイン
MY_DOMAIN1="{mc1.example.com}"
MY_DOMAIN2="{mc2.example.com}"

# API アクセス情報
ZONE_ID="{zone-id}"
RECORD_ID1="{record-id-1}"
RECORD_ID2="{record-id-2}"
AUTH_EMAIL="{mail@example.com}"
API_TOKEN="{API-token}"

# IPアドレス取得サービスのリスト
services=(
  "https://api.ipify.org/"
  "https://checkip.amazonaws.com/"
  "https://ipv4.icanhazip.com/"
  "https://4.icanhazip.com/"
)

ip_address=""

# IPアドレスを取得する関数
get_ip_address() {
  local url="$1"
  ip_address=$(curl -s "$url")
}

# IPアドレスの正規表現パターン
ip_pattern="^([0-9]{1,3}\.){3}[0-9]{1,3}$"

# 正規表現でIPアドレスが検証できるかチェック
is_valid_ip() {
  [[ $ip_address =~ $ip_pattern ]]
}

# IPアドレス取得のループ
for service in "${services[@]}"; do
  get_ip_address "$service"

  if is_valid_ip; then
    break
  fi
done

# ipアドレス更新テスト
# ip_address = 8.8.8.8

echo ""
echo "IP address : ${ip_address}"
echo ""

echo "{mc1.example.com} result"
echo "---------------------------------------------------------"

# CloudFlare API Aレコード更新
curl --request PUT \
  --url https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records/$RECORD_ID1 \
  --header 'Content-Type: application/json' \
  --header "X-Auth-Email: $AUTH_EMAIL" \
  --header "Authorization: Bearer $API_TOKEN" \
  --data '{
    "content": "'"$ip_address"'",
    "name": "'"$MY_DOMAIN1"'",
    "proxied": false,
    "type": "A",
    "comment": "{minecraft server 1}"
  }'

echo ""
echo "---------------------------------------------------------"
echo ""

echo "{mc2.example.com} result"
echo "---------------------------------------------------------"

# CloudFlare API Aレコード更新
curl --request PUT \
  --url https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records/$RECORD_ID2 \
  --header 'Content-Type: application/json' \
  --header "X-Auth-Email: $AUTH_EMAIL" \
  --header "Authorization: Bearer $API_TOKEN" \
  --data '{
    "content": "'"$ip_address"'",
    "name": "'"$MY_DOMAIN2"'",
    "proxied": false,
    "type": "A",
    "comment": "{minecraft server 2}"
  }'

echo ""
echo "---------------------------------------------------------"
echo ""
```

わからない人は`Cloudflare DDNS`とか`Cloudflare API ドメイン更新`とかで調べると出てくるのでその辺見てやってください。

## おわりに

今回はサンセットがMinecraftサーバーでやりたかったことを実現するための方法を雑にまとめる記事でした。

意外と奥深いMinecraftサーバーの世界、これに少しは入っていけたのかなと思います。

今回の記事はとても雑なので不明点など聞きたいことがあればSNSのDMなどで連絡ください。

それでは、また次回。
