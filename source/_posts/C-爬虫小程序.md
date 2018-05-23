---
title: 'C# 爬虫小程序'
date: 2018-05-15 16:21:51
tags: C# 
categories: .NET
---

#### 设计思路

	主要基于Http Get请求网页数据，进行分析。涉及递归调用，多线程提高效率，守护线程等。

#### 相关技术

- 抽象类
- 多线程 
- 队列
- Http Get请求
- 字符串解析
	
#### 项目结构

- AbsChain 

 职责链抽象类，负责定义HTML处理方法，定义递归处理方法等

- AbsThreadManager

线程管理抽象类，负责定义守望线程，管理多线程

- UrlQueue 

URL队列对象，管理URL队列

- Crawl

爬虫对象，负责结合URL队列与职责链，运行爬取功能

- HttpGet

HTTP GET请求类，负责获取HTML文本

- ThreadEntity

 爬虫线程，实体对象


#### 简单爬虫示例

	以下示例为一个简单的获取HTML页面文本示例，可以做到下载文本，并进行分析，可以说是最简单的爬虫

```
            WebClient wc = new WebClient();
            byte[] response = wc.DownloadData("http://www.weather.com.cn/weather/101120501.shtml");
            string ss = Encoding.UTF8.GetString(response);
```

#### 项目代码调用示例

- 创建继承类，继承职责链，负责具体爬虫方法


```
    public class NodeChain : AbsChain
    {
        #region 去除头部的'与"
        /// <summary>
        /// 去除头部的'与"
        /// </summary>
        /// <param name="url"></param>
        /// <returns></returns>
        private string RemoveQuotation(string url)
        {
            if ((url.IndexOf("'") == 0) || (url.IndexOf("\"") == 0))
            {
                url = url.Remove(0, 1);
                if (url.IndexOf("'") != -1)
                {
                    url = url.Remove(url.IndexOf("'"), 1);
                }
                if (url.IndexOf("\"") != -1)
                {
                    url = url.Remove(url.IndexOf("\""), 1);
                }
            }
            if (url.IndexOf(" ") != -1)
            {
                url = url.Remove(url.IndexOf(" "));
            }
            return url;
        }
        #endregion

        #region 处理网页
        /// <summary>
        /// 处理网页
        /// </summary>
        /// <param name="html"></param>
        protected override void Process(string html)
        {
            try
            {
                Regex re = new Regex(@"href=(?<web_url>[\s\S]*?)>|href=""(?<web_url>[\s\S]*?)""|href='(?<web_url>[\s\S]*?)'");
                MatchCollection mc = re.Matches(html);
                foreach (Match m in mc)
                {
                    string url = m.Groups["web_url"].ToString();
                    url = this.RemoveQuotation(url);
                    if (url.IndexOf("http://") != -1)
                    {
                        UrlQueue.GetInstance().Enqueue(url);
                    }
                }
                string title = string.Empty;
                re = new Regex(@"<title[\s\S]*?>(?<title>[\s\S]*?)</title>");
                Match temp = re.Match(html.ToLower());
                title = temp.Groups["title"].ToString();
                if (!string.IsNullOrEmpty(title))
                {
                    Console.WriteLine(string.Format("网页标题：{0}",title));
                    Console.WriteLine(string.Format("网页URL：{0}", this.Url));
                }
            }
            catch
            {
            }
        }
        #endregion
    }
```

- 创建线程管理继承类，负责重写新建职责链对象

```
    public class ThreadManager:AbsThreadManager
    {
        protected override AbsChain GetChainHeader()
        {
            return new NodeChain();
        }
    }
```

- 设置URL入口，运行爬虫

```
            try
            {
                Console.Title = System.Configuration.ConfigurationManager.AppSettings["Title"].ToString();
                Console.WriteLine("Process is running！");
                
                string url = System.Configuration.ConfigurationManager.AppSettings["URL"].ToString();
                UrlQueue.GetInstance().Enqueue(url);
                ThreadManager thread = new ThreadManager();
                thread.Start();
            }
            catch (Exception ex)
            {
            }
```

#### GitHub

[.NET-App/NetSpider/](https://github.com/BMBH/.NET-App/tree/master/NetSpider)
