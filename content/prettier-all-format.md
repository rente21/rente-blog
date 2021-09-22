---
title: 'prettierで全てのファイルをformatする方法'
date: 2021-09-22
tags: ['prettier']
---

あるプロジェクトに.prettierrc 等の設定ファイルがあり、その設定で全てを format したい時

```zsh
yarn add -D prettier # プロジェクト自体にprettierを導入
yarn prettier --write src/**/**.tsx # src直下のファイル全てのtsxファイル
yarn prettier --write src/**/**.{ts, tsx} # src直下のファイル全てのts,tsxファイル
```
