---
title: 'C# 妈妈再打我一下生成器'
date: 2018-05-16 13:44:32
tags: C# 
categories: .NET
---

### C# 妈妈再打我一下生成器

- 设计背景

	网上很火的一个“妈妈再打我一下”的漫画图片，给了网友无限的想象发挥空间，此小程序可以给图片添加配文的形式，快速生成图片

- 设计思路

	GDI+ 绘图技术，在图片基础上添加文字

- 相关技术

	GDI+

- 代码示例

	```
	            Image imag = pictureBox1.Image;
                Graphics g = Graphics.FromImage(imag);
                Font font = new System.Drawing.Font("宋体", 12, (System.Drawing.FontStyle.Bold));
                LinearGradientBrush brush = new LinearGradientBrush(new Rectangle(0, 0, imag.Width, imag.Height), Color.Red, Color.Red, 1.2f, true);
                g.DrawString(this.txtUp.Text, font, brush, 50, 150);
                g.DrawString(this.txtMiddle.Text, font, brush, 50, 320);
                g.DrawString(this.txtDown.Text, font, brush, 50, 500);
                g.Dispose();

                pictureBox1.Image = imag;

                Clipboard.SetDataObject(pictureBox1.Image);
	```

	
- GitHub

	[.NET-App/PicGenerater/](https://github.com/BMBH/.NET-App/tree/master/PicGenerater)
