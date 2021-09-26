---
title: '【Laravel+React】どっちもhttps化してみる(他デバイスでの確認も可能)'
date: 2021-09-26
tags: ['Laravel', 'React']
---

API+SPA で Laravel+create-react-app を使用していて、どちらも https 化してみた。  
今回の実装パターンだと、スマホなどの他デバイスでも https でアクセスできる。

## Laravel は valet で https 化

```shell
sudo apachectl stop # apacheを起動している場合は停止(nginxを基盤としているため)

composer global require laravel/valet # valetをインストール
valet # 初期セットアップ

cd project-path # laravelのフォルダに移動
valet link # valetに紐付け
valet secure # https化

php artisan serve # サーバーを起動してない場合は起動
```

これにより https://[laravel プロジェクトのフォルダ名].test でアクセスできる。

しかし、今回はスマホなどでも確認したいため、「php artisan serve」は稼働させつつ、以下のコマンドを打つ。

```shell
valet share
```

するとこんな画面が表示される。

![](/images/3.png)

これにより https://\*\*\*.ngrock.io で他デバイスからでもアクセスできる。

ただこれは一時的にではあるが、**url を知っていれば誰でもアクセスできる**ため、その点は注意だ。

## create-react-app は デフォルトで可能

```shell
HTTPS=true yarn start
```

これにより https://localhost:3000、他デバイスからはhttps://192.168.1.2:3000 などでアクセスできる。

192.168.1.2（wifi 等の同じネットワーク内だけでアクセスできる url）の部分は、サーバー起動時の出力で分かる。

![](/images/4.png)

## おまけ

package.json の scripts にこんな追記をしてみる。

```json {fn="package.json"}
{
  ...
  "scripts": {
    ...
    "https": "HTTPS=true REACT_APP_NGROCK_ID=$ID yarn start",
    ...
  },
  ...
}
```

すると

```shell
ID="***.ngrock.ioにあった***部分のID" yarn https
```

でアプリを起動すると、アプリ内で「**process.env.REACT_APP_NGROCK_ID**」で ngrock の id を参照可能だ。

axios などを使用していて、API の url を共通化してたりする場合に便利だ。（以下実装例）

```ts
let api_url = '';
if (
  window.location.protocol === 'https:' &&
  process.env.NODE_ENV === 'development'
) {
  api_url = `https://${process.env.REACT_APP_NGROCK_ID}.ngrok.io`;
}
```

以上の作業により、SPA と API どちらも https 化されたアプリケーションを、複数デバイスからテストすることができる。
