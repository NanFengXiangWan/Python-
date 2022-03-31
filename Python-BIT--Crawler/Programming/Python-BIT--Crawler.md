# 1.网络爬虫之规则

### 单元1：Requests库入门

#### Requests库的get()方法

1. `r = requests.get(url)`

    1. ![image.png](assets/image-20220327140115-m9u25wx.png)
2. 完整使用方法：`r = requests.get(url,params=None,**kwargs)`

    1. url：拟获取页面的url链接
    2. params：url中的额外参数，字典或字节流格式（可选）
    3. **kwargs：12个控制访问的参数（可选）
3. Response对象

    1. ```python
        >>> import requests
        >>> r = requests.get("https://www.baidu.com") 
        >>> print(r.status_code)  
        200
        >>> type(r) 
        <class 'requests.models.Response'>
        >>> r.headers
        {'Cache-Control': 'private, no-cache, no-store, proxy-revalidate, no-transform', 'Connection': 'keep-a
        live', 'Content-Encoding': 'gzip', 'Content-Type': 'text/html', 'Date': 'Sun, 27 Mar 2022 06:07:57 GMT
        ', 'Last-Modified': 'Mon, 23 Jan 2017 13:24:33 GMT', 'Pragma': 'no-cache', 'Server': 'bfe/1.0.8.18', '
        Set-Cookie': 'BDORZ=27315; max-age=86400; domain=.baidu.com; path=/', 'Transfer-Encoding': 'chunked'}
        ```
    2. Response对象：包含服务器返回的所有信息，也包含请求的Request信息。
    3. `r.headers` OR `r.request.headers` 两者的区别在于一个是直接访问Response对象，一个针对Request对象
4. Response对象属性

    1. |属性|说明|
        | :---------------------: | :--------------------------------------------------: |
        |r.status_code|HTTP请求的返回状态，200表示连接成功，404表示失败|
        |r.text|HTTP响应内容的字符串形式，即url对应的页面内容|
        |r.*encoding*<br />|从HTTP header中猜测的响应内容编码方式|
        |r.apparent_encoding<br />|从内容中分析出的响应内容编码方式（备选编码方式）<br />|
        |r.content|HTTP响应内容的二进制形式（图片）|
    2. r.encoding：查看网站中的header字段，如不存在charset字段，则编码为ISO­8859­1，备选编码则通过分析网站内容来判断编码

#### 爬取网页的通用代码框架

1. 理解Requests库的异常

    1. |异常|说明|
        | :-------------------------: | :-------------------------------------------: |
        |requests.ConnectionError|网络连接错误异常，如DNS查询失败、拒绝连接等|
        |requests.HTTPError|HTTP错误异常|
        |requests.URLRequired|URL缺失异常|
        |requests.TooManyRedirects|超过最大重定向次数，产生重定向异常|
        |requests.ConnectTimeout|连接远程服务器超时异常|
        |requests.Timeout|请求URL超时，产生超时异常|
        |r.raise_for_status()<br />|如果不是200，产生异常 requests.HTTPError<br />|
2. 爬取网页的通用代码框架

    1. ```python
        import requests


        def gethtmltext(url):
            try:
                r = requests.get(url, timeout=30)
                r.raise_for_status()
                r.encoding = r.apparent_encoding
                return r.text
            except:
                return "产生异常"


        if __name__ == '__main__':
            url = "http://www.baidu.com"
            print(gethtmltext(url))
        ```

#### HTTP协议及Requests库方法

* <u>HTTP协议（超文本传输协议）</u>

  * HTTP是一个基于“请求与响应”模式的、无状态的应用层协议
  * HTTP协议采用URL作为定位网络资源的标识
  * URL格式：http://host[:port][path] 

    * host：合法的Internet主机域名或IP地址
    * port：端口号，缺省端口为80
    * path：请求资源的路径
* <u>HTTP URL的理解</u>

  * URL是通过HTTP协议存取资源的Internet路径，一个URL对应一个数据资源
* <u>HTTP协议对资源的操作</u>

  * |方法|说明|
    | :-------: | :---------------------------------------------------------: |
    |GET|请求获取URL位置的资源|
    |HEAD|请求获取URL位置资源的响应消息报告，即获得该资源的头部信息|
    |POST<br />|请求向URL位置的资源后附加新的数据（表单）|
    |PUT|请求向URL位置存储一个资源，覆盖原URL位置的资源|
    |PATCH<br />|请求局部更新URL位置的资源，即改变该处资源的部分内容|
    |DELETE|请求删除URL位置存储的资源|
    |||
  * ![image.png](assets/image-20220327150504-9l2hide.png "不同方法对资源的操作")
* <u>Patch与Put的区别</u>

  * Patch：采用patch，仅向URL提交局部更新请求（节省网络带宽）
  * Put：采用Put，会覆盖之前的资源，所以你必须将全部所需字段提交到URL对应的位置上，未提交的字段会被删除，只保留你提交的部分，有点类似覆盖上传
* <u>Requests库的head使用方法</u>

  * ```python
    import requests

    r = requests.head("https://www.baidu.com")
    print(r.headers)
    ```
* <u>Requests库的post使用方法</u>

  * 向URL POST 一个字典，自动编码为form（表单）
  * 向URL POST 一个字符串，自动编码为data（数据）
  * POST方法根据用户提交内容的不同，在服务器上会做数据的相关整理
  * ```python
    #POST 字典会自动存入form表单中
    import requests

    payload = {'key1': 'value1', 'key2': 'value2'}
    r = requests.post('http://httpbin.org/post', data= payload)
    print(r.text)

    #POST 字符串会自动存入data数据中
    import requests

    # payload = {'key1': 'value1', 'key2': 'value2'}
    r = requests.post('http://httpbin.org/post', data='ABC')
    print(r.text)
    ```
* <u>Requests库的put使用方法</u>

  * PUT方法与POST方法类似，只不过它能够将原有的数据覆盖掉
  * ```python
    import requests

    payload = {'key1': 'value1', 'key2': 'value2'}
    r = requests.put('http://httpbin.org/put', data=payload)
    print(r.text)
    ```

#### Requests库主要方法解析

* `requests.request(method, url, **kwargs)`
* <u>method（请求方式）</u>

  * 对应GET/ HEAD/ POST/ PUT/ PATCH/ delete/ OPTIONS等7种方法
* <u>url</u>

  * 拟获取页面的url链接
* <u>**kwargs</u>
* 控制访问的参数，共13个（可选项）

  * <u>params：字典或字节序列，作为参数增加到url中</u>

    * ```python
      import requests

      kv = {'key1': 'value1', 'key2': 'value2'}
      r = requests.request('GET', 'https://www.nfxwblog.com/', params=kv)
      print(r.url)

      # https://www.nfxwblog.com/?key1=value1&key2=value2
      ```
  * <u>data：字典、字节序列或文件对象，作为Request的内容</u>（向服务器提供或提交资源时使用）

    * ```python
      import requests

      kv = {'key1': 'value1', 'key2': 'value2'}
      r = requests.request('POST', 'https://httpbin.org/post', data=kv)
      print(r.text)
      body = 'neirong'
      r = requests.request('POST', 'https://httpbin.org/post', data=body)
      print(r.text)
      ```
  * <u>json：JSON格式的数据，作为Request的内容</u>

    * ```python
      import requests

      kv = {'key1': 'value1'}
      r = requests.request('POST', 'https://httpbin.org/json', json=kv)
      print(r.text)
      ```
  * <u>headers：字典，HTTP定制头</u>

    * ```python
      import requests

      hd = {"user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit"}
      r = requests.request('GET', 'https://httpbin.org/user-agent', headers=hd)
      print(r.text)
      ```
  * <u>cookies：字典或CookieJar, Requests中的cookie </u>

    * ```python
      import requests

      cks = {"cookies": "Nan-Feng-NB"}
      r = requests.request('GET', 'https://httpbin.org/cookies', cookies=cks)
      print(r.text)
      ```
  * <u>auth：元组，支持HTTP认证功能</u>
  * <u>files：字典类型，传输文件</u>
  * <u>timeout：设定超时时间，秒为单位</u>（如果在timeout时间内没有返回请求内容，产生一个timeout异常）

    * ```python
      import requests

      r = requests.request('GET', 'https://www.nfxwblog.com', timeout=10)
      print(r.text)
      ```
  * <u>proxies：字典类型，设定访问代理服务器，可以增加登录认证</u>

    * ```python
      import requests

      # 在增加的http代理中可以增加用户名和密码的设置，在访问百度时使用的IP地址是
      # 代理服务器的IP地址，隐藏用户信息，防止爬虫的逆追踪
      pxs = {'http': 'http://user:pass@10.10.10.1:1234', 'https': 'http://10.10.10.1:4321'}
      r = requests.request('GET', 'https://www.baidu.com', proxies=pxs)
      ```
  * <u>allow_redirects：True/Flase，默认为True，重定向开关</u>
  * <u>stream：True/False，默认为True，获取内容后需不需要立即下载开关</u>
  * <u>verify：True/False，默认为True，认证SSL证书开关</u>
  * <u>cert：本地SSL证书路径</u>

* requests.get(url,params=None,**kwargs) 

  * **kwarg：12个控制访问参数
* requests.head(url, **kwarg) 

  * **kwarg：13个控制访问参数
* requests.post(url,  data=None, json=None, **kwargs) 

  * **kwarg：11个控制访问参数
* requests.put(url, data=None, **kwargs)

  * **kwarg：12个控制访问参数
* requests.patch(url, data=None, **kwargs) 

  * **kwarg：12个控制访问参数
* requests.delete(url, **kwargs) 

  * **kwarg：13个控制访问参数

### 单元2：网络爬虫的“盗亦有道”

* Robots协议（网络爬虫排除标准）

  * 作用：网站告知网络爬虫哪些页面可以抓取，哪些不行
  * 形式：在网站根目录下的robots.txt文件
  * 案例：百度（https://www.baidu.com/robots.txt）
* Robots协议的使用

  * 网络爬虫：自动或人工识别robots.txt，再进行内容爬取
  * 约束性：Robots协议是建议但非约束性，网络爬虫可以不遵守，但存在法律风险

### 单元3：Requests库网络爬虫实战（5个实例）

#### 实例1：京东\苏宁易购商品页面的爬取

```python
import requests
url = "https://item.jd.com/2967929.html"
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) "
                  "Chrome/80.0.3987.163 Safari/537.36",
}
r = requests.get(url, headers=headers)  # 因为京东啊，亚马逊都有反爬，所以要加入头部信息
r.raise_for_status()
r.encoding = r.apparent_encoding
print(r.text[:1001])
```

```python
import requests


def crawler_snyg(target):
    try:
        r = requests.get(url=target)
        r.raise_for_status()
        r.encoding = r.apparent_encoding
        return r.text[:1000]
    except:
        print("爬取失败")


if __name__ == '__main__':
    url = "https://product.suning.com/0000000000/000000011562764431.html?src=pindao_bingxi_44213315244_prod02&safp" \
          "=d488778a.bingxi.44213315244.2&safc=prd.0.0&safpn=10003.00002&utm_source=hao123&utm_midium=mingzhan" \
          "&utm_content=&utm_campaign= "
    print(crawler_snyg(url))
```

#### 实例 2 ：亚马逊商品页面的爬取

```python
import requests


def amazon(target, hed):
    try:
        r = requests.get(url=target, headers=hed)
        r.raise_for_status()
        r.encoding = r.apparent_encoding
        return r.text
    except:
        print("爬取失败")


if __name__ == '__main__':
    url = "https://www.amazon.cn/dp/B00B1Z65LQ/"
    kv = {"user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit"}
    print(amazon(url, kv))
```

#### 实例3：百度360搜索关键字提交

* 搜索引擎关键字提交接口
* 百度搜索关键字接口

  * [https://www.baidu.com/s?wd=keyword](https://www.baidu.com/s?wd=keyword)
* 360搜索关键字接口

  * [https://www.so.com/s?q=keyword](https://www.so.com/s?q=keyword)

```python
import requests


def baidu_so(value):
    try:
        kv = {'wd': value}
        r = requests.get("https://www.baidu.com/s", params=kv)
        print(f"状态码：{r.status_code}")
        print(f"当前URL：{r.url}")
        print(f"返回长度：{len(r.text)}")
    except:
        print("爬取失败")


if __name__ == '__main__':
    values = input("请输入要搜索的内容： ")
    print(baidu_so(values))
```

#### 实例4：网络图片的爬取和储存

```python
import requests


def dw_img(path, img_url):
    try:
        r = requests.get(img_url)
        print(f"响应状态码：{r.status_code}")
        r.raise_for_status()
        with open(path, 'wb') as f:  # 'wb'以二进制打开文件只用于写入，适用于非文本文件如有图片等
            f.write(r.content)  # 返回的二进制写入文件中
    except:
        print("下载失败")


if __name__ == '__main__':
    path = "E:/ColleCtion/Code_Projects/Python_Projects/" \
           "Python_Projects_1/test.jpg"
    url = "https://img.nfxwblog.com//blog/bolo741.jpg"
    dw_img(path, url)
```

```python
import requests
import os

url = "http://img0.dili360.com/ga/M00/48/F7/wKgBy1llvmCAAQOVADC36j6n9bw622.tub.jpg" 
root = "E://图片//爬虫//"
path = root + url.split('/')[-1]
try:
    if not os.path.exists(root):
        os.mkdir(root)
    if not os.path.exists(path):
        r = requests.get(url)
        r.raise_for_status()
        with open(path, 'wb') as f:
            f.write(r.content)      # r.content表示返回内容的二进制形式， f.close()           # 图片是以二进制形式存储的
            print("文件保存成功")
    else:
        print("文件已存在")
except:
    print("爬取失败")
```

#### 实例5：IP地址归属地自动查询

```python
import requests

'''
这里采用公共API来实现
接口地址： https://api.iyk0.com/ip
返回格式： TXT
请求方式： GET
请求示例： https://api.iyk0.com/ip/?ip=220.181.38.148
'''


def search_ip(ip):
    try:
        kv = {'ip': ip}
        url = "https://api.iyk0.com/ip/"
        r = requests.get(url, params=kv)
        print(f"状态码：{r.status_code}")
        r.raise_for_status()
        print(r.text)
    except:
        print("查询失败")


if __name__ == '__main__':
    ip = input("输入需要查询的IP地址： ")
    search_ip(ip)
```

# 2.网络爬虫之提取

### 单元4：Beautiful Soup库入门

#### Beautiful Soup库的安装

* Beautiful Soup

  * bs4 全名 BeautifulSoup，编写 Python爬虫常用库之一，主要用来解析HTML标签
  * 解析HTML页面信息标记与提取方法
  * ```python
    from bs4 import BeautifulSoup  # BeautifulSoup是一个类

    soup = BeautifulSoup("<html>A Html Text</html>", "html.parser") 
    ```

    * 两个参数：第一个参数是要解析的html文本，第二个参数是使用哪种解析器，对于HTML来讲就是html.parser，这个是bs4自带的解析器。
    * 如果一段HTML或XML文档格式不正确的话，那么在不同的解析器中返回的结果可能是不一样的。

#### Beautiful Soup库的基本元素

* Beautiful Soup库简介

  * Beautiful Soup库的理解：Beautiful Soup库是解析、遍历、维护“标签树”的功能库
* Beautiful Soup库引用

  * Beautiful Soup库，也叫beautiful soup4或bs4
  * `from bs4 import BeautifulSoup`

    * （说明从bs4库中引入了一个BeautifulSoup类型）或 import bs4（对Beautiful Soup库里的基本变量进行判断时）
* Beautiful Soup类

  * ```python
    from bs4 import BeautifulSoup  

    soup = BeautifulSoup('<p>data</p>', 'html.parser')# 使用bs4内置的解析器
    soup2 = BeautifulSoup(open("D://demo.html"), "html.parser") # 打开一个html文件
    ```
  * 简单来说Beautiful Soup库中的Soup变量对应一个HTML/XML文档的全部内容
* Beautiful Soup库解析器

  * |解析器|使用方法|条件|
    | :----------------: | :--------------------------------: | :--------------------: |
    |bs4的HTML解析器|BeautifulSoup(mk, 'html.parser')|安装bs4库|
    |lxml的HTML解析器|BeautifulSoup(mk, 'lxml')|pip install lxml|
    |lxml的XML解析器|BeautifulSoup(mk, 'xml')<br />|pip install lxml|
    |html5lib的解析器|BeautifulSoup(mk, 'html5lib')|pip install html5lib|
* Beautiful Soup基本元素

  * |基本元素|说明|
    | -----------------| ------------------------------------------------------------|
    |Tag|标签，最基本的信息组织单元，分别用<>和</>标明开头和结尾|
    |Name|标签的名字，<p>...</p>的名字是'p'，格式：<tag>.name|
    |Attributes|标签的属性，字典形式组织，格式：<tag>.attrs|
    |NavigableString|标签内非属性字符串，<>...</>中字符串，格式：<tag>.string<br />|
    |Comment|标签内字符串的注释部分，一种特殊的Comment类型|
* Tag 标签Name获取

  * ```python
    from bs4 import BeautifulSoup
    import requests

    demo = requests.get("https://python123.io/ws/demo.html")
    demo = demo.text
    soup = BeautifulSoup(demo, "html.parser")
    print(soup.title)  # 获取title标签
    print(soup.a)  # 获取页面第一个标签a里的内容
    print(soup.a.name)  # 查看标签a的名字
    print(soup.a.parent.name)  # 查看标签a的父级标签
    print(soup.a.parent.parent.name)  # 再查看p标签的父级标签
    ```
* Tag标签attrs（属性）获取

  * ```python
    from bs4 import BeautifulSoup
    import requests

    demo = requests.get("https://python123.io/ws/demo.html")
    demo = demo.text
    soup = BeautifulSoup(demo, "html.parser")
    tag = soup.a  # a 标签里的所有信息
    print(tag.attrs)  # 获取标签属性 返回的值是字典型式
    print(tag.attrs['class'])  # 获标签属性值里的内容
    print(tag.attrs['href'])
    print(type(tag.attrs['href'])  # 通过键访问返回的值是字符串类型
    ```
* Tag的NavigableString（字符串类型，表示尖括号标签之间的字符串）

  * ```python
    from bs4 import BeautifulSoup
    import requests

    demo = requests.get("https://python123.io/ws/demo.html")
    demo = demo.text
    soup = BeautifulSoup(demo, "html.parser")

    print(soup.a.string)  # 查看a标签的字符串信息
    print(soup.p.string)  # 查看p标签的字符串信息
    print(type(soup.p.string))  # 查看NavigableString类型表达
    ```
* 总结

  * ![image.png](assets/image-20220329200635-96c9602.png "标签的处理方法")

#### 基于bs4库的HTML内容遍历方法

* 基于bs4库的HTML内容遍历方法

  * 下行遍历（在根节点向叶子节点遍历）
  * 上行遍历（从叶子节点向根节点遍历）
  * 平级遍历（在平级节点之间互相遍历）
* 标签树的下行遍历

  * |属性|说明|
    | :------------: | :-------------------------------------------------------: |
    |.contents|子节点的列表，将<tag>所有儿子节点存入列表|
    |.children|子节点的迭代类型，与.contents类似，用于循环遍历儿子结点|
    |.descendants|子孙节点的迭代类型，包含所有子孙节点，用于循环遍历|
  * ```python
    from bs4 import BeautifulSoup
    import requests

    demo = requests.get("https://python123.io/ws/demo.html")
    demo = demo.text
    soup = BeautifulSoup(demo, "html.parser")

    print(soup.head)  # 头结点
    print(soup.head.contents)  # 以列表的形式返回head结点的所有儿子结点

    print(soup.body.contents)  # 以列表的形式返回body结点的所有儿子结点
    print(len(soup.body.contents))  # 查看获得的儿子结点列表的数量
    print(soup.body.contents[1])  # 返回的列表信息可以使用索引访问
    ```
  * ```python
    from bs4 import BeautifulSoup
    import requests

    demo = requests.get("https://python123.io/ws/demo.html")
    demo = demo.text
    soup = BeautifulSoup(demo, "html.parser")

    # 遍历儿子结点
    for chile in soup.body.children:
        print(chile)

    # 遍历子孙结点
    for chile in soup.body.children:
        print(chile)
    ```
* 标签树的上行遍历

  * |属性|说明|
    | ----------| ------------------------------------------|
    |.parent|节点的父亲标签|
    |.parents|节点先辈标签的迭代类型，用于遍历先辈节点|
  * ```python
    from bs4 import BeautifulSoup
    import requests

    demo = requests.get("https://python123.io/ws/demo.html")
    demo = demo.text
    soup = BeautifulSoup(demo, "html.parser")

    # 标签数的上行遍历
    for parent in soup.a.parents:  # 从a标签开始向上遍历
        if parent is None:  # 因为sopu本身也是一个标签None，当遍历到它时就不能打印出它的信息，这里我用pass代替
            pass
        else:
            print(parent.name)  # 打印出所有先辈的名字
    ```
* 标签数的平行遍历

  * ![300_08_2820220330-204439.png](assets/300_08_2820220330-204439-20220330204452-t5watvv.png "标签树的平行遍历条件")
  * ```python
    from bs4 import BeautifulSoup
    import requests

    demo = requests.get("https://python123.io/ws/demo.html")
    demo = demo.text
    soup = BeautifulSoup(demo, "html.parser")

    # 标签树的平行遍历遍历
    print(soup.a.next_sibling.next_sibling)  # 获取a标签的下一个标签的在下一个标签为另一个a标签

    print(soup.a.pervious_sibling)  # 获取a标签的前一个平行节点

    for sibling in soup.a.next_sibling:  # 遍历后续节点
        print(sibling)

    for sibling in soup.a.previous_siblings:  # 遍历前续节点
        print(sibling)
    ```
* ![300_12_0620220330-210408.png](assets/300_12_0620220330-210408-20220330210539-cdgk1fg.png "标签树上下平行遍历的方法")
