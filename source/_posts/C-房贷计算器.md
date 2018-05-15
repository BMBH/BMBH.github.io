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

- 代码示例

```
            //[贷款本金×月利率×（1+月利率）^还款月数]÷[（1+月利率）^还款月数－1]

            double yearNum = Z.Base.Util.Parser.TryToDouble(this.cmbYear.Text, this.format);
            double monthNum = yearNum * 12;
            double gMoney = Z.Base.Util.Parser.TryToDouble(this.txtGongMoney.Text, this.format);//公积金
            double sMoney = Z.Base.Util.Parser.TryToDouble(this.txtShangMoney.Text, this.format);//商贷

            double gRate = Z.Base.Util.Parser.TryToDouble(this.txtGongRate.Text, this.format) / 100f;//公积金利率
            double sRate = Z.Base.Util.Parser.TryToDouble(this.txtShangRate.Text, this.format) / 100f;//商贷利率
            double sUp = Z.Base.Util.Parser.TryToDouble(this.txtShangUp.Text, this.format) / 100f;//商贷上浮

            double gMonthRate = gRate / 12f;
            double sMonthRate = (sRate * (1 + sUp)) / 12f;

            double gPower = Math.Pow(1 + gMonthRate, monthNum);
            double sPower = Math.Pow(1 + sMonthRate, monthNum);

            double gMonth = (gMoney * gMonthRate * gPower) / (gPower - 1);
            double sMonth = (sMoney * sMonthRate * sPower) / (sPower - 1);

            this.txt.Clear();
            this.txt.AppendText(string.Format("公积金贷款金额：{0} 万元 \r\n", gMoney.ToString(this.format)));
            this.txt.AppendText(string.Format("公积金每月还款：{0} 万元 \r\n", gMonth.ToString(this.format)));

            this.txt.AppendText(string.Format("商业贷款金额：{0} 万元 \r\n", sMoney.ToString(this.format)));
            this.txt.AppendText(string.Format("商业贷款每月还款：{0} 万元 \r\n", sMonth.ToString(this.format)));

            this.txt.AppendText(string.Format("总贷款金额：{0} 万元 \r\n", (sMoney + gMoney).ToString(this.format)));
            this.txt.AppendText(string.Format("总贷款每月还款：{0} 万元 \r\n", (sMonth + gMonth).ToString(this.format)));

            this.txt.AppendText(string.Format("还款月数：{0} \r\n", monthNum));
```

- GitHub

	[.NET-App/Loaner/](https://github.com/BMBH/.NET-App/tree/master/Loaner)