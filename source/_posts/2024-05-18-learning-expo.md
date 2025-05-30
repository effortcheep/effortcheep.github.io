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
# 写在最前面 请最好使用 npm
npm create expo-app

# 需要 原生包
# https://docs.expo.dev/workflow/continuous-native-generation/
npm expo prebuild --platform ios/android

# 构建 eas build 生成 eas.json 文件
# https://docs.expo.dev/build/setup/
eas build:configure
```

+ ​​Expo Go 的限制​​
  + Expo Go 是一个通用客户端，仅支持纯 JavaScript 项目（即未添加原生模块或自定义配置的 Expo 项目）。
  + 如果你通过 eas build 生成了本地原生二进制文件（如 .apk 或 .ipa），或使用了 expo prebuild 生成了原生代码（如 android/ 和 ios/ 目录），项目就脱离了 Expo Go 的兼容范围。

本地 build

+ 使用开发客户端（Development Build）​​
  + 通过 expo run:android 或 expo run:ios 在设备上安装一个​​自定义的开发客户端​​（包含你的原生代码）。
  + 调试方式：
    + 运行 expo start 启动开发服务器。
    + 在开发客户端中手动输入开发服务器 URL（如 exp://192.168.x.x:8081），或通过 QR 码（需配合 expo start --dev-client）。

```bash
# 注意 Gradle 和 JDK 版本兼容
# expo 53  gradle 8.13  jdk 17.0
# npx expo run:android 此命令会报错 https://github.com/expo/expo/issues/28703
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


### [关于热更](https://docs.expo.dev/eas-update/getting-started/#publish-an-update)

[自建热更平台](https://github.com/expo/custom-expo-updates-server)

```xml
<!-- EXPO_UPDATE_URL 值应包含项目的 ID -->
<meta-data android:name="expo.modules.updates.EXPO_UPDATE_URL" android:value="https://u.expo.dev/your-project-id"/>
<meta-data android:name="expo.modules.updates.EXPO_RUNTIME_VERSION" android:value="@string/expo_runtime_version"/>
<meta-data android:name="expo.modules.updates.UPDATES_CONFIGURATION_REQUEST_HEADERS_KEY" android:value="{'expo-channel-name':'your-channel-name'}"/>
```

修改 app.json

```json
{
  "expo": {
    ...
    "updates": {
      ...
      "requestHeaders": {
        "expo-channel-name": "your-channel-name"
      }
      ...
    }
    ...
  }
}
```


```bash
# 热更
eas update --channel [channel-name] --message "[message]"
```