---
layout: post
title: 一个简单爬虫处理流程
categories: Python Spider
---

一个简单爬虫处理流程
========================================

本篇博客介绍了一个爬虫的基本实现，使用Python语言来实现一个简单爬虫，需要读者具有基础的网络概念和编程基础，如Html，Python等。





一、爬虫的基本结构
----------------------------------------

![爬虫基本流程][爬虫基本流程]

如上图，基本上任何一个主流爬虫的实现都是基于以下步骤：

+ 给出爬虫的初始URL，爬虫从初始URL开始爬取
+ 爬虫获取到一个URL对应的Html代码
+ 分析该Html代码获取数据
  - 获取目标数据（如博客文章，图片等）
  - 下一次请求使用的URL（只需要一条初始URL，爬虫就可以自动爬取所有网页）

一个爬虫基本上都会有一个URL队列，用来存储需要爬取的网页URL。爬虫从队列中逐条取出URL，获取和处理HTML代码，从源代码中获取目标数据，同时还要获取新的URL加入到URL队列中，这样，一个爬虫就可以自动爬取一个网站中的所有网页，而不需要我们给出所有的URL地址。

### 库的安装

在Python语言中，我们可以使用多个库的组合使用来实现一个爬虫，如使用requests或urlib等库来向服务器请求html代码，beautilfousoup和lxml库来解析Html代码获取数据。

使用pip工具来安装requests, beautifulsoup库：

```
# pip install requests
# pip install beautlfulsoup
```

在Python中导入这两个库，查看是否成功，下面显示了文章中使用的库的版本：

```
>>> import requests
>>> import bs4
>>>
>>> requests.__version__
'2.13.0'
>>> bs4.__version__
'4.5.3'
```

在这里不会介绍这两个库的使用方法，大家可以去它们的官网查看详细的教程，文档和教程：[Beautifulsoup4文档][Beautifulsoup4文档]、[Requests文档][Requests文档]。

**NOTE：** 使用requests, bs4这些基本库来实现一个爬虫需要自己处理各种错误，如404网页返回错误，html解析错误等，十分麻烦，所以我建议大家使用Scrapy库，Scrapy库是一个Python语言的爬虫库，它实现了基本的爬虫功能，如获取html网页代码，错误处理等功能，开发者只需要专注于编写解析HTML页面的代码，大大减少了开发复杂度，非常方便。





二、初始URL和URL队列
----------------------------------------

每一个爬虫都应该拥有一个获取下一条请求URL的方法，这个方法会提供一个URL来供爬虫程序使用。方法可以是列表，生成器或者函数，在这里使用一个使用一个列表来存储URL。

一个URL队列一开始必须拥有一个或多个URL来供爬虫启动时使用，爬虫处理完一个URL之后，在把获取到的新的URL加入到队列中去，如此循环。

为了简单起见，我们在这里使用列表来实现这个URL队列，代码如下：

```
urls = ["http://quotes.toscrape.com"]
```

### 请求Html源代码

一个网页的本质就是一个Html文件，我们使用requests库来获取quotes.toscrape.com网站上的页面

```
def request(url):
	resp = requests.get(url)

	if resp.status_code != 200:
		raise Exception("Request fails with http code {0}".format(resp.status_code))
	else:
		return resp.text  # HTML代码

```

在这里我只简单验证了是否成功获取到页面（返回200），但一个健壮的爬虫程序需要更多的错误处理来保证程序的正确运行。





三、获取所需数据
----------------------------------------

获取到Html代码之后就是从中获取数据了，我们使用beautifulsoup4库来解析页面代码。

在编写代码之前我们需要知道Html页面的结构，打开quotes.toscrape.com网站，并使用浏览器自带的源代码工具来查看源代码。

![html源代码][html源代码]

我们可以看到，目标数据（名人名句）被包含在一个text类的span中，知道了目标数据的位置，我们就可以编写代码了。

```
def parse(html):
	soup = bs4.BeautifulSoup(html, "html.parser")  # html.parser是Python语言内置的xml解析器
	spans = soup.find_all("span", class_="text")  # 获取所有text类的span元素

	for span in spans:
		print(span.string)  # span元素的内容

```

爬虫获取数据的方法基本就是如此，分析HTML源代码，从html标签中获取数据，我们可以一次获取多种数据，如本例子中的作者，句子的标签等数据。





四、获取下一条URL
----------------------------------------

除了获取目标数据之外，我们还应该从页面中获取新的URL加入到URL队列中，不然爬虫在爬取完URL队列中的初始URL之后就会停止运行，而我们希望它可以继续爬取其他页面。

获取新的URL的步骤和获取数据一样，我们分析新的URL在页面的位置，在用bs4库把它提取出来。

![新的URL][新的URL]

我们可以看到我们需要的新的URL就位于`<li class="next">`标签下，我们修改函数`parser`，添加获取URL的代码，代码如下：

```
def parser(html):
	...

	url = soup.find("li", class_="next").a["href"]

	return "http://quotes.toscrape.com" + url
```





五、总体代码结构
----------------------------------------

我们已经实现了一个爬虫的所有基本功能，接下来给出代码处理流程：

```
urls = ["http://qoutes.toscrape.com"]

for url in urls:
	html = request(url)
	new_url = parser(html)
	url.append(new_url)
```

这就是一个主流爬虫的处理流程，网上所有的爬虫基本都是基于以上流程来实现的，只是实现的方法各不相同，希望这篇博客可以让大家对爬虫有一个基本的了解，谢谢大家。





[Beautifulsoup4文档]: https://www.crummy.com/software/BeautifulSoup/bs4/doc/

[Requests文档]: http://docs.python-requests.org/en/master/

[爬虫基本流程]: http://oru8n3wml.bkt.clouddn.com/2017-04-01-simple-spider-1.jpg

[html源代码]: http://oru8n3wml.bkt.clouddn.com/2017-04-01-simple-spider-2.jpg

[新的URL]: http://oru8n3wml.bkt.clouddn.com/2017-04-01-simple-spider-3.jpg