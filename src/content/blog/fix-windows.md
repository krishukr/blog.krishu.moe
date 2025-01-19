---
author: Kris Hu
pubDatetime: 2022-08-10T16:46:20.000+08:00
modDatetime:
title: 修复 Windows 休眠后窗口布局变化
featured: false
draft: false
tags:
  - How to
  - Windows
description: 省流：Windows Update 搞坏的。
---

:::caution
这篇文章记录的情况发生于 2022 年，可能与当前情况有所不同。
:::

Windows Update ：）

## Table of contents

## 特征

锁屏/睡眠后，显示器进入休眠模式。再次唤醒后窗口布局、大小改变。

检查 Windows 更新记录。

<img src="https://img-cdn.akass.cn/12/2022/08/62f3720a87845.png!wp" />

`NVIDIA - Display` 被更新。其更新时间应为问题开始出现的时间。

<img src="https://img-cdn.akass.cn/12/2022/08/62f374a87e187.png!wp" />

## 修复

**降级 `NVIDIA - Display` 驱动。**我选择的版本是 `30.0.14.7247`。

前往 [Microsoft Update Catalog](https://www.catalog.update.microsoft.com/Search.aspx?q=30.0.14.7247) 下载你对应设备的驱动包。

你可以在右上角的搜索框加上你的显卡型号帮助你快速查找。

<img src="https://img-cdn.akass.cn/12/2022/08/62f376d1bb57e.png!wp" />

下载得到 `.cab` 文件。创建一个文件夹存放解压的驱动文件。如 `D:\nv`。

在 `.cab` 文件所在位置打开 PowerShell，输入 `expand <...>.cab -F:* <D:\nv>`。

**注意 `<>` 及其内的部分应该按你的实际情况进行填写。**

_Win + X_ 打开设备管理器。

<img src="https://img-cdn.akass.cn/12/2022/08/62f377f6683f7.png!wp" />

右键你的显卡打开属性页面。

<img src="https://img-cdn.akass.cn/12/2022/08/62f3787a3c1ea.png!wp" />

选择 _更新驱动程序_。

<img src="https://img-cdn.akass.cn/12/2022/08/62f378ae5f8a3.png!wp" />

看图操作吧，懒得说了。

<!-- import MultiColumn from "@components/MultiColumn.astro"; -->

<img src="https://img-cdn.akass.cn/12/2022/08/62f378e337066.png!wp" />

<img src="https://img-cdn.akass.cn/12/2022/08/62f3790caf2b5.png!wp" />

<img src="https://img-cdn.akass.cn/12/2022/08/62f3793558d7c.png!wp" />

<img src="https://img-cdn.akass.cn/12/2022/08/62f37960889dc.png!wp" />

**请注意这里的 `D:\nv` 应该按你的实际情况进行填写。**

<img src="https://img-cdn.akass.cn/12/2022/08/62f379a83dbf1.png!wp" />

**请确认此处 _显示兼容硬件_ 已对勾**。

**请确认此处 _型号_ 为你的显卡型号**。

否则请尝试下载另一个驱动包。

_下一页_ 安装即可。

---

**_以下为可选项。_**

关闭 Windows 自动更新，以绝后患。

_Win + R_ 输入 `gpedit.msc` 打开组策略编辑器。

_计算机配置_ $\to$ _管理模板_ $\to$ _Windows 组件_ $\to$ _Windows 更新_ $\to$ _配置自动更新_。

<img src="https://img-cdn.akass.cn/12/2022/08/62f37b648f8a3.png!wp" />

更改为 _已禁用_。

<img src="https://img-cdn.akass.cn/12/2022/08/62f37b80d0cce.png!wp" />
