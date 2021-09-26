---
title: 'S3のファイルがたまに403になる理由'
date: 2021-09-21
tags: ['AWS', 'S3']
---

オブジェクトを公開しているのにも関わらず、一部のファイルが 403 になることがあるので、その時の対策。

## 【結論】CORS 設定がない

①S3 のバケット画面を開き「アクセス許可」を選択

![](/images/1.png)

②CORS 設定までスクロール

![](/images/2.png)

③「編集」を押し、以下のコードを貼り付け保存

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
