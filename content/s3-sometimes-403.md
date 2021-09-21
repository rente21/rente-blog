---
title: 'S3のファイルがたまに403になる理由'
date: 2021-09-21
tags: ['AWS', 'S3']
---

オブジェクトを公開しているのにも関わらず、一部のファイルが403になることがあります。  
その時の対策です。

## 【結論】CORS 設定がない

S3のバケット画面を開き「アクセス許可」を押します。

![](/images/screenshot01.png)

CORS設定までスクロールします。

![](/images/screenshot02.png)

「編集」を押し、以下のコードを貼り付け保存します。

```shell
[
    {
        "AllowedHeaders": [
            "*"
        ],
        "AllowedMethods": [
            "GET"
        ],
        "AllowedOrigins": [
            "*"
        ],
        "ExposeHeaders": [],
        "MaxAgeSeconds": 3000
    }
]
```
