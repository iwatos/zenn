---
title: "APIを開発していく"
---
# API開発
GASでAPIのコードを書いていきます。
```ts: main.ts
//eslint-disable-next-line @typescript-eslint/no-unused-vars
function doGet(e: Record<string, unknown>) {
    console.log(e)
    return returnJson({ status: 'ok', method: 'post' })
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
yarn push でpushした後、
clasp open　でGASエディターの画面をブラウザで開きます。

デプロイ > 新しいデプロイ　をクリックし
種類の選択 から ウェブアプリにチェックを入れ、
自分として実行
アクセスできるユーザを全員
に設定してデプロイボタンをクリック

表示されたURLをクリックして
{"status":"ok","method":"get"}　もの表示が出ればデプロイ成功

# curlでの取得
curlでGASのAPIを叩く場合、GASのリダイレクト仕様のため以下にように叩く必要がある

```sh
# doGet
curl -H "Content-Type: application/json" -L <API_URL>
# doPost
curl -H "Content-Type: application/json" -L <API_URL> -d {}
```

# clasp deploy
claspからデプロイすることもできます
まずwebアプリケーションとして公開するためにappsscript.jsonに以下の設定を追記します
```
  "webapp": {
    "access": "ANYONE_ANONYMOUS",
    "executeAs": "USER_DEPLOYING"
  }
```

また先程ブラウザからデプロイしたときのデプロイIDをコピーして
`clasp deploy -i <デプロイID>`
と打てばデプロイできます。デプロイIDを指定しないとAPIのURLが変わってしまう恐れがあります。
package.jsonにコンパイル〜デプロイまでやってくれるコマンドを作りましょう

```diff json
"scripts": {
      "lint": "prettier './src/**/*.{js,ts}' && eslint './src/**/*.{js,ts}'",
      "lintfix": "prettier --write './src/**/*.{js,ts}' && eslint --fix './src/**/*.{js,ts}'",
      "push": "yarn lintfix && cp appsscript.json dist/appsscript.json && tsc && clasp push -f",
+     "deploy": "yarn push && clasp deploy -i <デプロイID>"
    },
```


# ログの確認
以上でコーディング〜デプロイ、動作確認までローカルでできるようになりました。
ですが現状では以下の問題があります。
- 動作確認をするのに毎回デプロイ（本番更新）が必要
- console.logなどログがローカルから確認できない

これらの問題点を解決するためにGCPの設定をします。