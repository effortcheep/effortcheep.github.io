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


## expo-router

可以使用默认的文件系统路由方式

[也可以是用旧的版本](https://github.com/craftzdog/react-native-animated-todo)


## 初始化项目流程

创建项目
```bash
pnpm create expo-app

# 需要 原生包
# https://docs.expo.dev/workflow/continuous-native-generation/
pnpx expo prebuild --platform ios/android

# 构建 eas build 生成 eas.json 文件
# https://docs.expo.dev/build/setup/
eas build:configure
```

本地 build

```bash
# 注意 Gradle 和 JDK 版本兼容
# expo 53  gradle 8.13  jdk 17.0
# pnpx expo run:android 此命令会报错 https://github.com/expo/expo/issues/28703
npx expo run:android

# 生成环境 打包流程 https://docs.expo.dev/guides/local-app-production/

cd android
## 打包 aab
./gradlew app:bundleRelease
## 打包 apk
./gradlew app:assembleRelease
```

减小包体积 在 android/app/build.gradle 文件中

```groovy
android {
  ...
  defaultConfig {
    ...
  }
  splits {
    abi {
      reset()
      enable true
      universalApk false  // 不生成通用APK
      include "arm64-v8a"  // 只打包这两种架构
    }
  }
  ...
}
```

