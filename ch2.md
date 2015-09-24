## urlib,urlib2 ：python网络通信基本框架

### 相关参考
[urllib2 文档](https://docs.python.org/2/library/urllib2.html)
[web 网络基础]


### urllib2,urllib介绍
在Python中，我们使用urllib2这个组件来进行网络通信。 urllib2是Python的一个获取URLs(Uniform Resource Locators)的组件。它以urlopen函数的形式提供了一个非常简单的接口。下面即是一个最简单的例子
```python
import urllib2
response = urllib2.urlopen('http://www.baidu.com/')
html = response.read()
print html

```

urlopen从字面上即可看出这是一个打开链接的函数，主要负责网络通信的接受信息。无法进行更深入的交互性工作（get请求）。对于交互性响应，urllib2提供了一个Request对象来映射你提出的请求。通过调用urlopen并传入Request对象，将返回一个相关请求response对象，这个应答对象如同一个文件对象，所以你可以在Response中调用.read()。

```python
import urllib2  
req = urllib2.Request('http://www.baidu.com')  
response = urllib2.urlopen(req)  
the_page = response.read()  
print the_page
```

Request 允许你进行POST请求发送,发送表单、登录信息等。
```python

import urllib  
import urllib2  

url = 'http://www.someserver.com/register.cgi'  
  
values = {'name' : 'xxx',  
          'location' : 'beijing',  
          'language' : 'Python2' }  

data = urllib.urlencode(values) # 编码工作
req = urllib2.Request(url, data)  # 发送请求同时传data表单
response = urllib2.urlopen(req)  #接受反馈的信息
the_page = response.read()  #读取反馈的内容
```
当然，data数据可以全部转化成url链接进行发送，这时候就是get 方式请求了
```python 

import urllib2  
import urllib

data = {}

data['name'] = 'xxx'  
data['location'] = 'beijing'  
data['language'] = 'Python2'

url_values = urllib.urlencode(data)  
print url_values

name=Somebody+Here&language=Python&location=Northampton  
url = 'http://www.example.com/example.cgi'  
full_url = url + '?' + url_values

data = urllib2.open(full_url)  

```

接下来介绍的是两个urllib2常用的方法 ： info and geturl 

1. geturl()：这个返回获取的真实的URL，这个很有用，因为urlopen(或者opener对象使用的)或许会有重定向。获取的URL或许跟请求URL不同。
2. info() : 这个返回对象的字典对象，该字典描述了获取的页面情况。通常是服务器发送的特定头headers。目前是httplib.HTTPMessage 实例


```python

from urllib2 import Request, urlopen, URLError, HTTPError


old_url = 'http://www.renren.com/324392671'
req = Request(old_url)
response = urlopen(req)  
print 'Old url :' + old_url
print 'Real url :' + response.geturl()



```



### URL (http)异常处理
当urlopen不能够处理一个response时，产生urlError。不过通常的Python APIs异常如ValueError,TypeError等也会同时产生。HTTPError是urlError的子类，通常在特定HTTP URLs中产生。

#### URLError
通常，URLError在没有网络连接(没有路由到特定服务器)，或者服务器不存在的情况下产生。
这种情况下，异常同样会带有"reason"属性，它是一个tuple（可以理解为不可变的数组），
包含了一个错误号和一个错误信息。
```python 
import urllib2

req = urllib2.Request('http://www.baibai.com')
try: 
    urllib2.urlopen(req)
except urllib2.URLError, e:  
    print e.reason

```

#### HTTPError
服务器上每一个HTTP 应答对象response包含一个数字"状态码"。有时状态码指出服务器无法完成请求。默认的处理器会为你处理一部分这种应答。
例如:假如response是一个"重定向"，需要客户端从别的地址获取文档，urllib2将为你处理。其他不能处理的，urlopen会产生一个HTTPError。

典型的错误包含"404"(页面无法找到)，"403"(请求禁止)，和"401"(带验证请求)。HTTP状态码表示HTTP协议所返回的响应的状态。比如客户端向服务器发送请求，如果成功地获得请求的资源，则返回的状态码为200，表示响应成功。如果请求的资源不存在， 则通常返回404错误。 

HTTP状态码通常分为5种类型，分别以1～5五个数字开头，由3位整数组成：

* 200：请求成功      处理方式：获得响应的内容，进行处理 
* 201：请求完成，结果是创建了新资源。新创建资源的URI可在响应的实体中得到    处理方式：爬虫中不会遇到 
* 202：请求被接受，但处理尚未完成    处理方式：阻塞等待 
* 204：服务器端已经实现了请求，但是没有返回新的信 息。如果客户是用户代理，则无须为此更新自身的文档视图。    处理方式：丢弃
* 300：该状态码不被HTTP/1.0的应用程序直接使用， 只是作为3XX类型回应的默认解释。存在多个可用的被请求资源。    处理方式：若程序中能够处理，则进行进一步处理，如果程序中不能处理，则丢弃
* 301：请求到的资源都会分配一个永久的URL，这样就可以在将来通过该URL来访问此资源    处理方式：重定向到分配的URL
* 302：请求到的资源在一个不同的URL处临时保存     处理方式：重定向到临时的URL 
* 304 请求的资源未更新     处理方式：丢弃 
* 400 非法请求     处理方式：丢弃 
* 401 未授权     处理方式：丢弃 
* 403 禁止     处理方式：丢弃 
* 404 没有找到     处理方式：丢弃 
* 5XX 回应代码以“5”开头的状态码表示服务器端发现自己出现错误，不能继续执行请求    处理方式：丢弃


HTTPError实例产生后会有一个整型'code'属性，是服务器发送的相关错误号。
Error Codes错误码
因为默认的处理器处理了重定向(300以外号码)，并且100-299范围的号码指示成功，所以你只能看到400-599的错误号码。BaseHTTPServer.BaseHTTPRequestHandler.response 是一个很有用的应答号码字典，显示了HTTP协议使用的所有的应答号。当一个错误号产生后，服务器返回一个HTTP错误号，和一个错误页面。你可以使用HTTPError实例作为页面返回的应答对象response。
这表示和错误属性一样，它同样包含了read,geturl,和info方法。


```python

import urllib2
req = urllib2.Request('http://bbs.csdn.net/callmewhy')

try:
    urllib2.urlopen(req)
except urllib2.URLError, e:
    print e.code
    #print e.read()


```


异常处理不同error的方式,因为HTTPError是URLError的子类，如果URLError在前面它会捕捉到所有的URLError（包括HTTPError ），所以需要把HTTPError放到前面


```python
from urllib2 import Request, urlopen, URLError, HTTPError

req = Request('http://www.baidu.com')

try:

    response = urlopen(req)

except HTTPError, e:
    print ' the request can not obtained.'
    print 'Error code: ', e.code

except URLError, e:

    print 'failed to connect to the internet.'
    print 'Reason: ', e.reason

else:
    print 'No exception was raised.'
    # everything is fine

```


#### 下载文件操作

urllib 模块提供的 urlretrieve() 函数。urlretrieve() 方法直接将远程数据下载到本地。

Help on function urlretrieve in module urllib:
urlretrieve(url, filename=None, reporthook=None, data=None)

* 参数 finename 指定了保存本地路径（如果参数未指定，urllib会生成一个临时文件保存数据。）
* 参数 reporthook 是一个回调函数，当连接上服务器、以及相应的数据块传输完毕时会触发该回调，我们可以利用这个回调函数来显示当前的下载进度。
* 参数 data 指 post 到服务器的数据，该方法返回一个包含两个元素的(filename, headers)元组，filename 表示保存到本地的路径，header 表示服务器的响应头


下面一个例子可以简单演示如何下载文件

```python
#!/usr/bin/python
#encoding:utf-8
import urllib
import os
def Schedule(a,b,c):
    '''''
    a:已经下载的数据块
    b:数据块的大小
    c:远程文件的大小
   '''
    per = 100.0 * a * b / c
    if per > 100 :
        per = 100
    print '%.2f%%' % per
url = 'http://www.python.org/ftp/python/2.7.5/Python-2.7.5.tar.bz2'
#local = url.split('/')[-1]
local = os.path.join('/data/software','Python-2.7.5.tar.bz2')
urllib.urlretrieve(url,local,Schedule)
######output######
#0.00%
#0.07%
#0.13%
#0.20%
#....
#99.94%
#100.00%
```



**一个简单的例子**

# coding: utf-8
import urllib,urllib2
#help(urllib2)
url="http://www.baidu.com/"

html=urllib2.urlopen(url)
code=html.getcode() # 检查返回状态 200 表示返回成功 404 表示找不到页面
print code
gic = urllib.quote(url,':?=/') #转换编码
# 编码：urllib.quote(string[, safe])，除了三个符号“_.-”外，将所有符号编码，后面的参数safe是不编码的字符，
#使用的时候如果不设置的话，会将斜杠，冒号，等号，问号都给编码了。
print gic


#if code == 200:
#   print html.read()
#   print html.info()
#else:
#   print "webpage is problem"

#content = html.read().decode('gbk','ignore').encode('utf-8')
# print content
