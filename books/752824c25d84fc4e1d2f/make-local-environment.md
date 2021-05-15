---
title: "ローカル環境構築"
free: true
---

# ローカルプロジェクト作成
## ディレクトリの作成
プロジェクト用ディレクトリ名: `gas-project-directory`
プロジェクト名: `gas-project`
として説明を進めます。適宜自分のプロジェクトに合わせて変更してください。

まずGASプロジェクト用のディレクトリを作成します
```shell
mkdir gas-project-directory
cd gas-project-directory
```

## git導入
ローカルでGASを開発する理由の一つはバージョン管理をしたいからです。
ということでgitを導入します
```sh
git init
```

また環境構築をしていく過程で、認証情報やビルド成果物などのpushしたくないファイルが出てくる予定です。
先にプロジェクト直下に以下の`.gitignore`を作成しておきましょう。
```sh
touch .gitignore
```

```sh:gitignore
dist/**
node_modules/**
.clasp.json
.clasprc.json
creds.json
client_secret_*.apps.googleusercontent.com.json
```

## yarn導入
TypeScriptや、GASローカル開発に使うClaspというものを利用するのにyarnを使います。
npmを利用したい方は、本記事中のyarnコマンドをnpmに置き換えてください。
```sh
yarn init
# yarn init v1.22.4
# question name (gas-project-directory): gas-project 
# question version (1.0.0): 
# question description: 
# question entry point (index.js): 
# question repository url: 
# question author: 
# question license (MIT): 
# uestion private: 
# success Saved package.json
# ✨  Done in 9.14s.
```
package.jsonが作成されていればOKです。


# TypeScript環境構築
## TypeScript導入
以下コマンドでtypescriptを入れます
```sh
yarn add typescript
```

 `tsconfig.json` をプロジェクト直下に作成します。
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

プロジェクト用ディレクトリの直下にソースコード`src`ディレクトリを作成し、
その配下に`main.ts`ファイルとしてhelloworldのコードを配置します。
```ts:src/main.ts
console.log("hello gas world!!")
```

次にTypeScriptのコンパイルが成功するか確認します
```sh
yarn run tsc
```

distフォルダとmain.jsファイルが作成されていればコンパイル成功です。
念の為動作を確認します
```sh
node dist/main.js
```
「hello gas world!!」 と出ればOKです

## ESLint、prettier導入
ESLintはリンター（コードを解析して直すべき部分を指摘してくれる）、
prettierはフォーマッター（コードを整形してくれる）です。
便利なので入れましょう。

以下コマンドでESLintとperettierを入れます。
TypeScriptへの対応や、ESLintとperettierを併用する際にひつようなモジュールも含めています。
```sh
yarn add eslint prettier @typescript-eslint/eslint-plugin @typescript-eslint/parser eslint-config-prettier
```

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

設定ファイルを作成したら、ESlintとprettierがそれぞれ動作することをコマンドで確認します。
```sh
yarn run prettier './src/**/*.{js,ts}'
yarn run eslint './src/**/*.{js,ts}'
```
ESLint、prettier導入もこれで完了です。

ESLint、prettierで自動でコードを整形してくれるコマンドが便利なので、
簡単に実行しやすいようにpackage.jsonにscriptコマンドを追加しておきましょう
```diff json:package.json
{
+  "scripts": {
+    "lintfix": "prettier --write './src/**/*.{js,ts}' && eslint --fix './src/**/*.{js,ts}'",
+  },
}
```

# clasp導入
GASをローカルで開発するためのモジュール`clasp`をインストールします。
```sh
yarn install clasp -g
```
グローバル環境を汚したくない方は`-g`オプションを外してください。
その場合本記事中の`clasp`コマンドは`yarn run clasp`に読み替えてください。