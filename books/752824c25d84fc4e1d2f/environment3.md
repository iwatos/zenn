
# デプロイする前の実行確認とログの確認
以上でコーディング〜デプロイ、動作確認までローカルでできるようになりました。
ですが現状では以下の問題があります。
- 動作確認をするのに毎回デプロイが必要
- console.logなどのログがローカルから確認できない

これらの問題点を解決するためにGCPの設定をします。

# GCPプロジェクトの作成
最初に実行したclasp login で認証したアカウントでGCPプロジェクトを作成します
https://console.cloud.google.com/


# Oauthの同意設定
Oauthの同意設定が必要なので
https://console.developers.google.com/apis/credentials/consent?project=[PROJECT_ID]
を開きます
外部→作成ボタン
アプリ名:任意
ユーザサポートメール:任意
メールアドレスを設デベロッパーの連絡先情報：任意
を指定して次
スコープは指定なし
テストユーザーは最初に実行したclasp login で認証したアカウントのメールアドレスを追加

# claspにプロジェクトIDを設定
claspにプロジェクトIDを設定します
以下のコマンドを打つと.clasp.jsonに設定値が追加されます
```sh
clasp setting projectId XXXXX
```
次に`casp open`でGASエディタを開き 設定ボタンからGCPプロジェクト番号を設定します。


# 認証情報を作成
https://console.cloud.google.com/apis/credentials?project=[GCP_PROJECT_ID]
認証情報を作成> OauthクライアントID
アプリケーションの種類：デスクトップアプリ
名前：任意
で作成
ファイルダウンロードし、creds.jsonという名前でディレクトリ直下に配置
`clasp login --creds creds.json`を実行し、認証>>権限許可

# 実行可能APIとしてデプロイ
appscriot.jsonに以下を追加
```json:appscriot.json
  "executionApi": {
    "access": "ANYONE"
  }
```
`yarn deploy-develop`を実行

clasp run <関数名>
clasp logs
でデバッグと実行ができる
yarn pushのみでいい

```ts:main.ts
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
    return { status: 'oaaaak', method: 'get' }
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