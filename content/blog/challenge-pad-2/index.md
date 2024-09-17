---
title: "チャレンジパッド2を弄る話"
description: "今更！？！？！？"
date: 2024-09-03T17:26:45+09:00
draft: false
tags: [blog, gadget]
categories: [blog]
summary: "今更！？！？(2024年9月)"
---

## はじめに

今更！？！？(2024年9月)

先日、妹の部屋を掃除していたところ解約済みのベネッセの教材が大量に発掘され、そこにチャレンジパッド2が眠っていました。

確かこれ改造できたよな…と電源を入れたところ普通に動いたので今回はこれで遊んでいきます。

## 改造手順

改造とは言ってもやることはAPKファイルから野良アプリを入れてAndroidタブレットとして使えるようにするだけです。

もっと高度なことを期待したオタクの君はここでブラウザバックしてどうぞ。

APKファイルを入れるだけとは言っても少し厄介なところがあるので順を追って解説していきます。

今回使用する端末の情報

- チャレンジパッド2 (TAB-A03-BR2)
- ビルド番号 `02.04.000 release-keys`

### 初期化

初期化手順はベネッセ公式がわかりやすく解説しているのでそれを見てください。  
[https://www.benesse.co.jp/member/helpdesk/close/process.html](https://www.benesse.co.jp/member/helpdesk/close/process.html)

自分の使ってる機種がわからないよーって人にも親切なベネッセさん流石ですね。  
[https://faq.benesse.co.jp/faq/show/40515](https://faq.benesse.co.jp/faq/show/40515)

初期化後はスタートボタンを絶対に押してはいけません。

ランチャーのアプリ一覧ボタンから設定アプリを起動してください。

### Wi-Fiに接続

Anrdoid 5.1なのでクイック設定(上部からスワイプ)からWi-Fiに接続できます。

方法はここで解説するまでもないので割愛。

### USBデバッグ許可

設定→タブレット情報→ビルド番号 を7回連打して開発者向けオプションを有効化します。

その後、一つ前の画面から開発者向けオプションに入り、USBデバッグを有効にします。

このとき、必要に応じて「スリープモードにしない」の項目を有効にします。

### 提供元不明アプリを許可

USBデバッグを有効化できたらPCと接続し、ADBコマンドで弄っていきます。

提供元不明のアプリを許可するには以下のコマンドを実行すれば良いみたいです。

```shell
adb shell content update --uri content://settings/secure --where 'name=\"install_non_market_apps\"' --bind value:i:1
```

…が、場合によっては以下のようなエラーでこれが使えないことがあります。

```plaintext
Error while accessing provider:settings
android.database.sqlite.SQLiteException: no such column: install_non_market_apps (code 1): , while compiling: UPDATE secure SET value=? WHERE name=install_non_market_apps
    at android.database.DatabaseUtils.readExceptionFromParcel(DatabaseUtils.java:181)
    at android.database.DatabaseUtils.readExceptionFromParcel(DatabaseUtils.java:137)
    at android.content.ContentProviderProxy.update(ContentProviderNative.java:568)
    at com.android.commands.content.Content$UpdateCommand.onExecute(Content.java:597)
    at com.android.commands.content.Content$Command.execute(Content.java:417)
    at com.android.commands.content.Content.main(Content.java:605)
    at com.android.internal.os.RuntimeInit.nativeFinishInit(Native Method)
    at com.android.internal.os.RuntimeInit.main(RuntimeInit.java:249)
```

この状態でデータベースに登録されている設定項目を出してみると…

```plaintext
Row: 0 _id=2, name=mock_location, value=0
Row: 1 _id=3, name=backup_enabled, value=0
Row: 2 _id=4, name=backup_transport, value=android/com.android.internal.backup.LocalTransport
Row: 3 _id=5, name=mount_play_not_snd, value=1
Row: 4 _id=6, name=mount_ums_autostart, value=0
Row: 5 _id=7, name=mount_ums_prompt, value=1
Row: 6 _id=8, name=mount_ums_notify_enabled, value=1
Row: 7 _id=9, name=accessibility_script_injection, value=0
Row: 8 _id=10, name=accessibility_web_content_key_bindings, value=0x13=0x01000100; 0x14=0x01010100; 0x15=0x02000001; 0x16=0x02010001; 0x200000013=0x02000601; 0x200000014=0x02010601; 0x200000015=0x03020101; 0x200000016=0x03010201; 0x200000023=0x02000301; 0x200000024=0x02010301; 0x200000037=0x03070201; 0x200000038=0x03000701:0x03010701:0x03020701;
Row: 9 _id=11, name=long_press_timeout, value=500
Row: 10 _id=12, name=touch_exploration_enabled, value=0
Row: 11 _id=13, name=speak_password, value=0
Row: 12 _id=14, name=accessibility_script_injection_url, value=https://ssl.gstatic.com/accessibility/javascript/android/AndroidVox_v1.js
Row: 13 _id=15, name=lockscreen.disabled, value=1
Row: 14 _id=17, name=screensaver_activate_on_dock, value=0
Row: 15 _id=18, name=screensaver_activate_on_sleep, value=1
Row: 16 _id=19, name=screensaver_components, value=com.google.android.deskclock/com.android.deskclock.Screensaver
Row: 17 _id=20, name=screensaver_default_component, value=com.google.android.deskclock/com.android.deskclock.Screensaver
Row: 18 _id=21, name=accessibility_display_magnification_enabled, value=0
Row: 19 _id=22, name=accessibility_display_magnification_scale, value=2.0
Row: 20 _id=23, name=accessibility_display_magnification_auto_update, value=1
Row: 21 _id=25, name=immersive_mode_confirmations, value=
Row: 22 _id=26, name=install_non_market_apps, value=0
Row: 23 _id=27, name=wake_gesture_enabled, value=1
Row: 24 _id=28, name=lock_screen_show_notifications, value=1
Row: 25 _id=29, name=lock_screen_allow_private_notifications, value=1
Row: 26 _id=30, name=sleep_timeout, value=-1
Row: 27 _id=31, name=user_setup_device_owner, value=0
Row: 28 _id=32, name=default_input_method, value=jp.co.omronsoft.iwnnime.ml/.standardcommon.IWnnLanguageSwitcher
Row: 29 _id=33, name=enabled_input_methods, value=jp.co.omronsoft.iwnnime.ml/.standardcommon.IWnnLanguageSwitcher:com.google.android.googlequicksearchbox/com.google.android.voicesearch.ime.VoiceInputMethodService:com.google.android.inputmethod.japanese/.MozcService
Row: 30 _id=34, name=android_id, value=96da7351650db297
Row: 31 _id=35, name=input_methods_subtype_history, value=
Row: 32 _id=36, name=selected_input_method_subtype, value=-1
Row: 33 _id=37, name=lock_screen_owner_info_enabled, value=0
Row: 34 _id=38, name=user_setup_complete, value=1
Row: 35 _id=39, name=show_note_about_notification_hiding, value=0
Row: 36 _id=40, name=trust_agents_initialized, value=1
Row: 37 _id=41, name=bluetooth_name, value=TAB-A03-BR2
Row: 38 _id=42, name=bluetooth_address, value=AC:3F:A4:D6:C8:C3
Row: 39 _id=43, name=bluetooth_addr_valid, value=1
Row: 40 _id=44, name=screensaver_enabled, value=0
Row: 41 _id=45, name=location_providers_allowed, value=
Row: 42 _id=46, name=spell_checker_enabled, value=1
Row: 43 _id=47, name=accessibility_captioning_enabled, value=0
Row: 44 _id=48, name=high_text_contrast_enabled, value=0
Row: 45 _id=49, name=accessibility_display_inversion_enabled, value=0
Row: 46 _id=50, name=accessibility_display_daltonizer_enabled, value=0
Row: 47 _id=51, name=bluetooth_hci_log, value=0
Row: 48 _id=52, name=usb_audio_automatic_routing_disabled, value=0
Row: 49 _id=53, name=anr_show_background, value=0
```

`install_non_market_apps`が無い！

困りました。これでは設定を有効化できません。

ということでエラーが発生した場合は末尾に設定項目を自分で追加します。

以下の2つのコマンドのどちらかで追加できるはずです。

```shell
adb shell content insert --uri content://settings/secure --bind name:s:install_non_market_apps --bind value:i:1
```

```shell
adb shell settings put secure install_non_market_apps 1
```

### ランチャーのインストール

デフォルトのランチャーでは設定アプリしか起動できないため適当なランチャーをインストールします。

一番シンプルで無難なNova LauncherかLauncher3あたりを入れておきましょう。

Nova Launcherの場合は`6.2.19`がAndroid 5.1対応の最終バージョンなのでバージョンに気をつけて入れてください。

インストールする際はPCから

```shell
adb install APKファイルのパス
```

で入れましょう

### ストアのインストール

Playストアを入れたいところですが、入れても満足に動かなかったので諦めてAurora StoreやAPK Pureなどのサードパーティのストアを入れておきます。

個人的にはAurora Storeが一番まともに使えそうかなといった感じ。

ストアを入れたらブラウザやファイルマネージャーなどを入れておきます。デフォルトのものでは不便なので。

## やってもやらなくてもいいこと

ここまでくればもう比較的普通のAndroidタブレットとして使える状態になりましたが、まだ細かいところが気になるのでそこを弄っていきます。

### Play開発者サービスを入れる

こちらのリポジトリを使用するとGApps関連がインストールされます。

{{< github repo="Kobold831/EnableGPlayWithPC" >}}

ただ、インストールできてもPlayストアからのインストールやGoogle設定の起動などがうまく動きません。

Aurora Storeなどとの併用でアプリの動作率を上げていくために導入するといった感じですかね。

ちなみにこのツールWindows専用です。うーん微妙！

### その他

- ADBでベネッセ製アプリを強制削除
- ステータスバーのバッテリーアイコン内に残量を%で表示
- 時間を24時間表記にする
- パスワードを設定する
- etc…

## おわりに

今更！？！？って感じでしたね。

感圧式タッチパネルが使いにくいです。

使い道とやってほしいこと募集中です。

ちなみにサンセットは某事件でベネッセから図書カードもらってます。

それでは、また次回。
