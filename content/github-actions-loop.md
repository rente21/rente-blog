---
title: 'Github Actionsで繰り返し(ループ)処理'
date: 2021-09-27
tags: ['Github Actions']
---

## 【結論】for 文

linux の標準機能である for 文を使用する。

【例】build フォルダの中身を s3-bucket1 と s3-bucket2 というバケットに一気にデプロイしたい場合

```yaml {fn="ci-cd.yaml"}
- name: Deploy
  run: |
    for BUCKET_NAME in s3-bucket1 s3-bucket2; do

    aws s3 sync build/ s3://$BUCKET_NAME

    done
```

for 文の形式

```shell
for VAR_NAME in a b c; do
  # 処理($VAR_NAMEで使用可能)
done
```

## おまけ

### if 文と組み合わせてみる

```shell
for BUCKET_NAME in s3-bucket1 s3-bucket2; do

if [ $BUCKET_NAME = "s3-bucket1" ]; then
  echo "deploy to 1!!"
fi

if [ $BUCKET_NAME = "s3-bucket2" ]; then
  echo "deploy to 2!!"
fi

...

done
```

if 文の形式

```shell
if [ 条件式 ]; then
  # 処理
fi
```

### 区切り文字から配列を展開

```shell
BUCKETS="s3-bucket1,s3-bucket2"
for BUCKET in ${BUCKETS//,/ }; do
...
done
```

文字列置き換えの形式

```shell
${VALUE_NAME/FROM/TO} # 最初の一個だけ置き換え
${VALUE_NAME//FROM/TO} # 全部置き換え
```
