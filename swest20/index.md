[LTEモデム搭載プロトタイプ向けデバイス 「Wio LTE」](https://soracom.jp/products/wio_lte/) を用いたハンズオンテキストです

# 概要

※本ハンズオンは90分程度を見込みます （開発環境の準備は+60分です)

## 手順

* [開発環境の準備](#setup)
* [ハンズオン](#main)
    * [Wio LTE ハードウェア解説](#wiolte)
    * [Wio LTE Wio LTE の電源 ON/OFF 方法](#onoff)
    * [Wio LTE の "通常モード" と "DFUモード"](#dfu)
    * [ステップ 1: LED点灯](#step1)
    * [ステップ 2: デバイス稼働時間を SORACOM Harvest で可視化](#step2)
    * [ステップ 3: 温湿度センサーのデータを SORACOM Harvest で可視化](#step3)
    * [ステップ 4: 温湿度センサーのデータを SORACOM Funnel 利用して AWS IoT Core へ転送](#step4)
    * [おわりに](#end)

## Appendix(付録)

* [ハンズオンで使用した環境の構築方法](#appendix)
    * [AWS IoT Core](#aws-iot-core)

# 準備するもの

* パソコン / 1台
    * ※タブレット等の下記に当てはまらないPCでの受講は自己責任としております
    * Wi-Fi 接続可能
    * USB Type-A ポートが最低1つ以上あり、利用可能なこと
        * 電力供給が1A以上であること (USB 3.0対応していれば概ね安心です)
        * USB Type-C のみの機種につきましては、Type-A への変換アダプタのご用意をお願いいたします
    * OS: macOS(10.11 El Capitan 以上) もしくは Windows(8.1 以上)
    * ブラウザ: Google Chrome
    * ※ソフトウェアをインストールするため、PCに対する管理者権限を持っている事 と ブラウザでのアクセス制限(HTTPプロキシ等)がかかっていない事
* 有効なSORACOMアカウント / 1つ
    * 持っていない場合: 有効なクレジットカード(1枚) と この場で確認可能なメールアドレス(1つ) を利用し [SORACOM アカウントの作成](https://dev.soracom.io/jp/start/console/#account) の手順に沿って作成します
* SORACOMアカウントに登録済みの SORACOM Air SIM (nanoサイズ) / 1つ
    * 未登録の場合: [Air SIM の登録](https://dev.soracom.io/jp/start/console/#registsim) の手順に沿って登録します
* Wio LTE (本体、 アンテナ2本、 電源兼シリアルコンソール用microUSBケーブル) / 1式
* Grove デジタル温湿度センサー (DHT11) / 1つ
* Grove コネクタケーブル / 1つ

## 受講における注意点    

* 本ハンズオンで利用するサービスには有料のものを含んでおります
* ハンズオンに伴い発生した費用（ハンズオン中、ハンズオン後問わず）は原則として受講者にご負担いただいております。係る費用について確認・ご理解いただいたうえでの受講をお願いしております
    *  [SORACOM Air 料金](https://soracom.jp/services/air/cellular/price/)、[SORACOM Harvest 料金](https://soracom.jp/services/harvest/price/)、[SORACOM Beam 料金](https://soracom.jp/services/beam/price/)、[SORACOM Funnel 料金](https://soracom.jp/services/funnel/price/)
* 本ドキュメントで発生した不具合等につきましては弊社は一切責任を負いません

<h1 id="setup">開発環境の準備</h1>

Wio LTE を使うためには、開発環境の準備を行います  
OS 毎に準備がありますので、下記を参照の上開発環境を準備してください

* [Windows 編](https://github.com/soracom/handson/wiki/Wio-LTE-%E9%96%8B%E7%99%BA%E7%92%B0%E5%A2%83%E6%A7%8B%E7%AF%89---Windows-%E7%B7%A8)
* [macOS 編](https://github.com/soracom/handson/wiki/Wio-LTE-%E9%96%8B%E7%99%BA%E7%92%B0%E5%A2%83%E6%A7%8B%E7%AF%89-macOS-%E7%B7%A8)

<h1 id="main">ハンズオン</h1>

<h2 id="wiolte">《知識》Wio LTE ハードウェア解説</h2>

Wio LTE は Seeed Studio が開発・販売しているマイコンモジュールです  
Grove コネクタと STM32F4 マイコン、LTE Cat.1 モジュールが搭載されており、 Arduino IDE で素早くプロトタイピングができます

![Wio LTE](https://soracom.jp/img/products_wio_lte.png)

### USB 電源・アンテナ・ Grove コネクタピンの番号

![Wio LTE 本体](https://docs.google.com/drawings/d/e/2PACX-1vSgTN46GBaOzqnnmg6tzc7S6UOvzefQhewbGrrRYCrn4RKz1_MMO77zPoCFisCRjztfmHYuac9VIQmN/pub?w=639&h=299)

### Grove IoT スターターキット for SORACOM

「Grove IoT スターターキット for SORACOM」は、 Wio LTE 本体に加え、７つのセンサーと SORACOM Air SIM (日本向け) と SORACOM クーポンが一つにパッケージされたキットです

本ハンズオンでは **デジタル温湿度センサー** を利用します

![Grove IoT スターターキット for SORACOM](https://docs.google.com/drawings/d/e/2PACX-1vT7i6svO5xJj56o0x1aUaW3qhkb4aCWynxSzbwVG3boCqe-TQooCC-qAm0Nfa3QBIb9YIum1G7Zkw-Z/pub?w=640&h=347)

<h2 id="onoff">《知識》Wio LTE の電源 ON/OFF の方法</h2>

Wio LTE には電源スイッチがありませんので、下記作業で ON / OFF してください

### 電源 ON

microUSB ケーブルを Wio LTE の microUSB ポートに接続すると自動的にONになります

![Wio LTE と PC を microUSBケーブルで取り付けたところ](https://docs.google.com/drawings/d/e/2PACX-1vSgLUCOrN928URIfbcNC0VR4xwSBOCYm8ngs0d2edkWyu4ZC7VNoXjALvKOXv121zk3RZB2vF9J40fB/pub?w=640&h=480)

### 電源 OFF

microUSB ケーブルを抜きます。いきなり抜いて OK です

※シャットダウン処理は存在しません

<h2 id="dfu">《知識》Wio LTE の "通常モード" と "DFUモード"</h2>

Wio LTE は２つのモードを持っています
**この操作は Wio LTE の開発で何度も行うことになりますので、必ず覚えてください**

* 書き込まれたプログラムを実行する「通常モード」
* プログラムを書き込むことができる「DFUモード」

![フロー](https://docs.google.com/drawings/d/e/2PACX-1vQAcnymqWTTneRwnc9EFz21YvrmfCsIuV33yfqf1ODC_LKQR-6762CJDMclRIWC8BfUeDDLpC6KKs-2/pub?w=581&h=253)

これらのモードの切り替えは Wio LTE 上の **RSTボタン** と **BOOTボタン** の組み合わせで行います  
各ボタンの位置は下記のとおりです（ Wio LTE の表裏にボタンがあるため、横からみた図で確認ください）

![Wio LTE を横からみた図](https://docs.google.com/drawings/d/e/2PACX-1vRnhRiZC7-jRCqLaxJO6E7Bmq0_8BxornXgP1y6UHdYXhr6iBm_RNoV148oSzJKeHBYXRjYai9msQoz/pub?w=480&h=249)

### Wio LTE の動作モードを変更する・確認する

Wio LTE の動作モードの確認の仕方は、環境構築の準備でご確認ください

* [Windows で Wio LTE の動作モードを変更する・確認する](https://github.com/soracom/handson/wiki/Wio-LTE-%E9%96%8B%E7%99%BA%E7%92%B0%E5%A2%83%E6%A7%8B%E7%AF%89---Windows-%E7%B7%A8#wio-lte-%E3%81%AE-%E9%80%9A%E5%B8%B8%E3%83%A2%E3%83%BC%E3%83%89-%E3%81%A8-dfu%E3%83%A2%E3%83%BC%E3%83%89)
* [macOS で Wio LTE の動作モードを変更する・確認する](https://github.com/soracom/handson/wiki/Wio-LTE-%E9%96%8B%E7%99%BA%E7%92%B0%E5%A2%83%E6%A7%8B%E7%AF%89-macOS-%E7%B7%A8#wio-lte-%E3%81%AE-%E9%80%9A%E5%B8%B8%E3%83%A2%E3%83%BC%E3%83%89-%E3%81%A8-dfu%E3%83%A2%E3%83%BC%E3%83%89)

<h1 id="step1">【作業】ステップ 1: LED点灯</h1>

Wio LTE ライブラリに付属しているスケッチ例から、内蔵のLEDを点灯してみましょう

※Wio LTE 開発ツールの使い方を学びます

![ステップ1 ovewview](https://docs.google.com/drawings/d/e/2PACX-1vQuxiAIWQlqEF0aIgIZtYGemOl7BFPnqKTU3WxKbyk3F8PG0zkhYBWUrVUpJyq8LDqzQwAiBMFUxPKD/pub?w=371&h=247)

## 最初に. Wio LTE の電源を OFF にする

Wio LTE の microUSB ケーブルを抜き、電源を OFF にしてください

※いきなり抜いてOKです。また、すでに OFF になっている場合は次に進んでください

## 1. スケッチを作成する

### 1-1. Arduino IDE を起動する

* Windowsの場合: スタートメニュー等から **Arduino** を起動します
* macOSの場合: アプリケーションフォルダ等の中から **Arduino** を起動します

### 1-2. メニューの [ツール] > [ボード: "Arduino/Genuino UNO] で表示される一覧から **Wio Tracker LTE** を選択

![](https://docs.google.com/drawings/d/e/2PACX-1vQKCIKzOA6NSb0-3kNvL5i9lpZSNAS5OXklLbFITCP2vHvEjM2gL3qKdo8WzYZjifjajFe3YovtiUEI/pub?w=333&h=507)

### 1-3. メニューの [ファイル] > [スケッチ例] > [Wio LTE for Arduino] > [basic] > [LedSetRGB]

### 1-4. Wio LTE を PC を接続して DFUモード にする

### 1-5. 新しく開いたウィンドウの ![マイコンボードに書き込むアイコン](https://docs.google.com/drawings/d/e/2PACX-1vQiO83cFcX3LCXeioiTiaao57T4SGiIV6XZzcBP6poTwssCxmo7hLpoMh5qG3btyqgzs8Q-lAoE6Q0f/pub?w=100&h=100)(マイコンボードに書き込む) をクリック

### 1-6. Arduino IDE のウィンドウ下部に、下記のように表示されたら書き込み完了です  

```
DFU end
can't detach

   もしくは

Resetting USB to switch back to runtime mode
DFU end
```

### 1-7. Wio LTE を 通常モードにする (RSTボタンを押せば通常モードになります)

## 2. 確認

下図のように Wio LTE 内蔵のLEDがカラフルに光ります  

![Wio LTE / LedSetRGB](https://dev.soracom.io/img/gs_wio-lte/wio-lte_led.gif)

ここまでの手順の動画です (画面は Windows ですが、 macOS でも同様の手順です)

[![Wio LTE / LedSetRGB](http://img.youtube.com/vi/g9LiH_g-TuE/0.jpg)](http://www.youtube.com/watch?v=g9LiH_g-TuE)

## 3. やってみよう (LEDの変更周期)

スケッチ※を編集して LED の色が変わる周期を変えてみましょう  
※Arduino IDE では、プログラムのことを「スケッチ」と呼びます

スケッチ3行目の `INTERVAL` が周期を指定しています。ミリ秒(ms)単位になっており、 50 から変更してみましょう

変更前

```
#define INTERVAL  (50)
```

変更後

```
#define INTERVAL  (10)
```

スケッチを変更したら、先ほどの 1-4 ～ 1-7 の手順を行って、LEDの発行色の周期が変わっていることを確認してみてください

## 4. Wio LTE の動作を止める

止める方法は Wio LTE の電源を OFF (= microUSBケーブルを抜く) してください

## うまく動かなかったら（トラブルシュート）

**[ファイル] > [スケッチ例] の中に "Wio LTE for Arduino" が表示されない**

* 原因: "ボード" が "Wio Tracker LTE" になっていません
* 対策: メニューの [ツール] > [ボード: XXXX] から **Wio Tracker LTE** を選択してください

**「マイコンボードに書き込む」を実行した結果、ウィンドウに下記のように表示された**

```
exit status 1
ボードArduino/Genuino Unoに対するコンパイル時にエラーが発生しました。
```

* 原因: "ボード" が "Wio Tracker LTE" になっていません
* 対策: メニューの [ツール] > [ボード: XXXX] から **Wio Tracker LTE** を選択してください

**「マイコンボードに書き込む」を実行した結果、ウィンドウに下記のように表示された**

```
No DFU capable USB device available
DFU end
```

* 原因: Wio LTE が「通常モード」で書き込もうとした
* 対策: Wio LTE を「DFU モード」にしてから、再度「マイコンボードに書き込む」を実行してください

**「マイコンボードに書き込む」を実行した結果、ウィンドウに下記のように表示された**

```
java.io.IOException: jssc.SerialPortException: ....
    ... 色々表示されて ...
    ... 4 more もしくは ... 6 more
```

* 原因: おもに Windows でシリアルモニタ―を表示した後に発生します。シリアルポートの解放に失敗しています
* 対策: Arduino IDE 終了＆再度立ち上げてください。また Arduino IDE のシリアルモニターは使わずに TeraTerm を使うことで回避しやすくなります

**「マイコンボードに書き込む」を実行した結果、ウィンドウに下記のように表示された**

```
dyld: Library not loaded: /opt/local/lib/libusb-1.0.0.dylib
  Referenced from: /Users/user1/Library/Arduino15/packages/Seeeduino/tools/stm32_dfu_upload_tool/1.0.0/macosx/dfu-util/dfu-util
  Reason: image not found
/Users/user1/Library/Arduino15/packages/Seeeduino/tools/stm32_dfu_upload_tool/1.0.0/macosx/dfu_upload: line 5: 15851 Abort trap: 6           $(dirname $0)/dfu-util/dfu-util -d $2 -a $1 -D $3 -s $4 -R
DFU end
```

* 原因: macOS で libusb がインストールされていない
* 対策: [Wio LTE 開発環境の準備 / libusb のインストール](https://github.com/soracom/handson/wiki/Wio-LTE-%E9%96%8B%E7%99%BA%E7%92%B0%E5%A2%83%E6%A7%8B%E7%AF%89-macOS-%E7%B7%A8#libusb)を参照して libusb をインストールしてから、再度「マイコンボードに書き込む」を実行してください

<h1 id="step2">【作業】ステップ 2: デバイス稼働時間を SORACOM Harvest で可視化</h1>

Wio LTE の稼働時間を SORACOM Harvest で可視化してみましょう

※Wio LTE 単体の構成でモバイル通信の活用を学びます

![ステップ2 ovewview](https://docs.google.com/drawings/d/e/2PACX-1vQYbHwh3ec7Bfl7o8Hj3a6FvmnHJWfbeahOi0wsUn_BzhhC_-mX-sHmEV85-vFEdd5m6HQRAvh_lyrW/pub?w=371&h=247)

## 最初に. Wio LTE の電源を OFF にする

Wio LTE の microUSB ケーブルを抜き、電源を OFF にしてください

※いきなり抜いてOKです。また、すでに OFF になっている場合は次に進んでください

## 1. Wio LTE にアンテナを取り付ける

**※Wio LTE から microUSB ケーブルを抜いた状態にしてください**

Wio LTE に 添付されているアンテナ2本を取り付けます (**※アンテナは必ず2本取り付けてください**)

![Wio LTE にアンテナを取り付けたところ](https://docs.google.com/drawings/d/e/2PACX-1vQeeuUwMU4t2mjArmyPGsciGmXCYxl8gXPt_gTFiCh-iMOnv-c0s22otGPmvG7-PsMDX34Pe0d3afqC/pub?w=640&h=480)

## 2. Wio LTE に SIM を取り付ける

Wio LTE に SIM (nanoサイズ) を取り付けます

<img src="https://docs.google.com/drawings/d/e/2PACX-1vQ_jUPQyYsHnk0v5yFZpdbJ6n7l2U6Pf1SexxpQ1DyvrxYVbiDTmfT2GjyUmZpR7Dc2dtZneXpkFqZz/pub?w=550&amp;h=255">

### SIM の取り出し

SIM の取り出し方も一緒に紹介いたします

<img src="https://docs.google.com/drawings/d/e/2PACX-1vSHV26TeW7Z3rEbr-OOVQ-6GyI8GijLNChn5ayEspNVPZqmbZTj3pkDdBCZhLEetrMqLdDuXm-GpQk9/pub?w=683&amp;h=322">

<img src="https://docs.google.com/drawings/d/e/2PACX-1vRn4PgJFdW-IV-Fr4JTq7dj6wz4KBSyG3pM6W4Wxdbtt-FunzEy4aSK1_QpssqfoATEWfP_9IJL8vtV/pub?w=508&amp;h=370">

**抜く際フックを引っ張りすぎると金具が取れてしまうため、図示されている程度まで引っ張り出したらSIMを取り出し、金具を元に戻してください**

## 3. SORACOM Harvest の設定をする

**※ SORACOM のアカウントをお持ちでない方:** [SORACOM アカウントの作成](https://dev.soracom.io/jp/start/console/#account) の手順に沿って作成してください   
（有効なクレジットカード(1枚) と この場で確認可能なメールアドレス(1つ)が必要です）

**※ SORACOM アカウントを持っているが SIM を登録していない場合:** [Air SIM の登録](https://dev.soracom.io/jp/start/console/#registsim) の手順に沿って登録してください

### 3-1. [SORACOM Webコンソール](https://console.soracom.io/) で 左上[Menu] > [SIM グループ]

![soracom-menu](https://docs.google.com/drawings/d/e/2PACX-1vRhgmsjqpncv2HQ0jAZwiYf0knTfvmCMl6x_flrdeGQV4N60trp8M981gCAfitVSmXU4tqAYm6MmyRb/pub?w=331&h=410)
![soracom-menu-sim-group](https://docs.google.com/drawings/d/e/2PACX-1vTqI-f2K8n-TuUvVEGPnmDcFxG2f87so3Qfe5K11sn0pXG8Q4v2lJX0UT9tjlH7sDQRb1FC7aFfckjb/pub?w=353&h=290)

### 3-2. [追加] で、SIMグループを作成します (グループ名 `harvest` もしくは任意でかまいません)

![soracom-menu-sim-group-create](https://docs.google.com/drawings/d/e/2PACX-1vQ-wJ7Ixk-BQDtxXweBkhl-deBJzh3behOo_rQNNxm3gO73sKHEV_RvqO7cWrSKJT0AZltPaF_K0qPf/pub?w=381&h=315)

![soracom-menu-sim-group-create-dialog](https://docs.google.com/drawings/d/e/2PACX-1vRjDUj0AzCWEBNyy9GTqWf6jPANTk4WIEZcarMaYd9GhbM-_2AhBru9WglGRplqo0jUroC9rIq82G8h/pub?w=631&h=306)

### 3-3. 先ほど作成した SIMグループ をクリックし、 SORACOM Harvest の設定を ON にして保存をクリック

![soracom-menu-sim-group-list](https://docs.google.com/drawings/d/e/2PACX-1vTpWazZ3_xwnViyK1XJXVo3Aa8BeqhLsdeE4v1SHsUNUhKQw-mS15ZovR4kEzNfhJZw2PYdGEcLB9Fr/pub?w=316&h=334)

![soracom-harvest-on](https://docs.google.com/drawings/d/e/2PACX-1vRjSkL7huwCXXSSknkDnuVbPqYSo9a-rJ0PInLFa-mmgBx1fhhHdVu339RbtSuAuhY2bcFlyMxsGWs1/pub?w=504&h=685)

### 3-4. [SORACOM Webコンソール](https://console.soracom.io/) で 左上[Menu] > [SIM 管理]

![soracom-menu](https://docs.google.com/drawings/d/e/2PACX-1vRhgmsjqpncv2HQ0jAZwiYf0knTfvmCMl6x_flrdeGQV4N60trp8M981gCAfitVSmXU4tqAYm6MmyRb/pub?w=331&h=410)
![soracom-sim-mgr](https://docs.google.com/drawings/d/e/2PACX-1vTUi6LN6Hsctv4KdaZj8uOUFg_ZyROx73f1TzFq41KIlRzjUmE_bc2NR5UnS8cn15TD_S2s8FA-DHzA/pub?w=353&h=290)

### 3-5. Wio LTE に取り付けている SIM を選択 > [操作] > [所属グループ変更]

![soracom-select-sim](https://docs.google.com/drawings/d/e/2PACX-1vQpULGXvkk5htY266aDd2iWJueVphdm8DFRVy_BF5JnWnZfBBLF19U42ni5lU6VxN5ucmwqKHx4ACjg/pub?w=526&h=489)

### 3-6. 先ほど作成した SIMグループ に所属させる

![sim-group-select](https://docs.google.com/drawings/d/e/2PACX-1vR1DJQnKw0NVvv83qxiTiDkh0AYfF6u8g3En7EDQtt2M2OjCRzl_tmlB-02cyiLBHLwWHjpOshFKTAA/pub?w=643&h=334)

以上で SORACOM Web コンソール上での作業は終了です

下記のように Wio LTE に取り付けている SIM の "グループ" が、先ほど作った SIM グループ名になっていれば成功です

![sim-list](https://docs.google.com/drawings/d/e/2PACX-1vSjr7j-ld8piy6POBYX1r8Ib2nW1DLjwanI1bqDXS0VsWh6SFK8RXvfDop5X0hzg2Auq2aSvdH8eDPm/pub?w=520&h=464)

## 4. スケッチを作成する

### 4-1. Arduino IDE を起動する

### 4-2. メニューの [ツール] で [ボード: "Wio Tracker LTE"] と表示されていることを確認する

なっていなければ一覧から "Wio Tracker LTE" を選んでください

### 4-3. Arduino IDE の [ファイル] > [スケッチ例] > [Wio LTE for Arduino] > [soracom] > [soracom-harvest]

新しいウィンドウが開きます

### 4-4. Wio LTE と PC を接続して DFUモード にする

### 4-5. 新しく開いたウィンドウの ![マイコンボードに書き込むアイコン](https://docs.google.com/drawings/d/e/2PACX-1vQiO83cFcX3LCXeioiTiaao57T4SGiIV6XZzcBP6poTwssCxmo7hLpoMh5qG3btyqgzs8Q-lAoE6Q0f/pub?w=100&h=100)(マイコンボードに書き込む) をクリック

### 4-6. 書き込みが完了したら、Wio LTE を 通常モードにする (RSTボタンを押せば通常モードになります)

通常モードで起動次第 SORACOM Harvest へデータを送信し始めます (電源投入から送信開始までは20~25秒程度かかります)

## 5. 確認

### 5-1. [SORACOM Webコンソール](https://console.soracom.io/) で 左上[Menu] > [SIM 管理]

![soracom-menu](https://docs.google.com/drawings/d/e/2PACX-1vRhgmsjqpncv2HQ0jAZwiYf0knTfvmCMl6x_flrdeGQV4N60trp8M981gCAfitVSmXU4tqAYm6MmyRb/pub?w=331&h=410)
![soracom-sim-mgr](https://docs.google.com/drawings/d/e/2PACX-1vTUi6LN6Hsctv4KdaZj8uOUFg_ZyROx73f1TzFq41KIlRzjUmE_bc2NR5UnS8cn15TD_S2s8FA-DHzA/pub?w=353&h=290)

### 5-2. Wio LTE に取り付けている SIM を SORACOM Webコンソール上で選択し [操作] > [データを確認]

![harvest-view](https://docs.google.com/drawings/d/e/2PACX-1vRGN09AF9n0GafAg8Ut9s8QYAmEd4h5Oj4fTYUQjqKEFXCj_aIRjyS3u5zpim0eqtnnh-csIl6sAHaU/pub?w=526&h=489)

下図のように SORACOM Harvest 上で稼働時間が表示されるようになります  
※自動更新を ON にすると、画面の再描画が自動的に行われます  
※グラフ種類を折れ線グラフや棒グラフに変更することができます

![SORACOM Harvest で表示している様子](https://dev.soracom.io/img/gs_wio-lte/soracom-harvest-rendering.png)

## 6. やってみよう(送信間隔の変更)

スケッチを編集して稼働時間の送信間隔を変えてみましょう

スケッチ4行目の `INTERVAL` が周期を指定しています。ミリ秒(ms)単位になっており、 60000 (= 60秒) から変更し、マイコンボードに書き込んでみましょう

```
#define INTERVAL  (60000)
```

↓

```
#define INTERVAL  (5000)
```

## 7. Wio LTE の動作を止める

止める方法は Wio LTE の電源を OFF (= microUSBケーブルを抜く) してください

## うまく動かなかったら（トラブルシュート）

データが表示されない場合は、下記を確認してください

* Wio LTE へアンテナが正しく取り付けられているか？
* SIM が正しく挿入されているか？
* SIM が SORACOM に登録されているか？
* SORACOM Harvest が ON なグループに SIM が所属しているか？

以上を確認しても、まだ尚データ送信されない場合は RST ボタンを押して Wio LTE の再起動を行ってみてください

また、Arduino IDE のシリアルモニターで接続やデータ送信状況が確認できます

※注意: Windows の Arduino IDE のシリアルモニタは、 Wio LTE 開発においては動作が不安定になることがあるため、Windows の方は TeraTerm を使用してください

<h1 id="step3">【作業】ステップ 3: 温湿度センサーのデータを SORACOM Harvest で可視化</h1>

デジタル温湿度センサーのデータを SORACOM Harvest で可視化してみましょう

※センサーを組み合わせた開発を学びます

![ステップ3 ovewview](https://docs.google.com/drawings/d/e/2PACX-1vSWZW16P0NALgTq2O1w1MuAhlK_NagFV-HwU8NNa4Sui1mxDXUdK6Y4TSRfzrAgqDhd5IXPWTvgZjpG/pub?w=615&h=247)

## 最初に. Wio LTE の電源を OFF にする

Wio LTE の microUSB ケーブルを抜き、電源を OFF にしてください

※いきなり抜いてOKです。また、すでに OFF になっている場合は次に進んでください

## 1. Wio LTE にデジタル温湿度センサーを取り付ける

**※Wio LTE から microUSB ケーブルを抜いた状態にしてください**

Grove デジタル温湿度センサーを Wio LTE に取り付けます

Wio LTE の **D38** に取り付けてください

![Wio LTE にデジタル温湿度センサーを取り付けたところ](https://docs.google.com/drawings/d/e/2PACX-1vTZiJ7ep0q7IR2KoNJPnvHcTA_JMGEWTn5qwrlp8AC2YLAtETPRAGXss98LMKm061OlM76hDasUyIoS/pub?w=640&h=480)

## 2. スケッチを作成する

### 2-1. Arduino IDE を起動する

### 2-2. メニューの [ツール] で [ボード: "Wio Tracker LTE"] と表示されていることを確認する

なっていなければ一覧から "Wio Tracker LTE" を選んでください

### 2-3. Arduino IDE の [ファイル] > [スケッチ例] > [Wio LTE for Arduino] > [soracom] > [soracom-harvest]

※上記の通り新規に始めても、ステップ2の続きから始めても構いません

### 2-4. スケッチの8行目の行頭 `//` を取り除きます (アンコメントする)

変更前

```
// #define SENSOR_PIN    (WIOLTE_D38)
```

変更後

```
#define SENSOR_PIN    (WIOLTE_D38)
```

### 2-5. Wio LTE と PC を接続して DFUモード にする

### 2-6. 新しく開いたウィンドウの ![マイコンボードに書き込むアイコン](https://docs.google.com/drawings/d/e/2PACX-1vQiO83cFcX3LCXeioiTiaao57T4SGiIV6XZzcBP6poTwssCxmo7hLpoMh5qG3btyqgzs8Q-lAoE6Q0f/pub?w=100&h=100)(マイコンボードに書き込む) をクリック

### 2-7. 書き込みが完了したら、Wio LTE を 通常モードにする (RSTボタンを押せば通常モードになります)

通常モードで起動次第 SORACOM Harvest へデータを送信し始めます (電源投入から送信開始までは20~25秒程度かかります)

※標準では送信間隔が60秒です。早めたい場合は ステップ2 のやってみようを参考に `INTERVAL` を `5000` などに変えてください

## 3. 確認

SORACOM Harvest 上で温度(temp)・湿度(humi)が表示されるようになります  
※SORACOM Harvest の操作方法は ステップ2 で確認してください

![harvest-all-plot](https://docs.google.com/drawings/d/e/2PACX-1vSwebGsd_kOHhagej9sCP5WEVVYZt45KKKa_vgd343pLYyIMj95sFvdMtDtDSe3eixfDjJBizt3wlS5/pub?w=674&h=333)

## 4. Wio LTE の動作を止める

止める方法は Wio LTE の電源を OFF (= microUSBケーブルを抜く) してください

## うまく動かなかったら（トラブルシュート）

**Wio LTE を通常モードで動かして1分経ってもデータが表示されない**

* 原因: 電波状況などによりセルラー通信に失敗している
* 対策: RST ボタンを押して Wio LTE を再度起動しなおしてください

**SORACOM Harvest でデータ表示がされない**

* 原因: ステップ2 で行った稼働時間のデータが原因で、データのY軸が大きすぎて表示されない場合があります
* 対策: SORACOM Harvest 上で下記操作を行ってください
    - 凡例をクリックすることで当該のデータを非表示できる機能
    - 他の形式で表示する機能
    - 最大値/最小値の設定ができる機能

![harvest-exclude-plot](https://docs.google.com/drawings/d/e/2PACX-1vRa8wgI9GtmrCNPLiKhwF6o-tkCTg4QBC3xUBY_nxK3urV4B0r0b5yEsSLWjIy88tJJKAwCQYJVcvGm/pub?w=634&h=300)

![harvest-alt-plot](https://docs.google.com/drawings/d/e/2PACX-1vR_nFPgaTosGb5Ywy0KsNp6d7yP1MHkMcM6uUqT8fuw4WMdsSTn3fct1izl6MjEmeWLDq6yi3_5lnwW/pub?w=731&h=306)

<h1 id="step4">【作業】ステップ 4: 温湿度センサーのデータを SORACOM Funnel 利用して AWS IoT Core へ転送</h1>

本格的なデータ収集 IoT システムの構築として、クラウド側のデータ処理に AWS IoT Core を活用する構成を学びます

※本ハンズオンでは AWS IoT Core までとしていますが、下図は実際のシステム構成例として[IoT向けアーキテクチャパターン](https://www.slideshare.net/AmazonWebServicesJapan/aws-black-belt-online-seminar-2017-iot/34)を盛り込んでいます

![ステップ4 overview](https://docs.google.com/drawings/d/e/2PACX-1vRkHbjrlzpsr2HX34VyZsQIdlx5m7_BjbA-k3tiHmhkLh9OqbzXvuVLiOY3M6ZPwezpwKcMyU8cxnUH/pub?w=997&h=275)

## 最初に. Wio LTE の電源を OFF にする

Wio LTE の microUSB ケーブルを抜き、電源を OFF にしてください

※いきなり抜いてOKです。また、すでに OFF になっている場合は次に進んでください

## 1. SORACOM へ認証情報を保管する

### 1-1. [SORACOM Webコンソール](https://console.soracom.io/) で 左上[Menu] > [セキュリティ]

![soracom-menu](https://docs.google.com/drawings/d/e/2PACX-1vRhgmsjqpncv2HQ0jAZwiYf0knTfvmCMl6x_flrdeGQV4N60trp8M981gCAfitVSmXU4tqAYm6MmyRb/pub?w=331&h=410)
![soracom-menu-security](https://docs.google.com/drawings/d/e/2PACX-1vRjYW4eP-Cv1GTPmgGD01OEGiszvYmy1gmbsAQx50O6knq0UEPBozSQ8W3ezngFS6Z7B-8ArZkIXSWW/pub?w=344&h=334)

### 1-2. [認証情報ストア] > [認証情報を登録] で 認証情報入力画面を開きます

![soracom-cred-store](https://docs.google.com/drawings/d/e/2PACX-1vRoPNOwuGigBjl6MNfqQpXZL1MFmgPeUuwFLNF3TTRDZMQJf1G-HdwukH5otE7tdHy0YM9MWXmLac6z/pub?w=624&h=309)

### 1-3. 認証情報を登録する画面では下記の通り入力し、保存してください

* 認証情報ID: `awsiotcore-dataaccess`
* 概要: `AWS IoT Core DataAccess`
* 種別: *AWS 認証情報* (これを選択すると、下記IDを入力するテキストボックスが増えます)
* AWS Access Key ID: **ハンズオン運営から入手 (aws-accesskey-id)**
* AWS Secret Access Key ID: **ハンズオン運営から入手 (aws-secret-accesskey-id)**

![soracom-cred-save](https://docs.google.com/drawings/d/e/2PACX-1vR6S9T-9TZbcdB8XwcUrBE7MQCyIsT-zOA2LQGspbv5r72CxT_qc1pyIdgmIXqVU79qOEKfOzuA8vXZ/pub?w=644&h=642)

## 2. SORACOM Funnel 設定

### 2-1. [SORACOM Webコンソール](https://console.soracom.io/) で 左上[Menu] > [SIM グループ]

[追加] で、SIMグループを作成します (グループ名 `funnel` もしくは任意でかまいません)

### 2-2. 先ほど作成した SIMグループ をクリックし、 SORACOM Funnel の設定を開きます

### 2-3. SORACOM Funnel の設定では、下記の通り入力し、保存してください

* 転送先サービス: *AWS IoT*
* 転送先URL: **ハンズオン運営から入手 (endpoint-url)**
* 認証情報: *awsiotcore-dataaccess (AWS IoT Core DataAccess)* (先ほど作成した認証情報を選ぶ)
* 送信データ形式: *JSON*

![soracom-funnel](https://docs.google.com/drawings/d/e/2PACX-1vQ1u87_1m7Mlk-t9G33ho7s8f_4-F8pIGjksjJxBFRhgYYVwJiBWAOVRr0_XWv5ehjU_4hqDvH7kXI_/pub?w=926&h=435)

### 2-4. 左上[Menu] > [SIM 管理]

* Wio LTE に取り付けている SIM を選択 > [操作] > [所属グループ変更]
* 先ほど作成した SIMグループ に所属させる

## 3. スケッチを作成する

### 3-1. Arduino IDE を起動する

### 3-2. メニューの [ツール] で [ボード: "Wio Tracker LTE"] と表示されていることを確認する

なっていなければ一覧から "Wio Tracker LTE" を選んでください

### 3-3. Arduino IDE の [ファイル] > [スケッチ例] > [Wio LTE for Arduino] > [soracom] > [soracom-harvest]

※上記の通り新規に始めても、ステップ3 の続きから始めても構いません

### 3-4. スケッチの7行目の行頭 `//` を取り除きます (アンコメント)

7行目が下記の通りになっていることを確認してください。なっていない場合は先頭の `//` を取り除いて、下記と同じようにしてください

```
#define SENSOR_PIN    (WIOLTE_D38)
```

### 3-5. スケッチの69行目の `harvest.soracom.io` と `8514` の2か所を `funnel.soracom.io` と `23080` へ変更します

変更前

```
  connectId = Wio.SocketOpen("harvest.soracom.io", 8514, WIOLTE_UDP);
```

変更後

```
  connectId = Wio.SocketOpen("funnel.soracom.io", 23080, WIOLTE_UDP);
```

### 3-6. Wio LTE と PC を接続して DFUモード にする

### 3-7. 新しく開いたウィンドウの ![マイコンボードに書き込むアイコン](https://docs.google.com/drawings/d/e/2PACX-1vQiO83cFcX3LCXeioiTiaao57T4SGiIV6XZzcBP6poTwssCxmo7hLpoMh5qG3btyqgzs8Q-lAoE6Q0f/pub?w=100&h=100)(マイコンボードに書き込む) をクリック

### 3-8. 書き込みが完了したら、Wio LTE を 通常モードにする (RSTボタンを押せば通常モードになります)

通常モードで起動次第 SORACOM Funnel へデータを送信し始めます (電源投入から送信開始までは15~20秒程度かかります)

※標準では送信間隔が60秒です。早めたい場合は ステップ2 のやってみようを参考に `INTERVAL` を `5000` などに変えてください

## 4. 確認

AWS IoT Core 上でのデータ着信は、運営側で行います。  
Wio LTE に挿した SIM の IMSI を運営に伝えてください。 IMSI は SORACOM Web コンソールで確認することができます

## 5. Wio LTE の動作を止める

止める方法は Wio LTE の電源を OFF (= microUSBケーブルを抜く) してください

## うまく動かなかったら（トラブルシュート）

利用可能なツールは「シリアルモニター」「SORACOM Webコンソール上のSIMのログ」です  
※運営側は CloudWatch や AWS IoT Core ダッシュボードも利用できます

**(シリアルモニターを見ると) 1回しかデータを送っていない**

* 原因: SORACOM Funnel へのホスト名やポート番号が違う
    * 対策: スケッチ69行目の SORACOM Funnel へのホスト名やポート番号を確認し、再度マイコンへ書き込んでください

**データは送られているが AWS IoT Core 上で確認できない**

* 原因: SORACOM Funnel が ON なグループに SIM が所属していない
    * 対策: SIM を SIM グループへ所属させるようにしてください

**ここまで到達した方は、スタッフにお声がけください**

<h1 id="end">おわりに</h1>

おつかれさまでした  
Wio LTE ハンズオンは以上で終了です。最後に注意点の確認をお願いいたします

## 《知識》今後の費用について

SORACOM Air SIM は**実際の通信の有無に限らず**日当たりの基本料金が発生いたします    
詳細は [ご利用料金 - 日本向け Air SIM](https://soracom.jp/services/air/price/) をご覧ください

### SORACOM Harvest

本ハンズオンで使用した SORACOM Harvest は、**実際の通信の有無に限らず** SORACOM Harvest が利用可能状態の SIM の枚数に応じて日当たりのオプション料金が発生します  
詳細は [SORACOM Harvest のご利用料金](https://soracom.jp/services/harvest/price/) をご覧ください

### SORACOM Beam および SORACOM Funnel

本ハンズオンで使用した SORACOM Beam および SORACOM Funnel は、発生したリクエストに応じた課金がされるサービスです  
詳細は [SORACOM Beam のご利用料金](https://soracom.jp/services/beam/price/) ならびに [SORACOM Funnel のご利用料金](https://soracom.jp/services/funnel/price/) をご覧ください

### クーポンの登録方法

[クーポンコードの使い方](https://soracom.zendesk.com/hc/ja/articles/218121348-%E3%82%AF%E3%83%BC%E3%83%9D%E3%83%B3%E3%82%B3%E3%83%BC%E3%83%89%E3%81%AE%E4%BD%BF%E3%81%84%E6%96%B9%E3%82%92%E6%95%99%E3%81%88%E3%81%A6%E3%81%8F%E3%81%A0%E3%81%95%E3%81%84)をご覧ください

※その他[クーポンに関する注意事項](https://soracom.zendesk.com/hc/ja/articles/115002423011-%E3%82%AF%E3%83%BC%E3%83%9D%E3%83%B3%E3%82%B3%E3%83%BC%E3%83%89%E3%82%92%E9%81%A9%E7%94%A8%E3%81%97%E3%81%A6%E3%81%84%E3%81%A6%E3%82%82%E6%96%99%E9%87%91%E3%81%8C%E8%AB%8B%E6%B1%82%E3%81%99%E3%82%8B%E3%81%AE%E3%81%AF%E3%81%AA%E3%81%9C%E3%81%A7%E3%81%99%E3%81%8B-)もご参照ください

### 長期利用割引

Air SIM のご契約期間をお約束いただくことで、基本料金が割引になる仕組みをご検討ください

* [長期利用割引 - 日本向け Air SIM](https://soracom.jp/services/air/cellular/discount_price/)
* [長期利用割引に関する手続きやFAQ](https://blog.soracom.jp/blog/2017/05/23/long-term-discount/)

### MQTT ブローカーについて

今回利用した Mosquitto の MQTT ブローカーは、試験用の公開サーバです。誰でも読み書きが可能であるため、実システムでの利用はおやめください。

代替するプロダクトは以下の通りです

* オンプレ型
    * [Mosquitto](https://mosquitto.org/)
* フルマネージド型
    * [Amazon MQ](https://aws.amazon.com/jp/amazon-mq/)
    * [AWS IoT Core](https://aws.amazon.com/jp/iot-core/)
    * [Azure IoT Hub](https://azure.microsoft.com/ja-jp/services/iot-hub/)
    * [Google Cloud IoT Core](https://cloud.google.com/iot-core/)
    * [PubNub](https://www.pubnub.com/)

※ すべてが MQTT の機能をサポートしているわけでは無いので要件に応じてご選択ください。

## 【作業】Wio LTE を片付ける

### Wio LTE の初期化

Wio LTE を初期化しておくことで、次回起動時に不用意な通信などを防ぐことができます

作業としては「空っぽのスケッチ」を書き込むことで、初期化できます

1. Arduino IDE を起動する
2. メニューの [ツール] で [ボード: "Wio Tracker LTE"] と表示されていることを確認する
    * なっていなければ一覧から "Wio Tracker LTE" を選んでください
3. Arduino IDE の [ファイル] > [新規ファイル]
4. Wio LTE を PC を接続して DFUモード にする
5. 新しく開いたウィンドウの ![マイコンボードに書き込むアイコン](https://docs.google.com/drawings/d/e/2PACX-1vQiO83cFcX3LCXeioiTiaao57T4SGiIV6XZzcBP6poTwssCxmo7hLpoMh5qG3btyqgzs8Q-lAoE6Q0f/pub?w=100&h=100)(マイコンボードに書き込む) をクリック
6. 書き込みが完了したら、Wio LTE を 通常モードにする (RSTボタンを押せば通常モードになります)

### SIM の取り出し

<img src="https://docs.google.com/drawings/d/e/2PACX-1vSHV26TeW7Z3rEbr-OOVQ-6GyI8GijLNChn5ayEspNVPZqmbZTj3pkDdBCZhLEetrMqLdDuXm-GpQk9/pub?w=683&amp;h=322">

<img src="https://docs.google.com/drawings/d/e/2PACX-1vRn4PgJFdW-IV-Fr4JTq7dj6wz4KBSyG3pM6W4Wxdbtt-FunzEy4aSK1_QpssqfoATEWfP_9IJL8vtV/pub?w=508&amp;h=370">

**抜く際フックを引っ張りすぎると金具が取れてしまうため、図示されている程度まで引っ張り出したらSIMを取り出し、金具を元に戻してください**

### 箱へ戻す

手渡された時同様に箱へ格納してください

* アンテナは取り外して Wio LTE 本体が格納されていた白いトレーのミゾに入れてください
* microUSB ケーブルは白いトレーを取り外した中に入れてください

### 本コンテンツについて

本コンテンツの著作は株式会社ソラコムに帰属します。再利用についてはソラコムへお問合せください。

<h1 id="appendix">Appendix(付録): ハンズオンで使用した環境の構築方法</h1>

<h2 id="aws-iot-core">AWS IoT Core</h2>

* IAM
    * IAM で専用ユーザ作成 (e.g. `awsiot-dataaccess-for-handson`)
    * *awsiot-dataaccess-for-handson* の Access Key ID 発行
    * *awsiot-dataaccess-for-handson* へ *AWSIoTDataAccess* ポリシー割り当て

※ "モノ" や "ポリシー" の作成や設定は不要です

# 運営の方へ

これらを準備、運営側シートへ書き込んでください

* ハンズオンID (e.g. `h0813` など)
* IAM Access Key ID
* IAM Secret Access Key ID

## ステップ 4 における AWS IoT Core へのデータ着信確認

いづれかの方法を使ってください

* AWS IoT Core 管理画面のテストツール
* [AWS IoT Core Monitor](https://ma2shita.s3.amazonaws.com/awsiot.html) (AWS Cognito の設定が別途必要です)

### トラブルシュート

* IAM 確認
    * IAM アカウントに紐づいている Access Key ID
    * IAM アカウントへの割り当てポリシー ( AWSIoTDataAccess ポリシーが必要 )
* AWS IoT Core 確認
    * カスタムエンドポイント (リージョン違い等)
    * Subscribe しているトピック名
* [AWS IoT Core Monitor](https://ma2shita.s3.amazonaws.com/awsiot.html) を使っての確認の場合
    * Cognito と AWS IoT Core のリージョン
    * ID プールの ID
    * 認証されていないロールへの割り当てポリシー ( `AWSIoTDataAccess` ポリシーが必要 )
    * Subscribe しているトピック名

EoT