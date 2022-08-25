---
title: 一个简单的web IM程序
date: 2022-08-25 15:24:03
tags:
categories:
---


### 主要技术栈
- React
- Indexeddb(Dexie)
- Socket.io
- Nodejs + Express + typeorm + Mysql + Redis
- Pnpm monorepo
- Docker compose + Github Actions CI

实现细节请查看github仓库: [simple-chat](https://github.com/lilong7676/simple-chat)

<!-- more -->

### 目前主要功能:
- user register & login
- single chat

### start dev:

```
$ pnpm install && npm run dev-server && npm run dev-static
```
then open *localhost:3000/app* in browser.

### build for production:
```
$ docker compose up
```
then open *localhost/app* in browser.
