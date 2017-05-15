---
title: 利用 Python 获取豆瓣电影排行榜 Top250 的数据
date: 2016-04-24 08:57:29
tags: spider
categories: Python
---

平时利用在地铁公交的时间简单学习了解了 Python，学习参考的是[廖雪峰的Python教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)，但是要想学好，还是要上手练习才会有好的效果，于是周末就利用 Python 写了一个简单的爬虫，来获取豆瓣电影排行榜 Top250 的数据，实现了保存数据到文件，保存海报图片。

我利用的是 [requests](http://www.python-requests.org/en/master/) (HTTP for Humans ,哈哈)和 [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/)，具体使用方法可参考官网，安装方式：
``` bash
pip install requests
pip install beautifulsoup4
```
安装好以后就可以开始了，首先打开[豆瓣电影排行榜](https://movie.douban.com/top250)，查看网页源码
![html](http://7xrac3.com1.z0.glb.clouddn.com/Douban_Top250.png)
箭头部分就是我们需要的数据，包括：排名，名字，评分，评分人数，短评，同时当我们查看下一页的时候，发现链接格式是这样的
"https://movie.douban.com/top250?start=25"
我们可以得知网页是按 25 来分页的，于是代码如下：
``` python
 def movies_spider(start):
    global file_content
    url = "https://movie.douban.com/top250?start=%d" % start
    response = requests.get(url).text
    soup = BeautifulSoup(response)
    movie_list = soup.find_all('div', {'class': 'item'})
    for item in movie_list:
        order = int(item.find('em').string)
        icon_path = item.find('img').get('src')

        info = item.find('div', {'class': 'info'})
        name = info.find('span', {'class': 'title'}).string
        save_pic(icon_path, name)
        rating_num = info.find('span', {'class': 'rating_num'}).string
        total = info.find('span', {'class': 'rating_num'}).find_next_sibling().find_next_sibling().string
        inq = info.find('span', {'class': 'inq'})
        try:
            quote = inq.get_text()
        except AttributeError:
            quote = 'None'
            print("Type error")

        file_content += "%d\t%s\t评分:%s\n\t%s\t简评:%s\n\n" % (order, name, rating_num, total, quote)
```

这就是核心部分，其实只要结合 BeautifulSoup 的文档以及网页源码，还是挺容易理解的，那剩下的部分就是保存文件和保存图片，代码已经放到 Github ,有需要的可前往查看 [Github](https://github.com/Zhiw/CrawlerPractise/blob/master/douban/movie_top250.py)

参考链接
- [廖雪峰的 Python 教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)
- [requests](http://www.python-requests.org/en/master/)
- [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/)
- [豆瓣电影排行榜](https://movie.douban.com/top250)

