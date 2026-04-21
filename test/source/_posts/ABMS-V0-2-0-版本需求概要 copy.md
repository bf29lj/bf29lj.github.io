---
title: ABMS V0.2.0 版本需求概要
date: 2026-04-21 16:53:10
categories:
	- 项目开发
	- 需求
tags:
	- ABMS
sticky: true
toc: true
mathjax: true
home_cover: https://images.unsplash.com/photo-1517248135467-4c7edcad34c4?auto=format&fit=crop&w=1200&q=80
home_cover_height: 220
post_cover: https://images.unsplash.com/photo-1454165804606-c3d57bc86b40?auto=format&fit=crop&w=1600&q=80
post_cover_height: 320
reward: false
copyright_info: false
author: bf39lj
---

在 `ABMS V0.1.0` 中，我们已经使用 `iDME` 实现了整个 `ABMS` 的基础框架，整体完成度是比较高的。接下来我们要具体进一步使用 `iDME` 的内置功能对其中的功能进行改写和扩展。

<!-- more -->

本文分为两个部分，一个是 `V0.2` 新的需求和解决方案，一个是 `V0.1` 建议改进的一些问题。也希望大家能积极参与项目的建设。

## V2.0 新的需求

新的需求的主要技术要点是：

> 使用BasicObject、VersionObject、分类管理、版本服务、编码生成器等xDM-F内置的抽象数据模型和功能，实现miniAPP功能要求提及的版本管理、分类管理、扩展信息等功能。

要修改的实体包括设备，物料，物料分类，BOM。

### 设备扩展属性

#### 问题

前期我们完成了设备的扩展属性功能，这个功能是使用的 iDME 的内置功能实现的，问题不大。

新增的功能如下：

> 预置设备类型、各类型对应的扩展信息等基础数据，以支持创建设备时的选择类型信息，创建设备、更新设备时录入相应的扩展信息。

这个功能要求创建并记录设备的模板，扩展属性的配置和各个属性的默认值。

比如 `立式加工中心` 有如下的模板

![1776766506223](/images/posts/abms-v0-2-0/1776766506223.png)

基本信息是在实体属性里面定好的，但是扩展属性需要在属性库中由用户自定义。模板中要包含扩展属性的配置信息，在这个例子中，扩展属性配置要包括工作台尺寸（字符串类型），X/Y/Z轴行程（字符串类型），主轴转速（字符串类型） 等等。

#### 解决方案

比较简单的想法是创建一个 JSON 文件，如下所示：

```json
{
 // 基础属性默认值
  "no": "MC-VMC-001",
  "name": "立式加工中心",

 // 扩展属性创建 + 赋值请求
  "extAttrsRequests": [
    {
      "nameEn": "voltage",
      "name": "电压",
      "type": "DECIMAL_WITH_PRECISION",
      "description": "设备电压参数",
      "value": "4.5"
    },
    {
      "nameEn": "workspaceSize",
      "name": "工作台尺寸",
      "type": "STRING",
      "description": "工作台的桌面大小",
      "value": "850 × 500 mm"
    }
  ],

 // 其他默认值
}
```

为了方便存储，可引入 **设备模板实体 (EquipmentTemplate)** 其继承于 `Equipment` 实体，新增属性 `extAttrsRequests`，其为 JSON 类型。

这个新增的属性到后端，做的事情就是，首先调用创建扩展属性接口，将 `nameEn`，`name`，`type` 和 `description` 作为请求参数进行创建，在属性库中创建并且建立连接，然后提取基础属性和扩展属性的值，将默认值填入新增设备的对话框中，由用户自己决定修改哪些值然后用户自己提交。

为了管理设备模板实体，需要对设备模板进行 CRUD 操作的基本维护，还有应用模板，从模板创建等功能。

---

建议阅读官方文档：[https://support.huaweicloud.com/qs-idme/idme_qs_0001.html]
