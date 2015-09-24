## 反爬虫必杀技：模拟浏览器操作

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



### Selenium
selenium可以说是模拟浏览器操作的终极杀气，它可以直接操作浏览器内核，打开浏览器，甚至模拟鼠标操作进行页面点击、信息抓取。

#### 参考资料

http://selenium.googlecode.com/git/docs/api/py/index.html

https://selenium-python.readthedocs.org/

#### 安装与基本环境配置

要想进行爬虫，必要的组件一定要安装，因为selenium利用浏览器内核进行操作，所以下载相关浏览器内核是必须的。selenium默认的浏览器是firefox,其对firefox支持也是最好的。当然有时候我们需要利用IE,chrome等进行爬虫，这样我们就需要下载相关的浏览器内核插件，[点击这里chrome](http://chromedriver.storage.googleapis.com/index.html?path=2.19/)， [点击这里寻找相关资料](http://www.seleniumhq.org/download/)
首先按照selenium程序包:

```python
pip install selenium
```
##### 一个简单的例子：
下面就死一个很简单的selenium利用chrome浏览器搜索的例子，从中可以看出，selenium操作也很人性化。

```python
#coding:utf-8
import time
from selenium import webdriver # 相关函数的导入

driver = webdriver.Chrome('chromedriver')  # 控制chrome浏览器引号里面是driver的地址，一般放在所属路径下
# 相关的还有
# driver = webdriver.Firefox() #默认控制firefox浏览器
# driver = webdriver.Ie("IEDriverserver.exe")#控制IE浏览器

driver.get('http://www.baidu.com/');
time.sleep(5) # Let the user actually see something!
search_box = driver.find_element_by_name('q') # 通过名字寻找相关的元素
search_box.send_keys('ChromeDriver') # 根据上一个东东，（form）敲入一定的按键字母
search_box.submit()# 发送
time.sleep(5) # Let the user actually see something!
driver.quit() # 浏览器退出并关闭窗口的每一个相关的驱动程序

browser.close() #关闭当前窗口 ，用哪个看你的需求了
```

#### 页面导航与交互
打开页面仅仅是第一步，我们还需要很多与页面互动相关的操作和技巧来帮助我们搜集和定位信息。


##### 前进或者后退
想要回退到上一页面，或者前进到下一页面，我们可以利用下面简单命令实现

```python
driver.forward()
driver.back()
```

##### 搜寻与定位
假如一个html信息定义如下：

```html
<input type="text" name="passwd" id="passwd-id" />
```
我们可以通过如下几个方式来定位，详细的信息会在**元素定位**部分讨论

```python
element = driver.find_element_by_id("passwd-id")
element = driver.find_element_by_name("passwd")
element = driver.find_element_by_xpath("//input[@id='passwd-id']")
```

如果你找到一个html元素（表单），想要往里面输入字符，可以利用send_keys, 甚至可以模拟按键操作（比如登录页面要求输入完按回车）


值得注意的是，send_key命令相当于对元素里原有的文本信息进行附加(append),要想清楚元素里的信息，重新添加可以先利用.clear() 命令，再.send_key()

```python
element.send_keys("some text")
element.send_keys(" and some", Keys.ARROW_DOWN)

element.clear()
element.send_keys("some text")

```

##### 填充表格的高级操作



我们可以利用.send_keys()来填充文本信息。但对于选择或者下拉表格又该如何处理呢？利用selenium处理起来同样很简单：

```python

element = driver.find_element_by_xpath("//select[@name='name']")
all_options = element.find_elements_by_tag_name("option")
for option in all_options:
    print("Value is: %s" % option.get_attribute("value"))
    option.click()

```
上述命令帮助我们首先利用xpath方式寻找select 元素，在其中找到option,循环输出每个option的value信息并选择

selenium还直接有select命令，帮助我们点选相关元素

```python
from selenium.webdriver.support.ui import Select
select = Select(driver.find_element_by_name('name'))
select.select_by_index(index)
select.select_by_visible_text("text")
select.select_by_value(value)

```
有了select当然也有deselect选项,下面将id选项中的所有子选项都取消勾选

```python

select = Select(driver.find_element_by_id('id'))
select.deselect_all()


```
当我们需要知道一个页面表单中所有已经选择的信息名称时，可以利用下面方式完成。此外，我们可以利用.options返回所有可得的option选项

```python
select = Select(driver.find_element_by_xpath("xpath"))
all_selected_options = select.all_selected_options

options = select.options

driver.find_element_by_id("submit").click()
```

如何提交表单呢？selenium有两种方式帮助我们提交。第一种是根据标签或者其他属性定位到提交按钮，第二种是selenium自动寻找提交按键然后完成提交。

```python
driver.find_element_by_id("submit").click()

element.submit()

```


**复选框的处理**：若通过浏览器打个这个页面我们看到三个复选框和两个单选框。下面我们就来定位这三个复选框。

```python
from selenium import webdriver
import time
import os
dr = webdriver.Firefox()
file_path = 'file:///' + os.path.abspath('checkbox.html')
dr.get(file_path)
# 方法一
# 选择页面上所有的 input，然后从中过滤出所有的 checkbox 并勾选之
inputs = dr.find_elements_by_tag_name('input')
for input in inputs:
if input.get_attribute('type') == 'checkbox':
input.click()
time.sleep(2)
#方法二
# 选择所有的 checkbox 并全部勾上
checkboxes = dr.find_elements_by_css_selector('input[type=checkbox]')
for checkbox in checkboxes:
checkbox.click()
time.sleep(2)


dr.quit(

````



##### 窗口框架转换
现在web技术使得网页存在多个frame或者窗口，这对于爬虫的信息获取存在着很大障碍，而selenium向我们提供了在各个窗口(window)、框架(frame)下转换和返回的方法：

```python
<a href="somewhere.html" target="windowName">Click here to open a new window</a>
driver.switch_to_window("windowName")

driver.switch_to_frame("frameName")

driver.switch_to_frame("frameName.0.child") #would go to the frame named “child” of the first subframe of the frame called “frameName”. All frames are evaluated as if from *top*.

driver.switch_to_default_content()# 返回到上一级frame
```

示例学习：


frame.html
```html

<html>
<head>
<meta http-equiv="content-type" content="text/html;charset=utf-8" />
<title>frame</title>
<script type="text/javascript"
async=""src="http://ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js
"></script>
<link href="http://netdna.bootstrapcdn.com/twitter-bootstrap/2.3.2/css/bootstra
p-combined.min.css" rel="stylesheet" />
<script type="text/javascript">(document).ready(function(){});</script>
</head>
<body>
<div class="row-fluid">
<div class="span10 well">
<h3>frame</h3>
<iframe id="f1" src="inner.html" width="800",
height="600"></iframe>
</div>
</div>
</body>
<script
src="http://netdna.bootstrapcdn.com/twitter-bootstrap/2.3.2/js/bootstrap.
min.js"></script>
</html>

```

inner.html

```html

<html>
<head>
<meta http-equiv="content-type" content="text/html;charset=utf-8" />
<title>inner</title>
</head>
<body>
<div class="row-fluid">
<div class="span6 well">
<h3>inner</h3>
<iframe id="f2" src="http://www.baidu.com"
width="700"height="500"></iframe>
<a href="javascript:alert('watir-webdriver better than
selenium webdriver;')">click</a>
</div>
</div>
</body>
</html>


```
frame.html 中嵌套 inner.html 

下面通过 switch_to_frame() 方法来进行定位:
```python

#coding=utf-8
from selenium import webdriver
import time
import os
browser = webdriver.Firefox()
file_path = 'file:///' + os.path.abspath('frame.html')
browser.get(file_path)
browser.implicitly_wait(30)
#先找到到 ifrome1（id = f1）
browser.switch_to_frame("f1")
#再找到其下面的 ifrome2(id =f2)
browser.switch_to_frame("f2")
#下面就可以正常的操作元素了
browser.find_element_by_id("kw").send_keys("selenium")
browser.find_element_by_id("su").click()
time.sleep(3)
browser.quit()
```

类似于如下方式的定位：

```html

<html>
<head>
<meta http-equiv="content-type" content="text/html;charset=utf-8" />
<title>Level Locate</title>
<script type="text/javascript" async=""
src="http://ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js"></script>
<link href="http://netdna.bootstrapcdn.com/twitter-bootstrap/2.3.2/css/bootstrap-combined.min.css" rel="stylesheet" />
</head>
<body>
<h3>Level locate</h3>
<div class="span3">
<div class="well">
<div class="dropdown">
<a class="dropdown-toggle" data-toggle="dropdown"
href="#">Link1</a>
<ul class="dropdown-menu" role="menu"
aria-labelledby="dLabel" id="dropdown1" >
<li><a tabindex="-1" href="#">Action</a></li>
<li><a tabindex="-1" href="#">Another
action</a></li>
<li><a tabindex="-1" href="#">Something else
here</a></li>
<li class="divider"></li>
<li><a tabindex="-1" href="#">Separated
link</a></li>
</ul>
</div>
</div>
</div>
<div class="span3">
<div class="well">
<div class="dropdown">
<a class="dropdown-toggle" data-toggle="dropdown"
href="#">Link2</a>
<ul class="dropdown-menu" role="menu"
aria-labelledby="dLabel" >
<li><a tabindex="-1" href="#">Action</a></li>
<li><a tabindex="-1" href="#">Another
action</a></li>
<li><a tabindex="-1" href="#">Something else
here</a></li>
<li class="divider"></li>
<li><a tabindex="-1" href="#">Separated
link</a></li>
</ul>
</div>
</div>
</div>

</body>
<script src="http://netdna.bootstrapcdn.com/twitter-bootstrap/2.3.2/js/bootstrap.min.js"></script>
</html>

```

<div  align="center"> 
<img src="./pic/picweb.png" width = "600" height = "400" alt="定位层级" align=center />
</div>

**定位流程：** 先点击显示出1个下拉菜单，然后再定位到该下拉菜单所在的 ul，再定位 这个 ul 下的某个具体的 link


##### 跳出窗口的处理

有时候，我们点击一些东西，会跳出子窗口出来，比如确认，警示，或者内部选项。selenium 支持我们对跳出的窗口进行选择：

```python
alert = driver.switch_to_alert()
```


```python
element = driver.find_element_by_name("source")
target = driver.find_element_by_name("target")

from selenium.webdriver import ActionChains
action_chains = ActionChains(driver)
action_chains.drag_and_drop(element, target).perform()
```


##### cookie
selenium当然支持cookie的使用，下面命令帮助我们获得与设置cookie方便我们利用cookie进行页面访问

```python
# 打开目标页面
driver.get("http://www.example.com")

# 设置cookie
cookie = {'name': 'foo', 'value' : 'bar'}
driver.add_cookie(cookie)

# 得到当前页面的cookie
driver.get_cookies()

```



#### 元素定位
网络爬虫一个很重要的内容就是定位元素，selenium对元素定位支持的非常全面，我们甚至不需要beautifulsoup对页面进行规整解析。
selenium支持对单个元素定位的方法如下（返回满足条件的第一个元素）：

* find_element_by_id
* find_element_by_name
* find_element_by_xpath
* find_element_by_link_text
* find_element_by_partial_link_text
* find_element_by_tag_name
* find_element_by_class_name
* find_element_by_css_selector

多个元素的定位（返回满足条件的全部列表）：

* find_elements_by_name
* find_elements_by_xpath
* find_elements_by_link_text
* find_elements_by_partial_link_text
* find_elements_by_tag_name
* find_elements_by_class_name
* find_elements_by_css_selector


想要定位元素内部属性信息？ 利用**.get_attribute("name")** 


简单示例：

```python
from selenium.webdriver.common.by import By

driver.find_element(By.XPATH, '//button[text()="Some text"]')
driver.find_elements(By.XPATH, '//button')


```
**By.[]** 有如下： 

ID = "id"
XPATH = "xpath"
LINK_TEXT = "link text"
PARTIAL_LINK_TEXT = "partial link text"
NAME = "name"
TAG_NAME = "tag name"
CLASS_NAME = "class name"
CSS_SELECTOR = "css selector"

这里想要着重强调一下通过xpath搜索，xpath搜索可以基本上帮助你找到所有想要找到的信息。
假如html页面有下列信息：

```html
<html>
 <body>
  <form id="loginForm">
   <input name="username" type="text" />
   <input name="password" type="password" />
   <input name="continue" type="submit" value="Login" />
   <input name="continue" type="button" value="Clear" />
  </form>
</body>
<html>

```
我们想要寻找form的信息可以通过如下方式定位：

```python

login_form = driver.find_element_by_xpath("/html/body/form[1]")
login_form = driver.find_element_by_xpath("//form[1]")
login_form = driver.find_element_by_xpath("//form[@id='loginForm']")
#Absolute path (would break if the HTML was changed only slightly)
#First form element in the HTML
#The form element with attribute named id and the value loginForm
```
第一个方式是绝对寻址，但如果网页信息发生更改，就失效
第二个方式是寻找网页中第一个form元素
第三个是寻找存在id=loginForm的form元素

同样，我们可以利用此得到username元素

```python
username = driver.find_element_by_xpath("//form[input/@name='username']")
username = driver.find_element_by_xpath("//form[@id='loginForm']/input[1]")
username = driver.find_element_by_xpath("//input[@name='username']")
```

```python
clear_button = driver.find_element_by_xpath("//input[@name='continue'][@type='button']")
clear_button = driver.find_element_by_xpath("//form[@id='loginForm']/input[4]")
```

tag\[@[property]='name'\]\[@[property]='name'\]... 是利用tag中各种属性定位的方法

下面是一些学习xpath的链接，供大家查阅：

* [W3Schools XPath Tutorial](http://www.w3schools.com/Xpath/)
* [Xpath Tutorial](http://www.zvon.org/comp/r/tut-XPath_1.html)


#### 添加等待
因为网络速度，和网页上各种javascript和插件载入。我们有时候需要花费一段时间等待网页完全载入才可以。这时候我们就可以利用selenium提供的两种网络等待命令来进行了。

##### 精确等待

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

driver = webdriver.Firefox()
driver.get("http://somedomain/url_that_delays_loading")
try:
    element = WebDriverWait(driver, 10).until(
        EC.presence_of_element_located((By.ID, "myDynamicElement"))
    )
finally:
    driver.quit()

```
上述命令表示 driver等待10秒钟，如果10秒内 ID为"myDynamicElement" 没有出现，那么返回出错命令。
EC.[条件]有如下选择：

* title_is
* title_contains
* presence_of_element_located
* visibility_of_element_located
* visibility_of
* presence_of_all_elements_located
* text_to_be_present_in_element
* text_to_be_present_in_element_value
* frame_to_be_available_and_switch_to_it
* invisibility_of_element_located
* element_to_be_clickable - it is Displayed and Enabled.
* staleness_of
* element_to_be_selected
* element_located_to_be_selected
* element_selection_state_to_be
* element_located_selection_state_to_be
* alert_is_present

```python
from selenium.webdriver.support import expected_conditions as EC

wait = WebDriverWait(driver, 10)
element = wait.until(EC.element_to_be_clickable((By.ID,'someid')))

```
##### 隐含等待

一个隐含等待就是具体设置等待时间，然后看载入没有

```python
from selenium import webdriver

driver = webdriver.Firefox()
driver.implicitly_wait(10) # seconds
driver.get("http://somedomain/url_that_delays_loading")
myDynamicElement = driver.find_element_by_id("myDynamicElement")

```



#### 其他技巧

###### 最大化浏览器：

```python
from selenium import webdriver
import time
browser = webdriver.Firefox()
browser.get("http://www.baidu.com")
print "浏览器最大化"
browser.maximize_window() #将浏览器最大化显示
time.sleep(2)

```



##### 设置浏览器宽、高
```python
from selenium import webdriver
import time
browser = webdriver.Firefox()
browser.get("http://m.mail.10086.cn")
time.sleep(2)
#参数数字为像素点
print "设置浏览器宽480、高800显示"
browser.set_window_size(480, 800) time.sleep(3)
browser.quit()

```


#### 按键用法

##### 一般用法

```python

#coding=utf-8
from selenium import webdriver
from selenium.webdriver.common.keys import Keys #需要引入 keys 包
import os,time
driver = webdriver.Firefox()
driver.get("http://passport.kuaibo.com/login/?referrer=http%3A%2F%2Fwebcloud.ku
aibo.com%2F")
time.sleep(3)
driver.maximize_window() # 浏览器全屏显示
driver.find_element_by_id("user_name").clear()
driver.find_element_by_id("user_name").send_keys("fnngj")
#tab 的定位相相于清除了密码框的默认提示信息，等同上面的 clear()
driver.find_element_by_id("user_name").send_keys(Keys.TAB)
time.sleep(3)
driver.find_element_by_id("user_pwd").send_keys("123456")
#通过定位密码框，enter（回车）来代替登陆按钮
driver.find_element_by_id("user_pwd").send_keys(Keys.ENTER)

```

##### 键盘组合键用法

```python
#coding=utf-8
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
import os,time
driver = webdriver.Firefox()
driver.get("http://www.baidu.com")

#输入框输入内容
driver.find_element_by_id("kw").send_keys("selenium")
time.sleep(3)
#ctrl+a 全选输入框内容
driver.find_element_by_id("kw").send_keys(Keys.CONTROL,'a')
time.sleep(3)
#ctrl+x 剪切输入框内容
driver.find_element_by_id("kw").send_keys(Keys.CONTROL,'x')
time.sleep(3)
#输入框重新输入内容，搜索
driver.find_element_by_id("kw").send_keys(u"虫师 cnblogs")
driver.find_element_by_id("su").click()
time.sleep(3)
driver.quit()

```

#### 鼠标事件

ActionChains 类
 context_click() 右击
 double_click() 双击
 drag_and_drop() 拖动

普通的鼠标点击操作很简单，右击、双击、拖动也不复杂：
```python
driver.find_element_by_id(“xxx”).click()

#定位到要双击的元素
mouse =driver.find_element_by_xpath("xxx")
#对定位到的元素执行鼠标双击操作
ActionChains(driver).double_click(mouse).perform()

#定位元素的原位置
element = driver.find_element_by_name("source")
#定位元素要移动到的目标位置
target = driver.find_element_by_name("target")
#执行元素的移动操作
ActionChains(driver).drag_and_drop(element, target).perform()

```

##### 上传文件 

upload_file.html

```html
<html>
<head>
<meta http-equiv="content-type" content="text/html;charset=utf-8" />
<title>upload_file</title>
<script type="text/javascript"
async=""src="http://ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js
"></script>
<link
href="http://netdna.bootstrapcdn.com/twitter-bootstrap/2.3.2/css/bootstra
p-combined.min.css" rel="stylesheet" />
<script type="text/javascript">
</script>
</head>
<body>
<div class="row-fluid">
<div class="span6 well">
<h3>upload_file</h3>
<input type="file" name="file" />
</div>
</div>
</body>
<script src="http://netdna.bootstrapcdn.com/twitter-bootstrap/2.3.2/js/bootstrap.min.js"></script>
</html>

```
```python
#coding=utf-8
from selenium import webdriver
import os,time
driver = webdriver.Firefox()
#脚本要与 upload_file.html 同一目录
file_path = 'file:///' + os.path.abspath('upload_file.html')
driver.get(file_path)
#定位上传按钮，添加本地文件
driver.find_element_by_name("file").send_keys('D:\\selenium_use_case\upload_file.txt') ####最重要的例子
time.sleep(2)
driver.quit()
```

##### 处理下拉框

```html
<html>
<body>
<select id="ShippingMethod" onchange="updateShipping(options[selectedIndex]);" name="ShippingMethod">
<option value="12.51">UPS Next Day Air ==> $12.51</option>
<option value="11.61">UPS Next Day Air Saver ==> $11.61</option>
<option value="10.69">UPS 3 Day Select ==> $10.69</option>
<option value="9.03">UPS 2nd Day Air ==> $9.03</option>
<option value="8.34">UPS Ground ==> $8.34</option>
<option value="9.25">USPS Priority Mail Insured ==> $9.25</option>
<option value="7.45">USPS Priority Mail ==> $7.45</option>
<option value="3.20" selected="">USPS First Class ==> $3.20</option>
</select>
</body>
</html>

```

<div  align="center"> 
<img src="./pic/picweb2.png" width = "600" height = "400" alt="定位层级" align=center />
</div>


```python

#-*-coding=utf-8
from selenium import webdriver
import os,time
driver= webdriver.Firefox()

file_path = 'file:///' + os.path.abspath('drop_down.html')
driver.get(file_path)
time.sleep(2)
#先定位到下拉框
m=driver.find_element_by_id("ShippingMethod")
#再点击下拉框下的选项
m.find_element_by_xpath("//option[@value='10.69']").click() ##点击操作
time.sleep(3)
driver.quit()

```

#### 调用JS 

有时候，页面会通过js执行和隐藏各种信息，那么我们就需要对js进行操作，主要涉及的语言是driver.execute_script(''). 下面的例子表面如何调用页面中的js来隐藏按钮

```html
<html>
<head>
<meta http-equiv="content-type" content="text/html;charset=utf-8" />
<title>js</title>
<script type="text/javascript" async="" src="http://ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js"></script>
<link href="http://netdna.bootstrapcdn.com/twitter-bootstrap/2.3.2/css/bootstrap-combined.min.css" rel="stylesheet" />
<script type="text/javascript"> $(document).ready(function(){$('#tooltip').tooltip({"placement": "right"});});</script>
</head>
<body>
<h3>js</h3>
<div class="row-fluid">
<div class="span6 well">
<a id="tooltip" href="#" data-toggle="tooltip" title="
selenium-webdriver(python)">hover to see tooltip</a>
<a class="btn">Button</a>

```
下面是执行代码

```python

#coding=utf-8
from selenium import webdriver
import time,os
driver = webdriver.Firefox()
file_path = 'file:///' + os.path.abspath('js.html')
driver.get(file_path)

#######通过 JS 隐藏选中的元素##########第一种方法：
driver.execute_script('$("#tooltip").fadeOut();')
time.sleep(5)

```



#### 控制滚动条

有时候我们需要控制页面滚动条上的滚动条，但滚动条并非页面上的元素，这个时候就需要借助 js 是来进行操作。一般用到操作滚动条的会两个场景：

* 注册时的法律条文需要阅读，判断用户是否阅读的标准是：滚动条是否拉到最下方。
* 要操作的页面元素不在吸视范围，无法进行操作，需要拖动滚动条


用于标识滚动条位置的代码：如果滚动条在最上方的话，scrollTop=0 ，那么要想使用滚动条在最可下方，可以 scrollTop=100000，这样就可以使滚动条在最下方。

```python
<body onload= "document.body.scrollTop=0 ">
<body onload= "document.body.scrollTop=100000 ">
```
```python
#coding=utf-8
from selenium import webdriver
import time

#访问百度
driver=webdriver.Firefox()
driver.get("http://www.baidu.com")
#搜索
driver.find_element_by_id("kw").send_keys("selenium")
driver.find_element_by_id("su").click()
time.sleep(3)
#将页面滚动条拖到底部
js="var q=document.documentElement.scrollTop=10000"driver.execute_script(js)
time.sleep(3)
#将滚动条移动到页面的顶部
js="var q=document.documentElement.scrollTop=0"driver.execute_script(js)
time.sleep(3)
driver.quit()

```



