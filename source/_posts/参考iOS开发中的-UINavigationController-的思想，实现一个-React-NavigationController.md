---
title: 参考iOS开发中的 UINavigationController 的思想，实现一个 React NavigationController
date: 2022-08-25 15:33:09
tags:
categories:
---


#### 参考iOS开发中的 UINavigationController 的思想，实现了在React项目中的类似 iOS 页面转场的效果。
github：[navigation-controller](https://github.com/lilong7676/simple-chat/tree/main/packages/navigation-controller)

- 使用方式
  ```
  $ npm install @lilong767676/navigation-controller
  ```
<!-- more -->

  ```javascript
  import { NavigationController } from '@lilong767676/navigation-controller';


  // 先创建路由
  // 路由名 - 页面 配置
  export const routes = {
    root: App,
    [RouteNames.addFriendPage]: AddFriendsPage,
    [RouteNames.newFriendsPage]: NewFriendsPage,
    [RouteNames.chatHome]: ChatHome,
    [RouteNames.chatPage]: ChatPage,
  };

  // 初始化 NavigationController,单例模式，保证全局唯一
  const naviController = new NavigationController(routes);

  // 设置 rootContainer
  naviController.setRootContainer(ele);

  // 页面跳转
  NavigationController.push(RouteNames.addFriendPage, props);

  // 返回上一页面
  NavigationController.pop();

  // 销毁 NavigationController
  naviController.destroy();

  ```


#### 效果：
![](https://raw.githubusercontent.com/lilong7676/Picture/master/blog/image/2022-08-17%2014.56.19.gif)

