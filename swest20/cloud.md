# クラウド・ファースト デバイス

<img src="https://docs.google.com/drawings/d/e/2PACX-1vT1RWW4TCEhHGk1q6KGxybB1hVyWvOcegXJODfaOnWgfaH6apfZ39VJlOgvOuPnUQvOwu53kxfRBWZ4/pub?w=926&amp;h=518" alt="step4a Cloud / ovewview">

## 本セッションで利用する "クラウド"

AWS IoT Core を利用し、条件を AWS IoT Core にて判定するようにします。

<img src="https://docs.google.com/drawings/d/e/2PACX-1vR6wusGtYwMFtOCZ-MAUjBkU-WGYbqaQF8oDVhnMBfJpTytZlmzZSYDJVF4mEghzhApkxJ2sry5z5cI/pub?w=929&amp;h=520" alt="step4a Cloud / architecture">

### AWS IoT Core とは

IoT デバイス向けのメッセージ処理基盤です。  
フルマネージド・サービスであり、構築や運用の手間は AWS がすべて行ってくれます。

1台～億を超えるデバイスからのメッセージを受け付けることができ、またルールに応じて AWS の各サービスに接続できます。

### AWS Lambda とは

コードを書くだけで実行できる基盤です。  
フルマネージド・サービスであり、構築や運用の手間は AWS がすべて行ってくれます。

<img src="https://docs.google.com/drawings/d/e/2PACX-1vRRMKKl5SMjLhPip5UbneyF9nzrNgbhc_3q22peqWG7katUvSuQU2-Cn82EqAGCxip5q7DyiIHKOLZN/pub?w=881&amp;h=688">

<h1 id="step4a">【作業】ステップ 4A: 送信先を変更し、クラウド上での条件付けを行う</h1>

## 最初に. ステップ 4 まで終了していることを確認する

ステップ 4 まで終わっている Wio LTE に microUSB ケーブルを挿し、起動します。

もし既に Wio LTE の電源が入っている場合は、そのままにしておいてください。

## 1. SORACOM Funnel 設定

### 1-1. [SORACOM Web コンソール](https://console.soracom.io) で 左上[Menu] > [SIM グループ]

### 1-2. ステップ 4 で作成し設定した SORACOM Funnel のグループを開きます

### 1-3. SORACOM Funnel の設定を下記の通りに入力（変更）し、保存してください

* 転送先 URL: **ハンズオン運営から入手 (filter-endpoint-url)**

## 2. 確認

AWS IoT Core 上でのデータ着信は、運営側で行います。  
Wio LTE に挿した SIM の IMSI を運営に伝えてください。 IMSI は SORACOM Web コンソールで確認することができます

### AWS IoT Core 上のルールについて

ルールは以下のように設定されています。

監視対象のトピックに着信したメッセージが条件に一致した時に、アクションが実行されるようになっています。

* 監視対象: `filter/#` ( `#` はワイルドカード)
* 条件: `payloads.temp` が `30` を上回る時 (JSON のオブジェクト指定形式で宣言可能)
* アクション: `post-to-slack-max_filter` Lambda 関数を実行する

<img src="https://docs.google.com/drawings/d/e/2PACX-1vRq1AuLujKUW3BwUtRGBF8EorM0sS_oGm6prTB45rKUAAgf2c3hE5gHZI8eTb1g3PwjvQgSP9KINPWZ/pub?w=488&amp;h=329" alt="step4a Cloud / awsiotcore-rule">

### post-to-slack-max_filter Lambda 関数について

重要な部分は以下の通りです。この Lambda 関数は AWS IoT Core のルールから、メッセージが着信した毎に実行されます。

```python
def lambda_handler(event, context):
    slack_message = {
        'channel': SLACK_CHANNEL,
        'text': str(event['payloads'])
    }
    req = Request(HOOK_URL, json.dumps(slack_message))
```

* `event` に AWS IoT Core から渡されたデータが入った状態で実行されます
* `slack_message` を組み立てます
* `Request` を利用して `HOOK_URL` へ HTTP POST します

## 3. Wio LTE の動作を止める

止める方法は Wio LTE の電源を OFF (= microUSBケーブルを抜く) してください

**以上で本セッションは終了です**

[おわりに](index#end) に進んでください。

