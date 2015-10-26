## 反爬虫必杀技：模拟浏览器操作(1)

urllib2,urllib 模块可以帮助我们进行简单的爬虫设置，设置表头，设置代理等操作可以帮助我们绕过反爬虫机制进行持续爬虫。但是，很多时候强大的爬虫机制仍会阻止研究者以及程序员们的爬虫搜索。这时候我们就需要更为真实的模拟浏览器操作来帮助我们进行高级爬虫设置。幸运的是，python有着丰富的程序扩展包支持我们进行模拟浏览器操作。本节主要介绍三种常用的模拟浏览器拓展包，方便研究者以及程序员进行网络爬虫操作：

* mechanize
* Scrapy
* selenium

### 参考资料

http://blog.chinaunix.net/uid-26722078-id-3507409.html

http://dreamrunner.org/wiki/public_html/Python/Python%20Mechanize%20Cheat%20Sheet%20.html

http://stockrt.github.io/p/emulating-a-browser-in-python-with-mechanize/

https://pypi.python.org/pypi/mechanize/




### mechanize
mechanize是对urllib2的部分功能的替换，能够更好的模拟浏览器行为，在web访问控制方面做得更全面。小巧的mechanize具有着强大的功能，基本上使用urllib2的地方都可以利用mechanize进行替换，并且mechanize性能更好。下面就一般使用做一下介绍，希望能够帮助读者们快速熟悉mechanize并利用mechanize进行网络爬虫

#### 初始化并建立一个浏览器对象

```python
#!/usr/bin/env python
import sys,mechanize

#Browser 初始化浏览器
br = mechanize.Browser()

#options 基本设置：gzip压缩，redirect 链接等
br.set_handle_equiv(True)
br.set_handle_gzip(True)
br.set_handle_redirect(True)
br.set_handle_referer(True)
br.set_handle_robots(False)

#Follows refresh 0 but not hangs on refresh > 0
br.set_handle_refresh(mechanize._http.HTTPRefreshProcessor(), max_time=1)

#debugging 
br.set_debug_http(True)
br.set_debug_redirects(True)
br.set_debug_responses(True)

#User-Agent 你可以添加的更多
br.addheaders = [('User-agent', 'Mozilla/5.0 (X11; U; Linux i686; en-US; rv:1.9.0.1) Gecko/2008071615 Fedora/3.0.1-1.fc9 Firefox/3.0.1')]

```

#### 模拟浏览器行为

##### 打开网页

初始化完成后，就可以模拟浏览器行为进行浏览网页打开网页了
```python
r = br.open("wwww.baidu.com")
html = r.read()
print html
print br.response().read()
print br.title()
print r.info()
````

##### 查询

我们可以利用forms()方法查看网页上表单，找到特定表单后填写相应键值，可以利用post提交完成交互任务，我们分别利用谷歌查询、百度查询为例

```python
for f in br.forms():
    print f

br.select_form(nr=0)

# google 查询
br.form['q'] = 'python '
br.submit()
print br.response().read()

# baidu 查询
br.form['wd'] = 'pythonl'
br.submit()
print br.response().read()
```

##### 回退操作

```python
# Back
br.back()
print br.geturl()
```

##### 认证操作与cookie，代理

登录网站经常要填写认证信息，mechanize简化了urllib2认证的步骤，更为友好： 

```python
# 基本http认证
br.add_password('http://xxx.com', 'username', 'password')
br.open('http://xxx.com')


br.select_form(nr = 0)
br['email'] = username
br['password'] = password
resp = self.br.submit()

#通过导入cookielib模块，并设置浏览器cookie，可以在需要认证的网络行为之后不用重复认证登陆。
# 通过保存session cookie即可重新访问，Cookie Jar完成了该功能。
#!/usr/bin/env python
import mechanize, cookielib

br = mechanize.Browser()
cj = cookielib.LWPCookieJar()
br.set_cookiejar()


#Proxy 可以帮助我们设置http代理 绕过ip访问次数限制
br.set_proxies({"http":"proxy.com:8888"})
br.add_proxy_password("username", "password")

#Proxy and usrer/password
br.set_proxies({"http":"username:password@proxy.com:8888"})


```

##### 清除历史访问内存的办法
在使用mechanize的时候，运行不了多久我们就会发现内存占用率快速飙升，这是为什么呢？ 因为mechanize默认会保存模拟过的操作历史，导致占用的内存越来越大[注](http://stackoverflow.com/questions/2393299/how-do-i-disable-history-in-python-mechanize-module)。为何呢？ mechanize初始化Browser()的时候，如果你不给他传一个history对象作为参数，Browser()就会按照默认的方式（允许保存操作历史）来进行初始化。因此可以利用如下方式：

```python
class NoHistory(object):  
  def add(self, *a, **k): pass  
  def clear(self): pass  
  
b = mechanize.Browser(history=NoHistory())  

```

