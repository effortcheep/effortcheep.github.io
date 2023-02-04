---
title: 如果在win11上获取wifi密码
date: 2023-02-04 21:26:02
tags:
  - wifi
  - win11
  - cmd
---

```sh

# 显示 WLAN 上的配置文件
> netsh wlan show profiles


# 显示 配置 详细信息
> netsh wlan show profile name="${name}" key=clear

```
