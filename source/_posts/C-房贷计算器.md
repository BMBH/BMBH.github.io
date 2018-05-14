---
title: 'C# 房贷计算器'
date: 2018-05-14 12:21:36
tags: C# 
categories: .NET
---

### C# 房贷计算器

- 应用场景

	百度小程序中的房贷计算器不能满足我个人的需求，故而开发一个.NET小程序。希望后期能用JS重写，发布在网上供大家使用。

- 设计思路

	根据百度公式：等额本息月还款 = [贷款本金×月利率×（1+月利率）^还款月数]÷[（1+月利率）^还款月数－1]

- 相关技术

	- WinForm 键入事件
	- 字符串与浮点型数据转换

- 功能

	键入相关数据， 进行计算即可

- GitHub

	[.NET-App/Loaner/](https://github.com/BMBH/.NET-App/tree/master/Loaner)