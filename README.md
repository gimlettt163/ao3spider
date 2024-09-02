暂时就这样啦，文风内容分析还需要再考虑。

# 爬取基础数据

数据包括：文章标题、作者、发布时间、浏览量、点赞量、评论数、标记/收藏数，以及文章连接

所需的库：

```python
import re#正则表达式
import requests#请求网页
from bs4 import BeautifulSoup#用于解析 HTML，和 XML文档
import urllib.request, urllib.error  # 制定URL，获取网页数据
from urllib.parse import quote#中文检索词转编码
from openpyxl import Workbook#写入文件
from openpyxl import load_workbook#写入文件
import xlsxwriter#写入文件
import csv#写入文件
import pandas as pd
```

获取单篇文章信息：

```python
def GetInfo(html):
    if html is None:
        raise ValueError("HTML内容未定义")
    html = html.prettify()
    soup = BeautifulSoup(html,"html.parser",from_encoding="utf-8")
    title_tag = soup.find('a')
    url=title_tag.attrs["href"]
    title = title_tag.get_text(strip=True) if title_tag else "未找到标题"
    # 找到作者的<a>标签并提取文本
    author_tag = soup.find('a', rel="author")
    author = author_tag.get_text(strip=True) if author_tag else "未找到作者"
    comments_text= soup.find('dd', class_='comments')
    
    
    date=soup.find('p', class_='datetime').get_text(strip=True)
    
    if comments_text:
        comments = comments_text.get_text(strip=True)
    else:
        comments=0 
    bookmarks= soup.find('dd', class_='bookmarks')
    if bookmarks:
        bookmarks = bookmarks.get_text(strip=True)  
    else:
        bookmarks=0    
    if soup.find('dd', class_='kudos'):
        kudos=soup.find('dd', class_='kudos').get_text(strip=True)
    else:
        kudos=0
    if soup.find('dd', class_='hits'):
        hits=soup.find('dd', class_='hits').get_text(strip=True)
    else:
        hits=0  
    stats = {
    "标题":title,
    "作者":author,
    "点赞":kudos,
    "点击":hits,
    "评论":comments,   
    "收藏":bookmarks,
    "date":date,
    "url":"https://archiveofourown.org"+url
    }
    return stats
```

返回html信息（此模块代码来自CSDN非原创）：

```python
def askURL(url):
    head = {  
        "User-Agent": "Mozilla / 5.0(Windows NT 10.0; Win64; x64) AppleWebKit / 537.36(KHTML, like Gecko) Chrome / 80.0.3987.122  Safari / 537.36"
    }
    # 用户代理，表示告诉豆瓣服务器，我们是什么类型的机器、浏览器（本质上是告诉浏览器，我们可以接收什么水平的文件内容）
    request = urllib.request.Request(url, headers=head)
    html = ""
    try:
        response = urllib.request.urlopen(request)
        html = response.read().decode("utf-8")
    except urllib.error.URLError as e:
        if hasattr(e, "code"):
            print(e.code)
        if hasattr(e, "reason"):
            print(e.reason)
    return html
```

获取检索词共有几个返回网页：

```python
def NumsOfPages(url):
    html=askURL(url)
    # 定义HTML内容
    soup = BeautifulSoup(html,"html.parser",from_encoding="utf-8")
    PagesInfo=soup.find('ol',class_="pagination actions")
    if PagesInfo is None:
        return "未找到分页信息"
    num=0
    for item in PagesInfo.find_all('li'):
        pages = item.find('a')
        pages = item.get_text(strip=True) if item else "未找到标题"
        if(pages.isdigit()):
            num=pages
    return int(num)
```

Main Code

```python
if __name__ == "__main__":
    
    chinese_str = input("请输入查询角色：")
    file_path=chinese_str+'.csv'
    
    encoded_url = quote(chinese_str)
    base_url='https://archiveofourown.org/works/search?'
    query_url='work_search%5Bquery%5D='+encoded_url
    url=f"{base_url}&{query_url}"
    
    
    num=NumsOfPages(url)
    num=int(num)
    begin_url='https://archiveofourown.org/works/search?'
    end_url='&work_search%5Bquery%5D='+encoded_url
    
    fieldnames = ["标题", "作者", "点赞","点击", "评论", "收藏","url","date"]
    
    with open(file_path,'w', newline='',encoding='utf-8-sig') as csv_file:
        writer = csv.DictWriter(csv_file, fieldnames=fieldnames)       
        writer.writeheader()
        for page in range(1, num + 1):
            url = f"{begin_url}page={page}&{end_url}"
            html = askURL(url)  # 保存获取到的网页源码
            soup = BeautifulSoup(html, "html.parser")
            for item in soup.find_all('li', role="article"):
                singleInfo=GetInfo(item)
                writer.writerow(singleInfo)
                
        csv_file.close()

```

# 初步分析

爬取基本信息后可以进行初步数据分析，初步分析主要是从发布时间到如今时间间隔、点赞量、浏览量、评论、收藏数之间的相关关系。







# 文本分析

初步分析结束后，搜集点赞、浏览、点赞比前十的文章再进行文本分析，分析这些文章的特征：

题材、背景、高频词、主题

有些可以靠统计如jieba这种分词库统计出来，有些可能需要使用AI，比如分析文风之类的。
