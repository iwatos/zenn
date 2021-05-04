---
title: "環境構築 APIとして公開"
---
# APIとして公開
main.tsに以下の内容を記述してください。
```ts: main.ts
//eslint-disable-next-line @typescript-eslint/no-unused-vars
function doGet(e: Record<string, unknown>) {
    const result = executeDoGet(e)
    return returnJson(result)
}

//eslint-disable-next-line @typescript-eslint/no-unused-vars
function doPost(e: Record<string, unknown>) {
    const result = executeDoPost(e)
    return returnJson(result)
}

function executeDoGet(e: Record<string, unknown>) {
    console.log(e)
    return { status: 'ok', method: 'get' }
}

function executeDoPost(e: Record<string, unknown>) {
    console.log(e)
    return { status: 'ok', method: 'get' }
}

function returnJson(result: Record<string, unknown>) {
    const payload = ContentService.createTextOutput(
        JSON.stringify(result)
    ).setMimeType(ContentService.MimeType.JSON)
    return payload
}
```

GASをAPIとして利用した場合に利用できるメソッドはGETとPOSTのみとなります。
GETメソッドをリクエストした場合はdoGET、POSTメソッドをリクエストした場合はdoPOSTから処理が開始されます
またjsonとして結果を返却する場合は`returnJson()`の処理を通してあげる必要があります。

# 公開
GASのデプロイにはウェブアプリ、実行可能API,ライブラリなどいろいろな種類があります。
それらの説明は今回省きますが、誰でもURLを叩けばJson結果を取得できるようにしたい場合はウェブアプリとしてデプロイする必要があります。
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

この設定を追加後、以下のコマンドを打ちます
```sh
yarn push #appscriptの設定を反映
clasp deploy # デプロイ（公開）を実行
# 以下の結果が出たらOK
# Created version 1.
# - <固有のデプロイID> @1.
```
これでデプロイが完了し処理が公開されました

`https://script.google.com/macros/s/<固有のデプロイID>/exec`
にアクセスして`{"status":"ok","method":"get"}`の結果が返って来れば成功です

POSTメソッドの結果も確認したい場合はcurlを利用しましょう。
補足としてcurlでGASのAPIを叩く場合、GASのリダイレクト仕様のため以下のように叩く必要があります。
`-X POST`のオプションではうまく動かないので注意しましょう。
```sh
# doGet {"status":"ok","method":"get"} が返ってくる
curl -H "Content-Type: application/json" -L https://script.google.com/macros/s/<デプロイID文字列>/exec

# doPost {"status":"ok","method":"post"}が返ってくる
curl -H "Content-Type: application/json" -L https://script.google.com/macros/s/<デプロイID文字列>/exec -d {}
```

# デプロイ時のURL固定
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

# 検証環境・本番環境の作成
デプロイIDの異なる各デプロイは共存できます。
APIを修正する時に本番環境をいじるのは怖いので、検証と本番を用意しましょう。

まず先程作成したデプロイIDを検証環境デプロイIDとします。
そしてそれとは別に本番環境用のデプロイIDを作成します。
もう一度`clasp deploy`をすれば別のデプロイIDが生成されるのでそれを本番環境用のデプロイIDとします。
そしてpackage.jsonに検証と本番用のデプロイコマンドを作りましょう。

```diff json
"scripts": {
      "lint": "prettier './src/**/*.{js,ts}' && eslint './src/**/*.{js,ts}'",
      "lintfix": "prettier --write './src/**/*.{js,ts}' && eslint --fix './src/**/*.{js,ts}'",
      "push": "yarn lintfix && cp appsscript.json dist/appsscript.json && tsc && clasp push -f",
-     "deploy": "yarn push && clasp deploy -i <デプロイID>"
+     "deploy-develop": "yarn push && clasp deploy -i <検証用のデプロイID>",
+     "deploy-production": "yarn push && clasp deploy -i <本番用のデプロイID>"
},
```
これらもcurlコマンドを叩いて応答が来ればOKです。
