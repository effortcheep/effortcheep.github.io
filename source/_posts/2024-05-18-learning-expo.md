---
title: learning-expo
date: 2024-05-18 13:15:27
tags:
- React Naive
- expo
---

## 学习 Expo 开发

#### 使用 Expo 开发

项目根目录下没有 ios 和 android 文件夹
- 不支持需要原生配置的第三方库（不在 Expo SDK 内的）
- 不支持集成原生代码

**[router路由问题](https://github.com/expo/expo/issues/28898)**

```sh
npx create-expo-app@latest --template

## 要在网络上运行该项目，我们需要安装以下依赖，这将有助于在网络上运行该项目：
npx expo install react-dom react-native-web @expo/metro-runtime
```

#### [使用 development builds](https://expo.nodejs.cn/guides/local-app-development/)

- 支持安装任何第三方库
- 支持原生代码集成
- 支持修改任何项目配置

#### 使用Reac Native 开发，集成 Expo SDK 库


## UI 库

[gluestack](https://gluestack.io/)
