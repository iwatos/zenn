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

# claspを用意
以下のpackage.jsonをフォルダ直下に作成後、yarnかnpmでinstallを実行します
GASのローカル開発で用いる`clasp`のほか、`typescript` `eslint` `prettier`を導入してます。

```json:gas-project-directory/package.json
{
  "name": "<プロジェクト名>",
  "version": "1.0.0",
  "description": "Template of Google App Script ",
  "author": "<>",
  "license": "MIT",
  "scripts": {
    "lint": "prettier './src/**/*.{js,ts}' && eslint './src/**/*.{js,ts}'",
    "lintfix": "prettier --write './src/**/*.{js,ts}' && eslint --fix './src/**/*.{js,ts}'"
  },
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

GASをローカルで開発するためのツール`clasp`がインストールされます。
claspを利用してプロジェクトを新規作成します

まずは下記コマンドGoogleアカウント連携をおこないます。
ブラウザ認証画面が表示されますのでGoogleアカウントで認証してください。
```sh
clasp login 
```

次にプロジェクト作成をおこないます。
```sh
clasp create gas-project
# standalone, api, sheet, doc など初期設定を聞かれるので選ぶ
# どれを選んでも以降の作業に問題はないはず 
```
実行後にディレクトリ直下に　`.clasp.json` `appscript.json` が作成されます。


# ソースコードの配置
プロジェクト用ディレクトリの直下にソースコード`src`ディレクトリを作成します
その配下にサンプルとしてhelloworldのコードを置いておきます。
```ts:src/main.ts
console.log("hello gas world!!")
```


# TypeScript導入
先程の`yarn install`で導入したTypeScriptrの設定ファイルを作成します。
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
簡単に実行しやすいようにコマンドを追加しておきましょう
```diff json:package.json
{
  // 省略
+  "scripts": {
+    "lint": "prettier './src/**/*.{js,ts}' && eslint './src/**/*.{js,ts}'",
+    "lintfix": "prettier --write './src/**/*.{js,ts}' && eslint --fix './src/**/*.{js,ts}'",
+  },
  // 省略
}
```

# pushの対象設定
ローカルで生成したdistフォルダをプッシュするための設定をします。
実はGASはtypescriptの状態でpushをしても自動でコンパイルをしてプッシュしてくれるのですが。
tsconfigの設定を反映させたいためにdistフォルダをプッシュするようにします

まず`.clasp.json`にpush対象の設定をします
```diff json:.clasp.json
{
  "scriptId": "1n326kT-w55WXF7UBGRQuCYIRQ2qz3LEBK_22y3c-8K0xxsjZw4gHCfL9"
+ "rootDir": "dist" 
}
```

さらにpush対象の中にappscript.jsonを入れる必要があるため、
`package.json`にてGAプッシュ用のコマンドを作成します。
```diff json:package.json
{
  // 省略
  "scripts": {
    "lint": "prettier './src/**/*.{js,ts}' && eslint './src/**/*.{js,ts}'",
    "lintfix": "prettier --write './src/**/*.{js,ts}' && eslint --fix './src/**/*.{js,ts}'",
+   "push": "cp appsscript.json dist/appsscript.json && clasp push -f"
  },
  // 省略
}
```