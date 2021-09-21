---
title: 'Sentryのトランザクションの制限アラートが来た'
date: 2021-09-20
tags: ['Sentry']
---

Sentry からこんなアラートが来た。

```go
Approaching Transactions Quota
Your organization [organization name] has consumed 80% of its transactions capacity for the current usage period.
```

訳すと

```go
トランザクションクォータに近づいています
あなたの組織[組織名]は、現在の使用期間中にトランザクション容量の80％を消費しました。
```

おいおいマジか。

## 原因

Laravel で開発していたのだが、毎回リクエストごとにデータベースのロールバックを行えるよう、トランザクションを張っていた。

「SENTRY_TRACES_SAMPLE_RATE」というトランザクションの何%を送信するかというものに 1(100%)を指定していた。  
これを 0 にして解決。

```:.env
SENTRY_LARAVEL_DSN=https://******.sentry.io/******
SENTRY_TRACES_SAMPLE_RATE=1
```

↓

```:.env
SENTRY_LARAVEL_DSN=https://******.sentry.io/******
SENTRY_TRACES_SAMPLE_RATE=0
```
