---
title: "環境構築"
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
```gitignore:.gitgnore
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