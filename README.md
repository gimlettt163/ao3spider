# 想法
写同人没有评论反馈，很想知道自己处在什么水平，所以靠计算点赞比来估计。ao3比较古老，又不可能一篇篇文章算，用爬虫无疑是最快的办法。
# 过程
几年前尝试过爬虫，但到import requests和bs就over了……因为那时候抗拒代码又没有从心而发的需求……这次有了需求目标就很明确了。
我的理解大概是这样：网页用的html（*可能还有其他编码或者渲染模式？），每个模块的内容都有特定的格式或关键字<a herf>'dd'这样的标记，所以目标就是扒出这些标记然后读取数据。<br />
“扒出数据”和“读取数据”其实根本就是一步，soup.find()。所以最核心的模块还算简单：<br />
```
def GetInfo(html):
    if html is None:
        raise ValueError("HTML内容未定义")
    html = html.prettify()
    soup = BeautifulSoup(html,"html.parser",from_encoding="utf-8")
    title_tag = soup.find('a')
    title = title_tag.get_text(strip=True) if title_tag else "未找到标题"

    # 找到作者的标签并提取文本
    author_tag = soup.find('a', rel="author")
    author = author_tag.get_text(strip=True) if author_tag else "未找到作者"
    comments_text= soup.find('dd', class_='comments')
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
    "收藏":bookmarks   
    }
    return stats
```
