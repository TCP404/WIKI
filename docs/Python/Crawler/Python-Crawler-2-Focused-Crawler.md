---
title: Python【Crawler】聚焦爬虫
tags:
  - Python
  - 爬虫
categories:
  - Python
  - 爬虫
  - 基础
thumbnail: 'https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/thumbnail/python.png'
abbrlink: 28995
date: 2020-08-20 14:17:48
---

过滤爬取到的数据，取出想要的部分

<!--more-->

聚焦爬虫：爬取页面中指定的页面内容

-  编码流程
    1. 指定url
    2. 发起请求
    3. 获取响应数据
    4. 数据解析
    5. 持久化存储

- 数据解析分类
    1. 正则表达式
    2. bs4
    3. xpath (*)

数据解析原理
    - 解析的局部文本内容都会在标签之间或标签的属性中存储
    - 1. 进行标签的定位
    - 2. 标签或者标签对应的属性中存储的数据值进行提取（即解析~~~~）


## 正则表达式爬取

```python
import os
import requests
import re

# UA伪装
UA = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.89 Safari/537.36 Edg/84.0.522.44'
header = {'User-Agent': UA}

# 存储目录
folder = './qiushi'
if not os.path.exists(folder):
    os.mkdir(folder)


def main():
    # 分页处理
    for i in range(1, 3):
        url = f"https://www.qiushibaike.com/imgrank/page/{i}/"
        # 1. 爬取整张页面
        page_text = requests.get(url=url, headers=header).text
        # 2. 使用聚焦爬虫进行数据解析
        images = data_parse(page_text)

        length = len(images)  # 进度条所需

        for index, image in enumerate(images):
            # 3. 发起请求并获得数据
            image_content = requests.get(url=image, headers=header).content
            # 4. 持久化存储
            image_name = image.split('/')[-1]
            image_path = os.path.join(folder, image_name)
            with open(image_path, 'wb') as f:
                f.write(image_content)
                # 进度打印
                print(image_name + ' 下载成功！')
                print(f'{index} / {length}', end='\r')

        print(str(i) + "/ 2 page")


def data_parse(page_text):
    """ 解析出每张图片的url """

    # 分析网页后整理出正则表达式
    ex = r'<div class="thumb">.*?<img src="(.*?)" alt.*?></div>'
    images = re.findall(ex, page_text, re.S)
    images = ['https:' + x for x in images]  # 解析出来没有协议头，给增加上

    return images


if __name__ == "__main__":
    main()

--------------------------------------------------

# Output:

K7EUFEUIV3QY37P1.jpg 下载成功！
DFIAAL32X5J35JP2.jpg 下载成功！
...
NXUA4X1CMQP22UPP.jpg 下载成功！
FQILIKXCVMUIRXL8.jpg 下载成功！
1/ 2 page
95GRCYEUZANQ361J.jpg 下载成功！
2SXBFKSSK3JD3G2M.jpg 下载成功！
...
5YTNS4JH0PLZAO58.jpg 下载成功！
59CN77YAL198SM6M.jpg 下载成功！
2/ 2 page
```


## 示例网页

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>测试bs4</title>
</head>
<body>
    <div>
        <p>百里守约</p>
    </div>
    <div class="song">
        <p>李清照</p>
        <p>王安石</p>
        <p>苏轼</p>
        <p>柳宗元</p>
        <a href="https://www.song.com" title="赵匡胤" target="_self">
            <span>this is span</span>
            宋朝是最强大的王朝，不是军队的强大，而是经济很强大，国民都很有钱
        </a>
        <a href="" class="du">总为浮云能避日，长安不见使人愁</a>
        <img src="https://www.baidu.com/meinv.jpg" alt="">
    </div>
    <div class="tang">
        <ul>
            <li><a href="https://www.baidu.com" title="qing">清明时节雨纷纷，路上行人欲断魂。借问酒家何处有，牧童遥指杏花村。</a></li>
            <li><a href="https://www.163.com" title="qin">秦时明月汉时光，万里长征人未还。但使龙城飞将在，不教胡马度阴山。</a></li>
            <li><a href="https://www.126.com" alt="qi">岐王宅里寻常见，崔久堂前几度闻。正是江南好风景，落花时节又逢君。</a></li>
            <li><a href="https://www.sina.com" class="du">杜甫</a></li>
            <li><a href="https://www.dudu.com" class="du">杜牧</a></li>
            <li><b>杜小月</b></li>
            <li><i>度蜜月</i></li>
            <li><a href="https://www.haha.com" id="feng">凤凰台上凤凰游，凤去台空江自流。吴宫花草埋幽径，晋代衣冠成古丘。</a></li>
        </ul>
    </div>


</body>
</html>
```

> ![1](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Py/crawler/2-1.png)
> **以下爬虫示例均以此网页示例为基础**
有点丑，将就一下。


## bs4爬取

### 安装

win 下：
\>_ pip install bs4
\>_ pip install lxml

Linux 下：
\>_ pip install Beautifulsoup4


### 导入
> `from bs4 import BeautifulSoup`

### 使用
-  创建bs对象并传入待解析对象
    1. 传入待解析对象为本地文件
        ```python
        from bs4 import BeautifulSoup
        
        with open('./bs_test.html', 'r', encoding='utf-8') as f:
            bs = BeautifulSoup(f, 'lxml')    # 创建bs对象
        ```
    2. 传入待解析对象为网络请求
        ```python
        from bs4 import BeautifulSoup
        import requests
        
        url = "https://www.baidu.com"
        page_text = requests.get(url=url).text
        bs = BeautifulSoup(page_text, 'lxml')    # 创建bs对象
        ```

#### 定位数据


| 方法                                        | 用                                |        返值         |
| :------------------------------------------ |:---------------------------------- |--------------------- |
| tagName                                     | 返回首tagName标签                  | bs4.element.Tag       |
| find('tagName')                             | 返回首tagName标签                  | bs4.element.Tag       |
| find('tagName', class_/id/attrName='value') | 返回首属性为value的tagName标签     | bs4.element.Tag       |
| find_all('tagName')                         | 返回所符合要求的标签               | bs4.element.ResultSet |
| select(['selector' + ]'tagName')            | 通过CS选择器+标签名定位标签,返回多个 | bs4.element.ResultSet |
| select_one(['selector' + ]'tagName')        | 通过CS选择器+标签名定位标签,返回一个 | bs4.element.Tag       |

##### bs.tagName

1. `bs.tagName`：返回文档中**第一次**出现tagName对应的标签
    ```python
    with open('./bs_test.html', 'r', encoding='utf-8') as f:
        bs = BeautifulSoup(f, 'lxml')    # 创建bs对象
        
        tag_a = bs.a        # 定位数据
        
        print(tag_a)
        print(type(tag_a))  # <class 'bs4.element.Tag'>
    --------------------------------------------------
    # Output:
    <a href="https://www.song.com" target="_self" title="赵匡胤">
    <span>this is span</span>
        宋朝是最强大的王朝，不是军队的强大，而是经济很强大，国民都很有钱
    </a>
    <class 'bs4.element.Tag'>
    ```
##### bs.find()

2. `bs.find()`：
    1. `find('tagName')`：等同于 bs.tagName
        ```python
        with open('./bs_test.html', 'r', encoding='utf-8') as f:
            bs = BeautifulSoup(f, 'lxml')    # 创建bs对象
            
            tag_div = bs.find('div')         # 定位数据
            
            print(tag_div)
            print(type(tag_div))  # <class 'bs4.element.Tag'>
        --------------------------------------------------
        # Output:
        <div>
        <p>百里守约</p>
        </div>
        <class 'bs4.element.Tag'>
        ```
    2. `find('tagName', class_/is/attrName='value')`：通过限定属性来定位标签
        ```python
        with open('./bs_test.html', 'r', encoding='utf-8') as f:
            bs = BeautifulSoup(f, 'lxml')                  # 创建bs对象
            
            tag_a = bs.find('a', class_='du')        # 定位数据
            
            print(tag_a)
            print(type(tag_a))  # <class 'bs4.element.Tag'>
        --------------------------------------------------
        # Output:
        <a class="du" href="">总为浮云能避日，长安不见使人愁</a>
        <class 'bs4.element.Tag'>
        ```
    3. `bs.find_all('tagName')`：返回复合要求的所有标签（集合）
        ```python
        with open('./bs_test.html', 'r', encoding='utf-8') as f:
            bs = BeautifulSoup(f, 'lxml')       # 创建bs对象
            
            tag_p = bs.find_all('p')        # 定位数据
            
            print(tag_p)
            print(type(tag_p))  # <class 'bs4.element.ResultSet'>
        --------------------------------------------------
        # Output:
        [<p>百里守约</p>, <p>李清照</p>, <p>王安石</p>, <p>苏轼</p>, <p>柳宗元</p>]
        ```
##### bs.select()

1. `bs.select('selector' + 'tagName')`：可以通过**CSS择器+标签名**定位，包括层级选择器、标签选择器等
    ```python
    with open('./bs_test.html', 'r', encoding='utf-8') as f:
        bs = BeautifulSoup(f, 'lxml')       # 创建bs对象
        
        tag_div = bs.select('#feng')        # 定位数据
        
        print(tag_div)
        print(type(tag_div))  # <class 'bs4.element.ResultSet'>
    --------------------------------------------------
    # Output:
    [<a href="https://www.haha.com" id="feng">凤凰台上凤凰游，凤去台空江自流。吴宫花草埋幽径，晋代衣冠成古丘。</a>]
    <class 'bs4.element.ResultSet'>
    ```
    ```python
    with open('./bs_test.html', 'r', encoding='utf-8') as f:
        bs = BeautifulSoup(f, 'lxml')               # 创建bs对象
        
        tag1_a = bs.select('.tang > ul > li > a')  # 定位数据
        tag2_a = bs.select('.tang >ul a')          # 定位数据
        
        print(tag1_a)
        print(tag2_a)
        print(type(tag1_a))  # <class 'bs4.element.ResultSet'>
    
    --------------------------------------------------
    # Output:
    [<a href="https://www.baidu.com" title="qing">清明时节雨纷纷，路上行人欲断魂。借问酒家何处有，牧童遥指杏花村。</a>, <a href="https://www.163.com" title="qin">秦时明月汉时光，万里长征人未还。但使龙城飞将在，不教胡马度阴山。</a>, <a alt="qi" href="https://www.126.com">岐王宅里寻常见，崔久堂前几度闻。正是江南好风景，落花时节又逢君。</a>, <a class="du" href="https://www.sina.com">杜甫</a>, <a class="du" href="https://www.dudu.com">杜牧</a>, <a href="https://www.haha.com" id="feng">凤凰台上凤凰游，凤去台空江自流。吴宫花草埋幽径，晋代衣冠成古丘。</a>]
    [<a href="https://www.baidu.com" title="qing">清明时节雨纷纷，路上行人欲断魂。借问酒家何处有，牧童遥指杏花村。</a>, <a href="https://www.163.com" title="qin">秦时明月汉时光，万里长征人未还。但使龙城飞将在，不教胡马度阴山。</a>, <a alt="qi" href="https://www.126.com">岐王宅里寻常见，崔久堂前几度闻。正是江南好风景，落花时节又逢君。</a>, <a class="du" href="https://www.sina.com">杜甫</a>, <a class="du" href="https://www.dudu.com">杜牧</a>, <a href="https://www.haha.com" id="feng">凤凰台上凤凰游，凤去台空江自流。吴宫花草埋幽径，晋代衣冠成古丘。</a>]
    <class 'bs4.element.ResultSet'>
    ```
    ```python
     with open('./bs_test.html', 'r', encoding='utf-8') as f:
        bs = BeautifulSoup(f, 'lxml')       # 创建bs对象
        
        tag_div = bs.select_one('.du')    # 定位数据
        
        print(tag_div)
        print(type(tag_div))  # <class 'bs4.element.Tag'>
        
    --------------------------------------------------
    # Output:
    <a class="du" href="">总为浮云能避日，长安不见使人愁</a>
    <class 'bs4.element.Tag'>
    ```


#### 解析数据
    
| 方法        | 用                                       | 返值                      |
| :---------- |:----------------------------------------- |:-------------------------- |
| .text       | 返回标签下所有**直系和非直系标签**的所有本 | str                         |
| .get_text() | 返回标签下所有**直系和非直系标签**的所有本 | str                         |
| .string     | 返回标签下所有**直系标签**的所有本         | bs4.element.NavigableString |


##### 获取文本
1. `bs.tagName.text/string/get_text()`：获取标签之间的*所有文本**
    1. `text/get_text()`：可以获取标签下**直系和非直系**的所有文本
        ```python
        with open('./bs_test.html', 'r', encoding='utf-8') as f:
            bs = BeautifulSoup(f, 'lxml')       # 创建bs对象
            
            txt_li = bs.find('li').text
            
            print(txt_li)
            print(type(txt_li))    # <class 'str'>
            
        --------------------------------------------------
        # Output:
        清明时节雨纷纷，路上行人欲断魂。借问酒家何处有，牧童遥指杏花村。
        <class 'str'>
        ```
        ```python
        with open('./bs_test.html', 'r', encoding='utf-8') as f:
            bs = BeautifulSoup(f, 'lxml')       # 创建bs对象
            
            txt_li = bs.find('li').get_text()
            
            print(txt_li)
            print(type(txt_li))    # <class 'str'>
            
        --------------------------------------------------
        # Output:
        清明时节雨纷纷，路上行人欲断魂。借问酒家何处有，牧童遥指杏花村。
        <class 'str'>
        ```
    2. `string`：只能获取标签下**直系**的文本，没有返回 None
        ```python
        with open('./bs_test.html', 'r', encoding='utf-8') as f:
            bs = BeautifulSoup(f, 'lxml')       # 创建bs对象
            
            txt_li = bs.find('li').string
            
            print(txt_li)
            print(type(txt_li))    # <class 'bs4.element.NavigableString'>
            
        --------------------------------------------------
        # Output:
        清明时节雨纷纷，路上行人欲断魂。借问酒家何处有，牧童遥指杏花村。
        <class 'bs4.element.NavigableString'>
        ```

##### 获取属性
 1. `bs.tagName['attrName']`：获取标签中的**属性内容**
    ```python
    with open('./bs_test.html', 'r', encoding='utf-8') as f:
        bs = BeautifulSoup(f, 'lxml')       # 创建bs对象
        
        txt_href = bs.find('a')['href']
        
        print(txt_href)
        print(type(txt_href))    # <class 'str'>
        
    --------------------------------------------------
    # Output:
    https://www.song.com
    <class 'str'>
    ```

### 案例
> 从诗词名句网下载一整部《论语》

```python
import time
import requests
import os
from bs4 import BeautifulSoup

UA = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.89 Safari/537.36 Edg/84.0.522.44'
header = {'User-Agent': UA}

folder = '.\\爬虫\\论语'
if not os.path.exists(folder):
    os.mkdir(folder)


def req_catalog(url):
    """ 请求目录列表 """
    return requests.get(url=url, headers=header).text


def catalog_parse(res_text):
    """ 解析目录列表 """
    bs = BeautifulSoup(res_text, 'lxml')
    links = bs.select('.book-mulu > ul > li > a')
    catalog_list = [[x.string, x['href']] for x in links]

    return catalog_list


def download_content(catalog_list):
    """ 请求内容页 """
    url = 'https://www.shicimingju.com'
    length = len(catalog_list)

    for index, elem in enumerate(catalog_list):
        # 请求数据
        content_text = requests.get(url=url + elem[1], headers=header).text
        # 解析数据
        content = contents_parse(content_text)
        # 持久化存储
        filename = str(index + 1) + '-' + elem[0] + '.txt'
        filepath = os.path.join(folder, filename)
        with open(filepath, 'w', encoding='utf-8') as f:
            f.write(content)
            print("已下载：" + str(index + 1) + " / " + str(length), end="\r")
    return 1


def contents_parse(page_text):
    """ 解析内容 """
    bs = BeautifulSoup(page_text, 'lxml')
    contents = bs.select('.chapter_content > p')  # 取出所有p标签
    texts = [x.string for x in contents]          # 取出内容，过滤掉P标签
    content = ''
    for i in texts:
        content += str(i)
    return content


def main():
    url = "https://www.shicimingju.com/book/lunyu.html"

    catalog_list = list()
    try:
        catalog_text = req_catalog(url)
        catalog_list = catalog_parse(catalog_text)
        print("下载成功！") if download_content(catalog_list) else print("下载失败")
    except TimeoutError:
        print("TimeoutError\n")
        time.sleep(2)
        print("下载成功！") if download_content(catalog_list) else print("下载失败")
    except Exception:
        print("Exception\n")


if __name__ == '__main__':
    main()

```

## xpath爬取
最常用、通用性最强的，最便捷高效的一种解析方式。

### 解析步骤
1. 实例化一个etree对象，并且需要将被解析的页面源码数据加载到该对象中。
2. 通过调用etree对象中的xpath方法结合着xpath表达式实现标签的定位和内容的捕获
3. xpath定位到数据后返回的不是数据的内容，而已一个列表，里面放置了解析出来的**Element对象**


### 安装
\>_: pip install lxml

### 导入
> `from lxml import etree`

### 使用

- 创建etree对象并传入待解析对象
    1. 传入待解析对象为本地文件 `tree = etree.parse(filePath)`
    2. 传入待解析对象为网络请求 `tree = etree.HTML('page_text')`
- 定位数据：`tree.xpath(xpath表达式)`
    在XPath中有7种节点：元素、属性、文本、文档、命名空间、处理指令、注释。
    元素、属性、文本 为常用节点。
    ```html
    <html> 为文档节点
    <li>小米</li> 为元素节点
    class='blank' 为属性节点
    <!-- 这里是注释 --> 为注释节点
    ```

    | expression | description                         |
    | :--------- | :---------------------------------- |
    | nodeName   | 选择nodeName节点的所有子节点         |
    | /          | 从根节点或/前的节点开始，不跨层级匹配 |
    | //         | 从//前的节点开始，跨层级匹配         |
    | .          | 选择当前节点                        |
    | ..         | 选择当前节点的父节点                 |
    | @          | 匹配元素属性                        |
    | *          | 匹配所有节点                        |
    | @*         | 匹配节点所有属性                     |
    | []         | 按索引定位                          |

    1. `/`：表示从根节点开始定位
        > / 放在最前面的时候表示根节点，不是放在最前面的时候表示

        ```python
        from lxml import etree

        with open('./test.html', 'r', encoding='utf-8') as f:
            f_content = f.read()
            tree = etree.HTML(f_content)
            r = tree.xpath('/html/body/div/p')
            print(r)
            print(type(r))

        --------------------------------------------------
        xpath 在匹配的时候是贪婪的，示例中有两个 div 下都有 p，所以匹配到了5个
        # Output:
        [<Element p at 0x252e2de8780>,
         <Element p at 0x252e2de87c0>,
         <Element p at 0x252e2de8800>,
         <Element p at 0x252e2de8840>,
         <Element p at 0x252e2de8880>]
        <class 'list'>
        ```
    2. `//`：表示匹配多级
        > /a/b//c，就表示匹配 a 标签下的 b 标签下所有c标签

        ```python
        from lxml import etree

        with open('./test.html', 'r', encoding='utf-8') as f:
            f_content = f.read()
            tree = etree.HTML(f_content)
            r = tree.xpath('/html//a')        # 等价于 r = tree.xpath('//a')
            print(r)
            print(type(r))

        --------------------------------------------------
        html节点下总共有8个a标签，所以匹配到8个element对象
        # Output:
        [<Element a at 0x1e5f55e9680>,
         <Element a at 0x1e5f55e96c0>,
         <Element a at 0x1e5f55e9700>,
         <Element a at 0x1e5f55e9740>,
         <Element a at 0x1e5f55e9780>,
         <Element a at 0x1e5f55e9800>,
         <Element a at 0x1e5f55e9840>,
         <Element a at 0x1e5f55e9880>]
        <class 'list'>
        ```
    3. `@`：表示通过属性定位
        > `tag[@attrName="attrValue"]`
        > @后面加上属性名，比如class、id、href、src等等

        ```python
        from lxml import etree

        with open('./test.html', 'r', encoding='utf-8') as f:
            f_content = f.read()
            tree = etree.HTML(f_content)
            r = tree.xpath('/html//div[@class="song"]')
            print(r)
            print(type(r))

        --------------------------------------------------
        html节点下总共有8个a标签，所以匹配到8个element对象
        # Output:
        [<Element div at 0x24d64839640>]
        <class 'list'>
        ```
    4. `[]`：表示通过索引定位
        > `tag[index]`
        > 这里是索引是**从1开始的**

        ```python
        from lxml import etree

        with open('./test.html', 'r', encoding='utf-8') as f:
            f_content = f.read()
            tree = etree.HTML(f_content)

            print(tree.xpath('/html//div[@class="song"]/p[1]'))
            print(tree.xpath('/html//div[@class="song"]/p[1]/text()'))
            print(tree.xpath('/html//div[@class="song"]/p[2]'))
            print(tree.xpath('/html//div[@class="song"]/p[3]'))

        --------------------------------------------------
        这里的下标是从1开始的
        # Output:
        [<Element p at 0x1df3d4395c0>]
        ['李清照']
        [<Element p at 0x1df3d439580>]
        [<Element p at 0x1df3d439600>]
        <class 'list'>
        ```
    5. `/text()`：返回标签之间的文本，**取文本**
        > `tag/text()`：获取tag下**直系**的文本
        > `tag//text()`：获取tag下**直系和非直系**的文本

        ```python
        from lxml import etree

        with open('./test.html', 'r', encoding='utf-8') as f:
            f_content = f.read()
            tree = etree.HTML(f_content)
            r1 = tree.xpath('/html//div[@class="song"]/p[3]')
            r2 = tree.xpath('/html//div[@class="song"]/p[3]/text()')
            r3 = tree.xpath('/html//div[@class="song"]/p[3]/text()')[0]
            print(r1)
            print(r2)
            print(r3)

        --------------------------------------------------

        # Output:
        [<Element p at 0x1d4e4c29540>]
        ['苏轼']
        苏轼
        <class 'list'>
        ```
    5. `/@attrName`：返回标签的attrName属性的值
        > `tag/@attrName`：获取tag标签中的attrName属性的值

        ```python
        from lxml import etree

        with open('./test.html', 'r', encoding='utf-8') as f:
            f_content = f.read()
            tree = etree.HTML(f_content)
            r1 = tree.xpath('//div[@class="song"]/a/@href')
            print(r1)

            r2 = tree.xpath('//div[@class="song"]/img/@src')
            print(r2)

        --------------------------------------------------

        # Output:
        ['https://www.song.com', '']
        ['https://www.baidu.com/meinv.jpg']
        <class 'list'>
        ```
    5. `/@*`：返回标签的所有属性的值
        > `tag/@*`：获取tag标签中的所有属性的值

        ```python
        from lxml import etree

        with open('./test.html', 'r', encoding='utf-8') as f:
            f_content = f.read()
            tree = etree.HTML(f_content)
            r1 = tree.xpath('//div[@class="song"]/a[@target="_self"]/@*')
            print(r1)

        --------------------------------------------------

        # Output:
        ['https://www.song.com', '赵匡胤', '_self']
        <class 'list'>
        ```

### 案例
> 从彼岸图网下载4K图保存至本地

```python
import os
from lxml import etree
import requests

UA = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.89 Safari/537.36 Edg/84.0.522.44'
header = {'User-Agent': UA}

path = './爬虫/4k'
if not os.path.exists(path):
    os.mkdir(path)


def get_url():
    """ 获取所有图片地址 """
    url = 'http://pic.netbian.com/4kmeinv/'
    index_text = requests.get(url=url, headers=header).text

    tree = etree.HTML(index_text)
    a_list = tree.xpath('//div[@id="main"]/div[3]/ul/li/a')
    # response.encoding = 'utf-8'   # 处理中文乱码方式1，不一定有效

    img_list = list()
    for a in a_list:
        src: str = 'http://pic.netbian.com' + a.xpath('./@href')[0]
        title: str = a.xpath('./b/text()')[0] + '.jpg'
        title = title.encode('iso-8859-1').decode('gbk')  # 处理中文乱码方式2
        img_list.append((title, src))

    return img_list


def download_img(img_info):
    """ 下载图片 """

    img_content = requests.get(url=img_info[1], headers=header).content
    filepath = os.path.join(path, img_info[0])
    with open(filepath, 'wb') as f:
        f.write(img_content)


def main():
    img_list = get_url()
    for img_info in img_list:
        download_img(img_info)


if __name__ == '__main__':
    main()

```