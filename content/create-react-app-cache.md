---
title: 'create-react-app等のSPAで、キャッシュを回避する方法'
date: 2021-09-23
tags: ['SPA']
---

「create-react-app」や「Vue CLI」でアプリケーションを構築する際に、「Nuxt.js」や「Next.js」ではあまり見られない問題が発生することがある。  
キャッシュ問題だ。

デプロイ後、画面が真っ白になって、スーパーリロードしたら読み込まれるといった現象に遭遇したことがある人は多いはずだ。

最近この問題を回避できる方法を見つけたので、共有する。

## 【解決策】CDN(S3+Cloudfront) で配信

CDN のキャッシュクリアは非常に優秀なので、それを活用する。  
手順は以下の通りだ。(今回は S3+Cloudfront でのスタイルを示す)

1. S3 にビルド後のファイルをアップロード
2. Cloudfront を設定
3. リライト先を index.html からサーバーサイドの言語ファイルに書き換え、ファイル読み込みコマンドで Cloudfront のファイルを読み込む
4. デプロイごとに Cloudfront のキャッシュクリア

以下 Apache+PHP 構成での例を示す。  
環境に応じてカスタマイズして欲しい。

```ApacheConf {fn=".htaccess"}
Options -MultiViews
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.html [QSA,L]
```

↓

```ApacheConf {fn=".htaccess"}
Options -MultiViews
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ spa.php [QSA,L] # 変更したところ
```

```php {fn="spa.php"}
<?php
readfile("https://******.cloudfront.net/index.html");
```

こういった構成であれば、静的 Web サイトホスティングなどの、全部 SPA 構成にしなくても、キャッシュクリアを実現することができる。

たまにファイルが 403 の認可エラーになることがあるが、その際には S3 の CORS 設定に以下の内容を貼り付けていただきたい。　　
(参考：[S3 のファイルがたまに 403 になる理由 | rente blog](/s3-sometimes-403/))

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
