---
title: 'C# 抽签小程序'
date: 2018-05-14 09:56:54
tags: C#
categories: .NET
---

### C# 抽签小程序

- 应用场景

	设置一个Excel名单表，对名单进行随机抽取。

- 设计思路

	使用Timer定时器，运行定时器进行名单随机滚动，停止定时器获得抽签结果

- 相关技术

	- 随机数
	- Excel读取/导出
	- XML文档读写

- 相关类库

	- C1.C1Excel Excel操作相关

- 功能

	- 读取Excel名单
	- 名单随机抽签
	- 评分功能
	- Excel导出功能

- GitHub

	[.NET-App/Draw/](https://github.com/BMBH/.NET-App/tree/master/Draw)