## beautiful soup 网络文本解析的利器
在传统打开网页获取网页信息后，我们随后遇到的问题便是如何提取我们需要的关键信息。例如我们想要从人人或者facebook上找到我们的好友信息，但同时网站上充斥各种分享、广告、应用外挂等。如何精确定位与解析html文本是我们高效处理数据抓取的关键。这里我们介绍一个强大的文本分析程序包beautifulsoup。 Beautiful Soup 是用Python写的一个HTML/XML的解析器，它可以很好的处理不规范标记并生成剖析树(parse tree)。 它提供简单又常用的导航（navigating），搜索以及修改剖析树的操作。它可以大大节省你的编程时间。

官方说明如下：
Beautiful Soup提供一些简单的、python式的函数用来处理导航、搜索、修改分析树等功能。它是一个工具箱，通过解析文档为用户提供需要抓取的数据，因为简单，所以不需要多少代码就可以写出一个完整的应用程序。

* Beautiful Soup自动将输入文档转换为Unicode编码，输出文档转换为utf-8编码。你不需要考虑编码方式，除非文档没有指定一个编码方式，这时， Beautiful Soup就不能自动识别编码方式了。然后，你仅仅需要说明一下原始编码方式就可以了。
* Beautiful Soup已成为和lxml、html6lib一样出色的python解释器，为用户灵活地提供不同的解析策略或强劲的速度。

优势特点：

* Beautiful Soup 相比其他的html解析有个非常重要的优势。html会被拆解为对象处理。全篇转化为字典和数组。
* 相比正则解析的爬虫，省略了学习正则的高成本。
* 相比xpath爬虫的解析，同样节约学习时间成本。虽然xpath已经简单点了。（爬虫框架Scrapy就是使用xpath）

### 参考资料

http://www.crummy.com/software/BeautifulSoup/bs4/doc/

http://cuiqingcai.com/1319.html

http://blog.csdn.net/watsy/article/details/14161201

### 安装与入门介绍

linux下可以执行
```python
apt-get install python-bs4  
```

windows下可以利用python的安装包工具来安装

```python
easy_install beautifulsoup4  
  
pip install beautifulsoup4  
```

当然可以从网站上直接下载源码进行本地安装 

https://pypi.python.org/pypi/beautifulsoup4/4.3.2

```
pip install setup.py -build
```

接下来我们运用官方文档给出的例子来进行说明和介绍

```python
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""
```

创建beautifulsoup4 对象并对html进行解析

```python
from bs4 import BeautifulSoup
soup = BeautifulSoup(html)

print soup.prettify()# 格式化输出

# <html>
#  <head>
#   <title>
#    The Dormouse's story
#   </title>
#  </head>
#  <body>
#   <p class="title">
#    <b>
#     The Dormouse's story
#    </b>
#   </p>
#   <p class="story">
#    Once upon a time there were three little sisters; and their names were
#    <a class="sister" href="http://example.com/elsie" id="link1">
#     Elsie
#    </a>
#    ,
#    <a class="sister" href="http://example.com/lacie" id="link2">
#     Lacie
#    </a>
#    and
#    <a class="sister" href="http://example.com/tillie" id="link2">
#     Tillie
#    </a>
#    ; and they lived at the bottom of a well.
#   </p>
#   <p class="story">
#    ...
#   </p>
#  </body>
# </html>
```
beautifulsoup 提供了丰富的导航索引方法。可以直接**.title**获取  title 标签，若想获取标签内容，可以通过**.title.string**得到，想要查询上一级标签，可以通过**.title.parent**方式。因为beautifulsoup可以比较完善解析html,所以可以通过.p, .a等方式选取html文档中的段落（p）、链接（a）标签。同时可以通过**find_all('a')**等一系列函数查找到所有a标签等信息。


```python 
soup.title
# <title>The Dormouse's story</title>

soup.title.name
# u'title'

soup.title.string
# u'The Dormouse's story'

soup.title.parent.name
# u'head'

soup.p
# <p class="title"><b>The Dormouse's story</b></p>

soup.p['class']
# u'title'

soup.a
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>

soup.find_all('a')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.find(id="link3")
# <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>

```
beautifulsoup 提供了丰富的导航索引方法。可以直接**.title**获取 title  标签，若想获取标签内容，可以通过**.title.string**得到，想要查询上一级标签，可以通过**.title.parent**方式。因为beautifulsoup可以比较完善解析html,所以可以通过.p, .a等方式选取html文档中的段落（p）、链接（a）标签。同时可以通过**find_all('a')**等一系列函数查找到所有  a 标签等信息。下面将分别详细介绍

### 文档处理与信息挖掘

四大对象种类

Beautiful Soup将复杂HTML文档转换成一个复杂的树形结构,每个节点都是Python对象,所有对象可以归纳为4种:

* Tag
* NavigableString
* BeautifulSoup
* Comment

#### Tag
Tag 这里其实就是html 语言中各种标签。下面 title, a 等等HTML标签加上里面包括的内容就是 Tag。运用beautifulsoup可以很容易导出

```html 
<title>The Dormouse's story</title>
<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>
```

我们可以利用 soup加标签名轻松地获取这些标签的内容，这比利用正则表达式提取要方便很多。但注意，它查找的是在所有内容中的第一个符合要求的标签，如果要查询所有的标签，我们在后面进行介绍。

```python 
tag = soup.b
type(tag)

print soup.title
#<title>The Dormouse's story</title>

print soup.a
#<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>

```

对于 Tag，有很多重要属性，可以方便我们进行树型搜索，不过再介绍Navigating the tree and Searching the tree 之前先介绍两个重要的属性：name 和 attrs。

```python
# name 

print soup.name
print soup.head.name
#[document]
#head

# attrs
print soup.p.attrs
#{'class': ['title'], 'name': 'dromouse'}

```
如果我们想要单独获取某个属性，可以这样，例如我们获取它的 class 叫什么。同时还可以利用get方法，传入属性的名称，二者是等价的。我们还可以更改这些属性和内容

```python
print soup.p['class']
#['title']

print soup.p.get('class')
#['title']

soup.p['class']="newClass"
print soup.p
#<p class="newClass" name="dromouse"><b>The Dormouse's story</b></p>

del soup.p['class']
print soup.p
#<p name="dromouse"><b>The Dormouse's story</b></p>
```
#### NavigableString
我们获得列了标签，如果我们想进一步获得标签的内容的文字，我们可以用.string 操作,它的类型是一个 NavigableString，翻译过来叫"可以遍历的字符串"，不过我们最好还是称它英文名字吧。

```python
print soup.p.string
#The Dormouse's story

print type(soup.p.string)
#<class 'bs4.element.NavigableString'>
```

#### BeautifulSoup
BeautifulSoup 对象表示的是一个文档的全部内容.大部分时候,可以把它当作 Tag 对象，是一个特殊的 Tag，我们可以分别获取它的类型，名称，以及属性来感受一下


#### Comment
Comment 对象是一个特殊类型的 NavigableString 对象，其实输出的内容**不包括注释符号**，但是如果不好好处理它，可能会对我们的文本处理造成意想不到的麻烦。

我们找一个带注释的标签
```python
print soup.a
print soup.a.string
print type(soup.a.string)

#<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>
# Elsie 
#<class 'bs4.element.Comment'>
```


### 遍历文档树

#### 子节点

* .contents  可以将tag的子节点以列表的方式输出

* .children  是一个 list 生成器对象
```python
print soup.head.contents 
#[<title>The Dormouse's story</title>]
print soup.head.contents[0]
#<title>The Dormouse's story</title>
print soup.head.children
#<listiterator object at 0x7f71457f5710>
#遍历一下 获取children中内容
for child in  soup.body.children:
    print child
```

.contents 和 .children 属性仅包含tag的直接子节点，.descendants 属性可以对所有tag的子孙节点进行递归循环，和 children类似，我们也需要遍历获取其中的内容。

for child in soup.descendants:
    print child


#### 获取节点内容

如果一个标签里面没有标签了，那么 .string 就会返回标签里面的内容。如果标签里面只有唯一的一个标签了，那么 .string 也会返回最里面的内容。如果tag包含了多个子节点,tag就无法确定，string 方法应该调用哪个子节点的内容, .string 的输出结果是 None
```python
print soup.head.string
#The Dormouse's story
print soup.title.string
#The Dormouse's story

print soup.html.string
#None

```
要想获取多个内容，可以利用.strings，不过也需要遍历。.stripped_strings 可以帮助我们除去多余空格

```python
for string in soup.strings:
    print(repr(string))
    # u"The Dormouse's story"
    # u'\n\n'
    # u"The Dormouse's story"
    # u'\n\n'
    # u'Once upon a time there were three little sisters; and their names were\n'
    # u'Elsie'
    # u',\n'
    # u'Lacie'
    # u' and\n'
    # u'Tillie'
    # u';\nand they lived at the bottom of a well.'
    # u'\n\n'
    # u'...'
    # u'\n'
    
for string in soup.stripped_strings:
    print(repr(string))
    # u"The Dormouse's story"
    # u"The Dormouse's story"
    # u'Once upon a time there were three little sisters; and their names were'
    # u'Elsie'
    # u','
    # u'Lacie'
    # u'and'
    # u'Tillie'
    # u';\nand they lived at the bottom of a well.'
    # u'...'
```

#### 父节点

* .parent 属性 得到直接父节点

* .parents 属性 得到全部父节点

```python
p = soup.p
print p.parent.name
#body

content = soup.head.title.string
print content.parent.name
#title

content = soup.head.title.string
for parent in  content.parents:
    print parent.name
    
```


#### 兄弟节点

* .next_sibling  .previous_sibling 属性
    * 兄弟节点可以理解为和本节点处在同一级的节点，.next_sibling 属性获取了该节点的下一个兄弟节点，.previous_sibling 则与之相反，如果节点不存在，则返回 None
    * 注意：实际文档中的tag的 .next_sibling 和 .previous_sibling 属性通常是字符串或空白，因为空白或者换行也可以被视作一个节点，所以得到的结果可能是空白或者换行
    

```python    
print soup.p.next_sibling
#       实际该处为空白
print soup.p.prev_sibling
#None   没有前一个兄弟节点，返回 None
print soup.p.next_sibling.next_sibling
#<p class="story">Once upon a time there were three little sisters; and their names were
#<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>,
#<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a> and
#<a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>;
#and they lived at the bottom of a well.</p>
#下一个节点的下一个兄弟节点是我们可以看到的节点


for sibling in soup.a.next_siblings:
    print(repr(sibling))
    # u',\n'
    # <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>
    # u' and\n'
    # <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>
    # u'; and they lived at the bottom of a well.'
    # None

```

#### 前后节点

* .next_siblings  .previous_siblings 属性
    * 得到同一级的所有兄弟节点，以迭代形式给出。
* .next_element  .previous_element 属性
    * 与 之前的不同，它并不是针对于兄弟节点，而是在所有节点，不分层次


```python
for element in last_a_tag.next_elements:
    print(repr(element))
# u'Tillie'
# u';\nand they lived at the bottom of a well.'
# u'\n\n'
# <p class="story">...</p>
# u'...'
# u'\n'
# None
```



### 搜索文档树
筛选数据我们势必遇到如何快速定位的问题，beautifulsoup 向我们提供了几个很好的find方法帮助我们快速定位

* find_all( name , attrs , recursive , text , **kwargs )
    * name 参数  
        * 字符串：最简单的过滤器是字符串.在搜索方法中传入一个字符串参数,Beautiful Soup会查找与字符串完整匹配的内容
        * 正则表达式：如果传入正则表达式作为参数,Beautiful Soup会通过正则表达式的 match() 来匹配内容
        * 列表：如果传入列表参数,Beautiful Soup会将与列表中任一元素匹配的内容返回.
        * True：True 可以匹配任何值,会查找到所有的tag,但是不会返回字符串节点
        * 方法：如果没有合适过滤器,那么还可以定义一个方法,方法只接受一个元素参数 ,如果这个方法返回 True 表示当前元素匹配并且被找到,如果不是则反回 False
    * keyword 参数 
        * 包括 id, href, 等
        * 在这里我们想用 class 过滤，不过 class 是 python 的关键词，这怎么办？加个下划线就可以
    * text 参数 
        * 通过 text 参数可以搜搜文档中的字符串内容.与 name 参数的可选值一样, text 参数接受 字符串 , 正则表达式 , 列表, True
    * limit 参数
        * find_all() 方法返回全部的搜索结构,如果文档树很大那么搜索会很慢.如果我们不需要全部结果,可以使用 limit 参数限制返回结果的数量.效果与SQL中的limit关键字类似,当搜索到的结果数量达到 limit 的限制时,就停止搜索返回结果.
    * recursive 参数
        * 调用tag的 find_all() 方法时,Beautiful Soup会检索当前tag的所有子孙节点,如果只想搜索tag的直接子节点,可以使用参数 recursive=False .
        
```python

### name参数
## 字符串
print soup.find_all('a')
#[<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>, <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>, <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

## 正则表达式
import re
for tag in soup.find_all(re.compile("^b")):
    print(tag.name)
# body
# b

## 列表
soup.find_all(["a", "b"])
# [<b>The Dormouse's story</b>,
#  <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

## TRUE 
for tag in soup.find_all(True):
    print(tag.name)
# html
# head
# title
# body
# p
# b
# p
# a
# a

## 方法（隐函数形式）
def has_class_but_no_id(tag):
    return tag.has_attr('class') and not tag.has_attr('id')


soup.find_all(has_class_but_no_id)
# [<p class="title"><b>The Dormouse's story</b></p>,
#  <p class="story">Once upon a time there were...</p>,
#  <p class="story">...</p>]

### keyword
soup.find_all(id='link2')
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

soup.find_all(href=re.compile("elsie"))
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

soup.find_all(href=re.compile("elsie"), id='link1')
# [<a class="sister" href="http://example.com/elsie" id="link1">three</a>]

## 利用 class 过滤的方法
soup.find_all("a", class_="sister")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]


## 有些tag属性在搜索不能使用,比如HTML5中的 data-* 属性
data_soup = BeautifulSoup('<div data-foo="value">foo!</div>')
data_soup.find_all(data-foo="value")
# SyntaxError: keyword can't be an expression

## 但是可以通过 find_all() 方法的 attrs 参数定义一个字典参数来搜索包含特殊属性的tag
data_soup.find_all(attrs={"data-foo": "value"})
# [<div data-foo="value">foo!</div>]

### text 参数
soup.find_all(text="Elsie")
# [u'Elsie']

soup.find_all(text=["Tillie", "Elsie", "Lacie"])
# [u'Elsie', u'Lacie', u'Tillie']

soup.find_all(text=re.compile("Dormouse"))
[u"The Dormouse's story", u"The Dormouse's story"]

### limit参数

soup.find_all("a", limit=2)
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

### recursive参数的功能 

soup.html.find_all("title")
# [<title>The Dormouse's story</title>]

soup.html.find_all("title", recursive=False)
# []

```


        

除此之外，还有如下的方法，值得注意的是，beautifulsoup中很多方法是成对出现的 加"s"与不加"s"区别就在于不加的话返回的是第一个符合条件的结果，而加的话是返回所有符合条件的迭代器：

* find( name , attrs , recursive , text , \** kwargs )：  find_all() 方法的返回结果是值包含带查找一个元素的列表（所有结果）,而 find() 方法直接返回所查的元素的第一个结果

* find_parents()与find_parent() : 与find_all() 和 find() 只搜索当前节点的所有子节点,孙子节点等不同，find_parents() 和 find_parent() 用来搜索当前节点的父辈节点,搜索方法与普通tag的搜索方法相同,搜索文档搜索文档包含的内容

* find_next_siblings()与find_next_sibling()：这2个方法通过 .next_siblings 属性对当 tag 的所有后面解析的兄弟 tag 节点进行迭代, find_next_siblings() 方法返回所有符合条件的后面的兄弟节点,find_next_sibling() 只返回符合条件的后面的第一个tag节点

* find_previous_siblings()与find_previous_sibling()：这2个方法通过 .previous_siblings 属性对当前 tag 的前面解析的兄弟 tag 节点进行迭代, find_previous_siblings() 方法返回所有符合条件的前面的兄弟节点, find_previous_sibling() 方法返回第一个符合条件的前面的兄弟节点

* find_all_next()与find_next()： 这2个方法通过 .next_elements 属性对当前 tag 的之后的 tag 和字符串进行迭代, find_all_next() 方法返回所有符合条件的节点, find_next() 方法返回第一个符合条件的节点

* find_all_previous() 和 find_previous()
这2个方法通过 .previous_elements 属性对当前节点前面的 tag 和字符串进行迭代, find_all_previous() 方法返回所有符合条件的节点, find_previous()方法返回第一个符合条件的节点

### CSS选择器

html文档中，CSS样式也是一个非常重要的帮助我们定位相关信息的元素，好在beautifulsoup支持我们利用css样式来进行定位搜索，给我们增添了更多的灵活性。所用到的方法是 soup.select()，返回类型是 list

* 通过标签名查找

```python
print soup.select('title') 
#[<title>The Dormouse's story</title>]

print soup.select('a')
#[<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>, <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>, <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

print soup.select('b')
#[<b>The Dormouse's story</b>]

```

* 通过类名查找：这里只能单独查找一个类名，如果类名中存在空格，这个方式就失效了

```python
print soup.select('.sister')
#[<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>, <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>, <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

```

* 通过 id 名查找

```python
print soup.select('#link1')
#[<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>]
```

* 组合查找： 这种方式提供了复合查找的可能，帮助我们更为精确的定位

```python
print soup.select('p #link1')
#[<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>]

print soup.select("head > title")
#[<title>The Dormouse's story</title>]
```

* 属性查找 : 属性需要用中括号括起来，注意属性和标签属于同一节点，所以中间不能加空格，否则会无法匹配到。

```python
print soup.select('a[class="sister"]')
#[<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>, <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>, <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

print soup.select('a[href="http://example.com/elsie"]')
#[<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>]

#同样，属性仍然可以与上述查找方式组合，不在同一节点的空格隔开，同一节点的不加空格
print soup.select('p a[href="http://example.com/elsie"]')
#[<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>]


```

### 其他重要功能
