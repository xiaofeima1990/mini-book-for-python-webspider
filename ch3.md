## 越过反爬虫的障碍：高级爬虫技巧
python利用urllib2打开网页，默认的方式是把自己作为“Python-urllib/x.y”(x和y是Python主版本和次版本号,例如Python-urllib/2.7)传给服务器，因为这种方式与通过浏览器访问服务器是不同的，一些存在反爬虫机制的网站会阻止程序持续登录。，这种情况下，为了持续进行网络爬虫我们需要进行一定程度的“伪装”，来绕过网站服务器的反爬虫机制，获得想要的数据。主要方式有：模仿浏览器设置表头，更换代理ip，cookie处理


#### 登录情况
需要登录的情况可以通过先用浏览器找到所发的post 请求，从网页中获取后填入到程序中进行模拟登陆。这里以verycd 网站为例

```python
import urllib
import urllib2

url = 'http://www.verycd.com/signin'
data={ 
       'username':username,
       'password':password,
       'continue':'http://www.verycd.com/',
       'login_submit':u'登录'.encode('utf-8'),
       'save_cookie':1,}

data = urllib.urlencode(data)
req = urllib2.Request(url, data, headers)  
response = urllib2.urlopen(req)  
the_page = response.read() 

```


#### 设置表头
浏览器确认自己身份是通过User-Agent头，当你创建了一个请求对象，你可以给他一个包含头数据的字典。下面的例子发送跟上面一样的内容，但把自身模拟成Internet Explorer。

```python
import urllib  
import urllib2  

url = 'http://www.verycd.com/signin'

user_agent = 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)'   
headers = { 'User-Agent' : user_agent,
            'Referer':'http://www.cnbeta.com/articles'
            }
data={ 
       'username':username,
       'password':password,
       'continue':'http://www.verycd.com/',
       'login_submit':u'登录'.encode('utf-8'),
       'save_cookie':1,} 

data = urllib.urlencode(data)  
req = urllib2.Request(url, data, headers)  
response = urllib2.urlopen(req)  
the_page = response.read() 
```
#### 代理服务器
```python
import urllib2
proxy_support = urllib2.ProxyHandler({'http':'http://XX.XX.XX.XX:XXXX'})
opener = urllib2.build_opener(proxy_support, urllib2.HTTPHandler)
urllib2.install_opener(opener)
content = urllib2.urlopen('http://XXXX').read()
```
#### cookie 处理

cookie 处理

```python
import urllib2, cookielib
cookie_support= urllib2.HTTPCookieProcessor(cookielib.CookieJar())
opener = urllib2.build_opener(cookie_support, urllib2.HTTPHandler)
urllib2.install_opener(opener)
content = urllib2.urlopen('http://XXXX').read()
```

如果想同时用代理和cookie，那就加入proxy_support然后operner改为

```python
opener = urllib2.build_opener(proxy_support, cookie_support, urllib2.HTTPHandler)
```

保存cookie到文件中
```python 
import cookielib
import urllib2

#设置保存cookie的文件，同级目录下的cookie.txt
filename = 'cookie.txt'
#声明一个MozillaCookieJar对象实例来保存cookie，之后写入文件
cookie = cookielib.MozillaCookieJar(filename)
#利用urllib2库的HTTPCookieProcessor对象来创建cookie处理器
handler = urllib2.HTTPCookieProcessor(cookie)
#通过handler来构建opener
opener = urllib2.build_opener(handler)
#创建一个请求，原理同urllib2的urlopen
response = opener.open("http://www.baidu.com")
#保存cookie到文件
cookie.save(ignore_discard=True, ignore_expires=True)
```

#### 设置连接失败自动重试

```python
    def get(self,req,retries=3):
        try:
            response = self.opener.open(req)
            data = response.read()
        except Exception , what:
            print what,req
            if retries>0:
                return self.get(req,retries-1)
            else:
                print 'GET Failed',req
                return ''
        return data
        
   


``` 
#### 超时设置 
对链接超时我们可以利用socket 进行超时设置（在后续利用其他程序包中selenium、scrapy等，内容更为丰富，里面自动含有超时设置等信息）
但Python2.6以后我们可以直接 在urlopen中利用timeout 进行设置

```python
import urllib2
import socket
socket.setdefaulttimeout(5) # 5 秒钟后超时
urllib2.socket.setdefaulttimeout(5) # 另一种方法

import urllib2  
response = urllib2.urlopen('http://www.google.com', timeout=10)  
```