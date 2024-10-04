---
title: "初试HarmonyOS - 实现一个简单的APP"
collection: project
type: "Project"
permalink: /project/harmonyos-1
date: 2024-10-04
---

摘要：基于HarmonyOS的ArkTS框架，我实现了一个简单的APP进行数据获取和展示

# 环境介绍
HarmonyOS是华为开发的操作系统，目前已经更新到了HarmonyOS NEXT——也就是常说的纯血鸿蒙，有很开阔的发展前景。

我们团队中的项目需要开发一个移动端的APP进行数据展示，本着追逐风口和支持国产的想法，我们选择基于HarmonyOS进行APP的开发。这是我第一次进行APP开发，当时的HarmonyOS还是支持JAVA和ArkTS两种语言进行开发，因为官方在ArkTS方面有相关说明，所以我选用ArkTS进行开发。

我们使用华为的DevEco Studio作为IDE，选用的SDK版本为3.1.0(API 9)

# 产品说明
我们的项目是对轴承进行故障检测和寿命预测，并将检测和预测的结果存储在服务器中，由移动端进行数据的获取和展示。因此，APP的主要需求就是从网络中获取数据和恰当的数据展示。

# 代码介绍

