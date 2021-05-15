---
title: "デバッグ用の設定"
free: true
---

# APIのデプロイ
## API用のコードを作成
GASでAPIを作成する場合、利用できるHTTPメソッドはGETとPOSTのみです。
URLアクセスした時にGETの場合は`doGet`、POSTの場合は`doPost`関数が実行されるのでそれぞれ用意する必要があります。

なのでまずmain.tsに以下の内容を記述します。
```ts: main.ts
//eslint-disable-next-line @typescript-eslint/no-unused-vars
function doGet(e: Record<string, unknown>) {
    const result = executeDoGet(e)
    return converObjectToJsonString(result)
}

//eslint-disable-next-line @typescript-eslint/no-unused-vars
function doPost(e: Record<string, unknown>) {
    const result = executeDoPost(e)
    return converObjectToJsonString(result)
}

function executeDoGet(e: Record<string, unknown>) {
    console.log("start executeDoGet")
    return { status: 'ok', method: 'get' }
}

function executeDoPost(e: Record<string, unknown>) {
    console.log("start executeDoPost")
    return { status: 'ok', method: 'post' }
}

function converObjectToJsonString(result: Record<string, unknown>) {
    const payload = ContentService.createTextOutput(
        JSON.stringify(result)
    ).setMimeType(ContentService.MimeType.JSON)
    return payload
}
```
doGetは`{ status: 'ok', method: 'get' }`
doPostは`{ status: 'ok', method: 'post' }`
を結果として返す関数になっています。

GASでAPIを公開してjsonを返却する場合は、結果はオブジェクトのままでなく`converObjectToJsonString()`の処理を通す必要があります。

またローカルでESlintを利用していると`doGet`、`doPost`がどこからも参照されてないと怒られます。
これらは上述したようにここから処理が始まる関数で参照されないのは仕方がないので`//eslint-disable-next-line @typescript-eslint/no-unused-vars`のコメントで警告が出ないように防いでいます。


## ウェブアプリとしてデプロイ
GASのデプロイにはウェブアプリ、実行可能API、ライブラリなどいろいろな種類があります。
誰でもURLを叩けばJson結果を取得できるようにしたい場合はウェブアプリとしてデプロイする必要があります。
`appscript.json`にウェブアプリとしてデプロイする設定を追加します

```diff json:appscript.json
{
  "timeZone": "America/New_York",
  "dependencies": {
  },
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8",
+ "webapp": {
+  "access": "ANYONE_ANONYMOUS",
+  "executeAs": "USER_DEPLOYING"
+ }
}
```

この設定は、ブラウザ上でデプロイする場合の以下に相当します。
![](https://storage.googleapis.com/zenn-user-upload/hglnzqafszgnla0vudi3a3mw91h3)

この設定を追加後、以下のコマンドを打ちます
```sh
yarn push #appscriptの設定を反映
clasp deploy # デプロイ（公開）を実行
# 以下の結果が出たらOK
# Created version 1.
# - <デプロイID> @1.
```
これでデプロイが完了し処理が公開されました

`https://script.google.com/macros/s/<デプロイID>/exec`
にアクセスして`{"status":"ok","method":"get"}`の結果が返って来れば成功です

POSTメソッドの結果も確認したい場合はcurlを利用しましょう。
補足としてcurlでGASのAPIを叩く場合、GASのリダイレクト仕様のため以下のように叩く必要があります。
`-X POST`のオプションではうまく動かないので注意しましょう。
```sh
# doGet {"status":"ok","method":"get"} が返ってくる
curl -H "Content-Type: application/json" -L https://script.google.com/macros/s/<デプロイID>/exec

# doPost {"status":"ok","method":"post"}が返ってくる
curl -H "Content-Type: application/json" -L https://script.google.com/macros/s/<デプロイID>/exec -d {}
```

## デプロイ時のURL固定
単に`clasp deploy`を実行すると、デプロイIDとURLが毎回異なるものが生成されてしまいます。
これらを固定のIDにするには、オプションでデプロイIDを指定する必要があります。
`clasp deploy -i <デプロイID>`

毎回デプロイIDを指定するのは面倒なので、package.jsonにコンパイル〜ID指定デプロイまでやってくれるコマンドを作りましょう。
```diff json
"scripts": {
      "lint": "prettier './src/**/*.{js,ts}' && eslint './src/**/*.{js,ts}'",
      "lintfix": "prettier --write './src/**/*.{js,ts}' && eslint --fix './src/**/*.{js,ts}'",
      "push": "yarn lintfix && cp appsscript.json dist/appsscript.json && tsc && clasp push -f",
+     "deploy": "yarn push && clasp deploy -i <デプロイID>"
    },
```

## 検証環境・本番環境の作成
デプロイIDの異なる各デプロイは共存できます。
APIを修正する時に本番環境をいじるのは怖いので、検証と本番を用意しましょう。

まず先程作成したデプロイIDを検証環境デプロイIDとします。
もう一度`clasp deploy`をすれば別のデプロイIDが生成されるのでそれを本番環境用のデプロイIDとします。
そしてpackage.jsonに検証と本番用のデプロイコマンドを作りましょう。

```diff json
"scripts": {
      "lint": "prettier './src/**/*.{js,ts}' && eslint './src/**/*.{js,ts}'",
      "lintfix": "prettier --write './src/**/*.{js,ts}' && eslint --fix './src/**/*.{js,ts}'",
      "push": "yarn lintfix && cp appsscript.json dist/appsscript.json && tsc && clasp push -f",
-     "deploy": "yarn push && clasp deploy -i <デプロイID>"
+     "deploy-develop": "yarn push && clasp deploy -i <検証用のデプロイID> -d 'develop'",
+     "deploy-production": "yarn push && clasp deploy -i <本番用のデプロイID> -d 'production'"
},
```
これらもそれぞれcurlコマンドを叩いて応答が来ればOKです。


# `clasp run` コマンド利用のための設定
これまでの作業でコーディング〜デプロイ、動作確認までローカルでできるようになりました。
ですが現状では以下の問題があります。
- 動作確認をするのに毎回デプロイ（公開）が必要
- console.logのログがローカルから確認できない

これらはそれぞれ
- clasp run (デプロイ前のpushされているコードを実行)
- clasp logs (ログの出力)

コマンドが利用できれば解決しますが、そのためにはGCPの設定が必要です。

ややこしい作業が多くなるので、そこまで求めないという方は本章の作業はしなくても問題ありません。

## GCPプロジェクトの作成
最初に実行したclasp login で認証したアカウントでGCPプロジェクトを作成します
`https://console.cloud.google.com/`から
プロジェクトの作成 > 新しいプロジェクト を選択し
- プロジェクト名：任意
- 場所：任意

で新しいGCPプロジェクトを作成します
![](https://storage.googleapis.com/zenn-user-upload/17l4be50k870a6xyoqrd1l1sysdx)
![](https://storage.googleapis.com/zenn-user-upload/iwbdbhy8kjyb29875wn0d5rm2gmr)

## Oauthの同意設定
Oauthの同意設定が必要です。
GCPのダッシュボードに作成したGCPのプロジェクトIDが表示されているはずなのでそれをコピーし以下のリンクを開きます。
`https://console.developers.google.com/apis/credentials/consent?project=[PROJECT_ID]`

1. ラジオボタンで 外部を選択し 作成ボタンをクリック
2. アプリ情報の入力
   - アプリ名:任意
   - ユーザサポートメール:任意
   - メールアドレスを設デベロッパーの連絡先情報：任意
    を指定して次へ
3. スコープは指定なしで次へ
4. テストユーザーに最初に実行した`yarn run clasp login`で認証したアカウントのメールアドレスを追加して次へ
概要画面が表示されたらOauthの同意設定は完了です。

## claspにプロジェクトIDを設定
claspにプロジェクトIDを設定します
まず以下のコマンドを打ち.clasp.jsonにprojectIdを設定します
```sh
clasp setting projectId <作成したGCPのプロジェクトID>
```

次に`yarn run clasp open`でGASエディタを開き、設定ボタンからGCPプロジェクト番号（プロジェクトIDではないので注意）を設定します。
![](https://storage.googleapis.com/zenn-user-upload/shsd5gkmpoudu53itdo4xiid9eer)


## 認証情報を作成
`https://console.cloud.google.com/apis/credentials?project=[PROJECT_ID]`
から認証情報ファイルの作成します
1. 認証情報を作成> OauthクライアントID を選択
2. OAuth クライアント ID の作成
   - アプリケーションの種類：デスクトップアプリ
   - 名前：任意
3. ファイルをダウンロードし、creds.jsonという名前でプロジェクト直下に配置
   ![](https://storage.googleapis.com/zenn-user-upload/9qib0jncvsdxh3zghebfz7kad8jh)
4. `clasp login --creds creds.json`を実行するとブラウザで認証画面が開くので、認証 > 権限許可 を行う

## 実行可能APIとしてデプロイ
これでclasp run/clasp logsを使う環境が整いました。
しかしclasp runを実行するにはウェブアプリではなく実行可能APIとしてデプロイしておく必要があるのでその設定を入れます。
まずappscriot.jsonに実行可能APIとデプロイ設定をを追加します。
```diff json:appscriot.json
+  "executionApi": {
+    "access": "ANYONE"
+  }
```

そうしたらデプロイを実行します
```sh
yarn deploy-develop
```

これで`clasp run`,`clasp logs`以下を実行して結果が帰って来ればOKです。
```sh
clasp run executeDoGet
clasp run executeDoPost
clasp logs
```
`clasp run`では`converObjectToJsonString`の処理を通す必要がないので`executeDoGet`,`executeDoPost`を指定します