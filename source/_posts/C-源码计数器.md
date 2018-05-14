---
title: C# 源码计数器
date: 2018-05-11 08:47:38
tags: C# 
categories: .NET
---

### C# 源码计数器

- 应用场景

	编程工作中，有些文档需要填写代码量，例如申请软件著作权。查阅相关资料之后，编写了这个小程序。

- 编程思路

	主要思路为分析项目文件，根据项目文件查找代码文件，然后遍历代码文件进行分析

- 相关技术
	- 抽象类
	- 文件操作
	- 字符串解析

- 项目结构
	- 抽象类
		- FileAnalyser 文件分析类，负责校验文件，并定义分析文件方法
		- ProjectDocument 项目工程类，负责定义项目文件属性及方法
	- 实体类
		- FileEntity 文件实体类，负责存储代码文件信息
		- ProjectFileEntity 项目工程实体类，负责存储项目文件信息
		- LineEntity 代码行实体类，负责存储代码行信息
	- 集合类
		- FileCollection 文件实体集合，负责文件集合统计数据
	- 功能类
		- CSProjectDocument C#项目工程类，继承ProjectDocument类，负责分析C#项目工程文件
		- CSFileAnalyser cs文件分析类，继承FileAnalyser，负责分析cs文件

- 代码示例
	
	调用方法
```
            ProjectDocument mDocument = ProjectDocument.Create(this.txtFileName.Text);
            if (mDocument == null)
            {
                MessageBox.Show("Analyse for " + this.txtFileName.Text + " error !");
                return;
            }
            mDocument.ClearResults();
            mDocument.AnalyseAllFile();
```


- GitHub地址

	[BMBH/.NET-App](https://github.com/BMBH/.NET-App/tree/master/CodeCalc)
