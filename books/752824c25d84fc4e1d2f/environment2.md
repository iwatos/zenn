---
title: "APIを開発していく"
---
# API開発
GASでAPIのコードを書いていきます。
```ts: main.ts
//eslint-disable-next-line @typescript-eslint/no-unused-vars
function doGet(e: Record<string, unknown>) {
    console.log(e)
    return returnJson({ status: 'ok', method: 'get' })
}

//eslint-disable-next-line @typescript-eslint/no-unused-vars
function doPost(e: Record<string, unknown>) {
    console.log(e)
    return returnJson({ status: 'ok', method: 'post' })
}

function returnJson(result: Record<string, unknown>) {
    const payload = ContentService.createTextOutput()
        .setMimeType(ContentService.MimeType.JSON)
        .setContent(JSON.stringify(result))
    return payload
}
```

GASをAPIとして利用した場合に利用できるメソッドはGETとPOSTのみとなります。
GETメソッドをリクエストした場合はdoGET、POSTメソッドをリクエストした場合はdoPOSTから処理が開始されます
またjsonとして結果を返却する場合は`returnJson()`の処理を通してあげる必要があります。

# 公開
GASのデプロイにはウェブアプリ、実行可能API,ライブラリなどいろいろな種類があります。
それらの説明は今回省きますが、誰でも使えるAPIとして利用したいのでウェブアプリとしてデプロイする必要があります。
`appscript.json`にデプロイ設定を追加します

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
yarn push #sappscriptの設定反映
clasp deploy # デプロイ

# 以下の結果が出たらOK
# Created version 1.
# - <デプロイID文字列>AKfycbz4DNNS28VbZbn4bGsGAt7uLjwYspzN19Z3sIUUuEvWGA-mTwX5yNtdZrgBjWkYArg @1.
```

`https://script.google.com/macros/s/<デプロイID文字列>/exec`
にアクセスして`{"status":"ok","method":"get"}`の結果が返って来れば成功です

補足としてcurlでGASのAPIを叩く場合、GASのリダイレクト仕様のため以下にように叩く必要があります
```sh
# doGet
curl -H "Content-Type: application/json" -L https://script.google.com/macros/s/<デプロイID文字列>/exec
# {"status":"ok","method":"get"} が返ってくる

# doPost
curl -H "Content-Type: application/json" -L https://script.google.com/macros/s/<デプロイID文字列>/exec -d {}
# {"status":"ok","method":"post"} が返ってくる
```

# デプロイ時のURL固定
単に`clasp deploy`を実行すると、デプロイIDとAPIURLが毎回異なるものが生成されてしまいます。
これを固定のIDにするには、オプションでデプロイIDを指定する必要があります。
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

まず先程作成したデプロイIDを検証環境デプロイIDとしましょう。
そして本番環境用のデプロイIDを作成します。
もう一度`clasp deploy`をすれば別のデプロイIDができるのでそれを本番環境用のデプロイIDとします。
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
