---
title: "ローカルで動作する環境の構築"
free: true
---
# GASプロジェクトの準備
プロジェクト用ディレクトリ名: `gas-project-directory`
プロジェクト名: `gas-project`
として説明を進めます。適宜自分のプロジェクトに合わせて変更してください。

まずGASプロジェクト用のディレクトリを作成します
```shell
mkdir gas-project-directory
cd gas-project-directory
```

# git環境作成
環境構築をしていく過程で、認証情報やビルド成果物などのpushしたくないファイルが出てくるので、
`git init`をした後に、先にプロジェクト直下に以下の`.gitignore`を作成しておきましょう。
```
dist/**
node_modules/**
.clasp.json
.clasprc.json
creds.json
client_secret_*.apps.googleusercontent.com.json
```

# claspを用意
以下のpackage.jsonをフォルダ直下に作成後、yarnかnpmでinstallを実行します
GASのローカル開発で用いる`clasp`のほか、`typescript` `eslint` `prettier`を導入してます。

```json:gas-project-directory/package.json
{
  "name": "gas-project",
  "version": "1.0.0",
  "description": "",
  "author": "",
  "license": "MIT",
  "dependencies": {
    "@google/clasp": "^2.3.0",
    "@types/google-apps-script": "^1.0.21"
  },
  "devDependencies": {
    "eslint": "^7.18.0",
    "prettier": "^2.2.1",
    "@typescript-eslint/eslint-plugin": "^4.14.0",
    "@typescript-eslint/parser": "^4.14.0",
    "eslint-config-prettier": "^7.2.0",
    "typescript": "^4.1.3"
  }
}
```
```sh
yarn install # または `npm install`
```

これでGASをローカルで開発するためのツール`clasp`がインストールされます。

次にclaspを利用してプロジェクトを新規作成します
まずは下記コマンドでGoogleアカウント連携をおこないます。
ブラウザ認証画面が表示されますのでGoogleアカウントで認証してください。
```sh
yarn run clasp login 
```

次にプロジェクト作成をおこないます。
```sh
clasp create gas-project
# standalone, api, sheet, doc など初期テンプレートをどれにするか聞かれるので
# standaloneを選ぶ
```
実行後にディレクトリ直下に　`.clasp.json` `appscript.json` が作成されます。


# 動作テスト用のソースコード配置
プロジェクト用ディレクトリの直下にソースコード`src`ディレクトリを作成し、
その配下に`main.ts`ファイルとしてhelloworldのコードを配置します。
```ts:src/main.ts
console.log("hello gas world!!")
```


# TypeScript導入
先程の`yarn install`で導入したTypeScriptの設定ファイルを作成します。
以下の `tsconfig.json` をプロジェクト直下に作成します。
こちらの設定に詳しい方はご自由に内容を変更しても大丈夫です。

```json:tsconfig.json
{
    "inlude": [
        "src/**/*"
    ],
    "compilerOptions": {
        "target": "ES2015",
        "module": "commonjs",
        "allowJs": false,
        "outDir": "./dist",
        "strict": true,   /* Enable all strict type-checking options. */
        "skipLibCheck": true
    }
}
```

次にTypeScriptのコンパイルが成功するか確認します
```sh
yarn run tsc
```
distフォルダとmain.jsファイルが作成されていれば成功です。
念の為動作を確認します
```sh
node dist/main.js
# hello gas world!! と出ればOK
```

# ESLint、prettier導入
先程の`yarn install`で導入したESLint,prettierの設定ファイルを作成します。
以下の`.eslintrc.json` `.prettierrc.json` をディレクトリ直下に作成します。
こちらの設定に詳しい方はご自由に内容を変更しても大丈夫です。

```json:.eslintrc.json
{
  "root": true,
  "parser": "@typescript-eslint/parser",
  "plugins": [
    "@typescript-eslint"
  ],
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "prettier",
    "prettier/@typescript-eslint"
  ]
}
```
```json:.prettierrc.json
{
  "trailingComma": "es5",
  "tabWidth": 4,
  "semi": false,
  "singleQuote": true
}
```

設定ファイルを作成したら、以下コマンドが動作することを確認します。
```sh
yarn run prettier './src/**/*.{js,ts}'
yarn run eslint './src/**/*.{js,ts}'
```

以上でESLint、prettier導入もこれで完了です。
ESLint、prettierで自動でフォーマットをかけてくれるコマンドが便利なので、
簡単に実行しやすいようにpackage.jsonにscriptコマンドを追加しておきましょう
```diff json:package.json
{
+  "scripts": {
+    "lintfix": "prettier --write './src/**/*.{js,ts}' && eslint --fix './src/**/*.{js,ts}'",
+  },
}
```

# pushの対象設定
TypeScriptのコンパイルで生成したdistフォルダをGASサーバにプッシュするための設定をします。
実はGASはtypescriptの状態でpushをしても自動でjsにコンパイルしてプッシュしてくれるのですが、それだとtsconfigのコンパイル設定が適用できないため、ローカルでjsコンパイルをした結果のdistフォルダをプッシュするようにします

まず`.clasp.json`にdistフォルダをpush対象とする設定を追加します
```diff json:.clasp.json
{
  "scriptId": "<固有のスクリプトID>"
+ "rootDir": "dist" ＿
}
```

さらにpush対象（distフォルダ）の中にappscript.jsonを含める必要があるため、
`package.json`にてまずコンパイル用のscriptコマンドを作成します。
さらにフォーマッタ、コンパイルからGASサーバにプッシュするまでのコマンドも作ってしまいます。
```diff json:package.json
{
  "scripts": {
    "lintfix": "prettier --write './src/**/*.{js,ts}' && eslint --fix './src/**/*.{js,ts}'",
+   "compile": "cp appsscript.json dist/appsscript.json && tsc",
+   "push": "yarn lintfix && yarn compile && clasp push -f"
  },
}
```

ここまでできたら実際にpushを実行してみて反映されているか確認してみましょう。
反映されていれば成功です。
```sh
yarn push # distの内容をGASサーバにプッシュ
yarn run clasp open # ブラウザでGASエディタが開くので、distの内容が反映されていることを確認
```

# APIとして公開
main.tsに以下の内容を記述してください。
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
    return { status: 'ok', method: 'get' }
}

function converObjectToJsonString(result: Record<string, unknown>) {
    const payload = ContentService.createTextOutput(
        JSON.stringify(result)
    ).setMimeType(ContentService.MimeType.JSON)
    return payload
}
```

GASをAPIとして利用した場合に利用できるメソッドはGETとPOSTのみとなります。
GETメソッドをリクエストした場合はdoGET、POSTメソッドをリクエストした場合はdoPOSTから処理が開始されます
またjsonとして結果を返却する場合は`converObjectToJsonString()`の処理を通してあげる必要があります。

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


# デプロイする前の実行確認とログの確認
これまでの作業でコーディング〜デプロイ、動作確認までローカルでできるようになりました。
ですが現状では以下の問題があります。
- 動作確認をするのに毎回デプロイが必要
- console.logなどのログがローカルから確認できない

これらは
- clasp run（pushされているコードを実行）
- clasp logs(ログの出力) 
コマンドが利用できれば解決しますが、そのためにはGCPの設定が必要です。
ややこしい作業が多くなるので、そこまで求めないという方は本章の作業はしなくても問題ありません。

# GCPプロジェクトの作成
最初に実行したclasp login で認証したアカウントでGCPプロジェクトを作成します
`https://console.cloud.google.com/`から
プロジェクトの作成 > 新しいプロジェクト を選択し
- プロジェクト名：任意
- 場所：任意

で新しいGCPプロジェクトを作成します
![](https://storage.googleapis.com/zenn-user-upload/17l4be50k870a6xyoqrd1l1sysdx)
![](https://storage.googleapis.com/zenn-user-upload/iwbdbhy8kjyb29875wn0d5rm2gmr)

# Oauthの同意設定
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

# claspにプロジェクトIDを設定
claspにプロジェクトIDを設定します
まず以下のコマンドを打ち.clasp.jsonにprojectIdを設定します
```sh
clasp setting projectId <作成したGCPのプロジェクトID>
```

次に`yarn run clasp open`でGASエディタを開き、設定ボタンからGCPプロジェクト番号（プロジェクトIDではないので注意）を設定します。
![](https://storage.googleapis.com/zenn-user-upload/shsd5gkmpoudu53itdo4xiid9eer)


# 認証情報を作成
`https://console.cloud.google.com/apis/credentials?project=[PROJECT_ID]`
から認証情報ファイルの作成します
1. 認証情報を作成> OauthクライアントID を選択
2. OAuth クライアント ID の作成
   - アプリケーションの種類：デスクトップアプリ
   - 名前：任意
3. ファイルをダウンロードし、creds.jsonという名前でプロジェクト直下に配置
   ![](https://storage.googleapis.com/zenn-user-upload/9qib0jncvsdxh3zghebfz7kad8jh)
4. `clasp login --creds creds.json`を実行するとブラウザで認証画面が開くので、認証 > 権限許可 を行う

# 実行可能APIとしてデプロイ
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

これで以下を実行して結果が帰って来ればOKです。
clasp run ではconverObjectToJsonStringの処理を通す必要がないのでexecuteDoGet/executeDoPostを指定します
```sh
clasp run executeDoGet
clasp run executeDoPost
clasp logs
```