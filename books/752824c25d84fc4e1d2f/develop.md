---
title: "APIを開発していく"
---
# APIを作る
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
    const data = JSON.stringify(result)
    return ContentService.createTextOutput(data)
        .setMimeType(ContentService.MimeType.JSON)
        .getContent()
}
```

GASをAPIとして利用した場合に利用できるメソッドはGETとPOSTのみとなります。
GETメソッドをリクエストした場合はdoGET、POSTメソッドをリクエストした場合はdoPOSTから処理が開始されます
またjsonとして結果を返却する場合は`returnJson()`の処理を通してあげる必要があります。

