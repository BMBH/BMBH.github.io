---
title: 'C# QR二维码DEMO'
date: 2018-05-11 12:01:32
tags: C#
categories: .NET
---

#### QR二维码

	二维码的一种

#### 相关类库

	ThoughtWorks.QRCode 第三方类库

#### DEMO功能

- Encode 生成二维码图片

	- Encoding 编码
	- Correction Level 等级
	- Version 版本
	- Size 大小

- Decode 解密二维码
	
#### 生成二维码代码

```
            ThoughtWorks.QRCode.Codec.QRCodeEncoder qrEncoder = new QRCodeEncoder();
            qrEncoder.QRCodeEncodeMode = this.GetQRCodeEncodeMode();
            qrEncoder.QRCodeErrorCorrect = this.GetQRCodeErrorCorrect();
            qrEncoder.QRCodeVersion = this.GetQRCodeVersion();
            qrEncoder.QRCodeScale = this.GetQRCodeScale();
            Bitmap image = qrEncoder.Encode(this.txtData.Text, Encoding.Default);
```

#### 解密二维码代码

```
            try
            {
                QRCodeDecoder decoder = new QRCodeDecoder();
                //QRCodeDecoder.Canvas = new ConsoleCanvas();
                String decodedString = decoder.decode(new QRCodeBitmapImage(new Bitmap(this.picDecode.Image)));
                this.txtShow.Text = decodedString;
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
```

#### GitHub

[.NET-Demo/QRCode/](https://github.com/BMBH/.NET-Demo/tree/master/QRCode)

