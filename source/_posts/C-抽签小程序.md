---
title: 'C# 抽签小程序'
date: 2018-05-14 09:56:54
tags: C#
categories: .NET
---

### C# 抽签小程序

- 设计背景

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

- 代码示例


	- 导入名单

```
            using (System.Windows.Forms.OpenFileDialog dialog = new OpenFileDialog())
            {
                dialog.InitialDirectory = Application.StartupPath;
                dialog.Filter = "Excel文件(*.xls)|*.xls";
                dialog.CheckFileExists = true;
                if (dialog.ShowDialog(this) == System.Windows.Forms.DialogResult.OK)
                {
                    this.txtList.Text = dialog.FileName;
                    this.dt = ImportExcel(dialog.FileName, false, false);
                }
            }
```

	- 随机抽签



```
            Random rd = new Random();
            this.lbShow.Text = this.dt.Rows[rd.Next(1, this.dt.Rows.Count)][0].ToString();
```

	- 评分导出


```
            string apppath = Application.ExecutablePath;
            apppath = apppath.Substring(0, apppath.LastIndexOf("\\"));
            if (File.Exists(apppath + "\\Evaluation.xml"))
            {
                XmlDocument xmlDoc = new XmlDocument();
                xmlDoc.Load(apppath + "\\Evaluation.xml");
                XmlNodeList nodeList = xmlDoc.SelectNodes("//Evaluation//Evaluation");

                DataTable dt = new DataTable();
                dt.Columns.Add("姓名");
                dt.Columns.Add("评分");
                dt.Columns.Add("时间");

                for (int i = 0; i < nodeList.Count; i++)
                {
                    DataRow dr = dt.NewRow();
                    XmlElement xe = (XmlElement)nodeList[i];
                    dr[0] = ((XmlElement)xe.SelectNodes("//Evaluation//Name")[i]).InnerText;
                    dr[1] = ((XmlElement)xe.SelectNodes("//Evaluation//Points")[i]).InnerText;
                    dr[2] = ((XmlElement)xe.SelectNodes("//Evaluation//Time")[i]).InnerText;
                    dt.Rows.Add(dr);
                }
                ExportTo("Evaluation", "Evaluation", dt);
                MessageBox.Show("导出评分表将删除评分记录，请自行保存！");
                File.Delete(apppath + "\\Evaluation.xml");
            }
            else
            {
                MessageBox.Show("无评分记录！");
            }
```

- GitHub

	[.NET-App/Draw/](https://github.com/BMBH/.NET-App/tree/master/Draw)