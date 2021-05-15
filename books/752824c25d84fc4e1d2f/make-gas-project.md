---
title: "GASプロジェクト作成"
free: true
---

# GASプロジェクトの作成
## プロジェクト作成
claspを利用してプロジェクトを新規作成します

まずは下記コマンドでGoogleアカウント連携をおこないます。
ブラウザ認証画面が表示されますのでGoogleアカウントで認証してください。
また以降の作業はここで認証したアカウントで作業してください。
```sh
clasp login 
```

次にプロジェクト作成をおこないます。
```sh
clasp create gas-project
# standalone, api, sheet, doc など初期テンプレートをどれにするか聞かれるのでstandaloneを選ぶ
```
実行後にディレクトリ直下に　`.clasp.json` `appscript.json` が作成されます。

## プロジェクトの確認
GASプロジェクトが本当にGoogleサービス上で作成されてるかを以下コマンドで確認します。
ブラウザでGASエディター画面が開いて、プロジェクト名が左上に書いてあればOKです。
```
clasp open
```

こちらのリンクでも、GASプロジェクト一覧が確認できます。
https://script.google.com/home


# push設定
ローカルのファイルをGASサーバ上にpushするときの、対象ファイルを設定します。
今回はTypeScriptのコンパイルで生成したdistフォルダをGASサーバにプッシュするための設定をします。

実はGASはtypescriptの状態でpushをしても自動でjsにコンパイルしてプッシュしてくれるのですが、それだとtsconfigのコンパイル設定が適用できないため、ローカルでjsコンパイルをした結果のdistフォルダをプッシュするようにします

まず`.clasp.json`にdistフォルダをpush対象とする設定を追加します
```diff json:.clasp.json
{
  "scriptId": "<固有のスクリプトID>"
+ "rootDir": "dist" ＿
}
```

push対象の中には`appscript.json`を含める必要があるため、毎回`appscript.json`も`dist`フォルダに配置する必要があります。
コンパイル時に毎回配置するコマンドを打つのは面倒なので`package.json`にてコンパイル用のコマンドを作成しましょう(compile)。
さらにフォーマット〜コンパイル〜プッシュを同時にやるコマンドも作ってしまいます（push）。
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
clasp open # ブラウザでGASエディタが開くので、distの内容が反映されていることを確認
```