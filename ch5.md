## python爬虫的简单应用
这一部分简单介绍几个利用urllib 和 beautifulsoup 进行简单爬虫应用的例子，方便大家熟悉爬虫操作方法以及相关命令。

* 登录人人获取状态信息
* 摘取人人好友列表
* 抓取图片信息
* 下载文件

```python
#-*- coding: utf-8 -*-
import time
import sys
import urllib
import urllib2
import cookielib
import os
import re
from bs4 import BeautifulSoup
 
#import js2html
class renrenSpider:    
 
    def __init__(self,email,password):
        self.email = email
        self.password = password
        self.domain = 'renren.com'
        self.id = ''
        self.sid = ''
        try:
            self.cookie = cookielib.CookieJar()
            self.cookieProc = urllib2.HTTPCookieProcessor(self.cookie)
        except:
            raise
        else:
            opener = urllib2.build_opener(self.cookieProc)
            opener.addheaders = [('User-Agent','Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/40.0.2214.111 Safari/537.36')]
            urllib2.install_opener(opener)
     
    def login(self):
        url='http://3g.renren.com/login.do' #登陆人人网3g首页
        postdata = {
                    'email':self.email,
                    'password':self.password,
                    }
         
        req = urllib2.Request(url,urllib.urlencode(postdata))
        index = urllib2.urlopen(req).read()
        indexSoup = BeautifulSoup(index)  #首页soup
                     
        indexFile = open('index.html','w')
        indexFile.write(indexSoup.prettify())
        indexFile.close()
        tmp = indexSoup.select('.cur')[0]
        idHref =  tmp.parent.contents[2]['href']
        m = re.findall(r"\d{6,}",str(idHref)) #正则匹配获取用户的id
        self.id = m[0]
        print "用户ID为：" + self.id
 
        m = re.findall(r"(?<=sid=).*?(?=&)",str(idHref)) #正则匹配获取用户的sid
        self.sid = m[0]
        print "用户的sid为:" + self.sid
 
    def getStatus(self):
        #获取个人状态页面
 
        statusDate = [] #保存状态时间
        statusContent = [] #保存状态内容
        originStatusContent = []  #保存转发原文内容
        url = 'http://3g.renren.com/profile.do' #登陆到个人主页
        profileGetData = {
                          'id':str(self.id),
                          'sid':self.sid  
                         }
        req = urllib2.Request(url,urllib.urlencode(profileGetData))
        profile = urllib2.urlopen(req).read()
        profileSoup = BeautifulSoup(profile)
        # print profileSoup.prettify()
        url = profileSoup.select('.sec')[5].find_all('a')[3]['href']    #获得链接
 
        req = urllib2.Request(url)
        statusFile = urllib2.urlopen(req).read()
        statusSoup = BeautifulSoup(statusFile)
        statusFile = open("status.html",'w')
        statusFile.write( statusSoup.prettify())
        statusFile.close()
         
        totalPageHtml = statusSoup.select(".gray")[0].contents
        totalPage = re.findall(r"(?<=/)\d+(?=[^\d])",str(totalPageHtml))
        totalPage = int(totalPage[0])
        print "总共有:"+str(totalPage)+"页"
         
        nowPage = 1
        #totalPage = 15
        while (nowPage<=totalPage) :
            print "当前正在获取第"+str(nowPage)+"页状态信息"
            statusList = statusSoup.select(".list")[0].children
            for child in statusList:
                #statusNum = statusNum+1
                if (child.select(".time")):# Step1: 找时间戳，确定为状态信息
                    statusDate.append(child.select(".time")[0].string) 
                    if (child.select(".forward")): #step2:找class名为forward的内容，这部分为转的状态
                        tempStr = str(child.a.next_element)
                        m = re.findall(r"^.*?(?=转自)",tempStr)                            
                        if m: 
                            statusContent.append(m[0])
                        else :
                            statusContent.append("无")
 
                        originStatusContent.append(child.select(".forward")[0].a.next_element.next_element) 
                    else:   #step3:这些是原创内容，直接保存
                       statusContent.append(child.a.next_element)
                       originStatusContent.append("无")                    
            nowPage = nowPage+1
            if (nowPage>totalPage): break
            nextPageUrl =str(statusSoup.select(".l")[0].a['href']) #查找下一页URL并跳转
            req = urllib2.Request(nextPageUrl)
            statusFile = urllib2.urlopen(req).read() 
            statusSoup = BeautifulSoup(statusFile)                            
            # for state in statusList:
            #     print state.name     
         
        finalFile = open("UserData/"+self.id,"w")
        for i in range (0,len(statusDate)):
            finalFile.write(u"第"+str(i+1)+"条:"+"\n")
            finalFile.write(u"时间:"+str(statusDate[i])+"\n")
            finalFile.write(u"状态："+str(statusContent[i])+"\n")
            finalFile.write(u"转发原文："+str(originStatusContent[i])+"\n")
            finalFile.write("\n") 
 
if __name__ == '__main__':
    email = raw_input(u"输入人人网账号")
    password = raw_input(u"输入人人网密码")
    reload(sys)
    sys.setdefaultencoding('utf-8')  
    renrenLogin = renrenSpider(email,password)
    renrenLogin.login()
    renrenLogin.getStatus() 


---
---
## 抓取某人好友列表


```python
# coding: utf-8
import time
import math
import sys
import urllib
import urllib2
import cookielib
import os
import re
from bs4 import BeautifulSoup

reload(sys)   
sys.setdefaultencoding('utf-8')



# #I need to log the renren at first
# #open the pages

# #cookie
# try:
#     cookie = cookielib.CookieJar()
#     cookieProc = urllib2.HTTPCookieProcessor(cookie)
# except:
#     raise
# else:
#     opener = urllib2.build_opener(cookieProc)
#     opener.addheaders = [("User-Agent","Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/40.0.2214.111 Safari/537.36")]
#     urllib2.install_opener(opener)

# url="http://3g.renren.com/getfriendlist.do?f=all&id=231019740&sid=PQIddnfivxat8MZ66tiYBH&96r9zp&ret=profile.do%3Fid%3D231019740-n-%E7%94%B0%E4%B8%B0%E7%9A%84%E4%B8%AA%E4%BA%BA%E4%B8%BB%E9%A1%B5-n-0"
# req = urllib2.Request(url)
# index = urllib2.urlopen(req).read().encode('utf-8')
# indexSoup = BeautifulSoup(index)  #首页soup
                     
# indexFile = open('tianfeng.html','w')
# indexFile.write(indexSoup.prettify())
# indexFile.close()

#------------------------------------------------------------------------------
#spider the friends of tianfeng
infile= file('tianfeng.html').read().encode('utf-8')
statusSoup =  BeautifulSoup(infile)
statusName= [] #保存状态时间
statusSchool = [] #保存状态内容

#user ID 
tmp=statusSoup.select('a[href^="http://3g.renren.com/profile.do?"]')
m = re.findall(r"\d{6,}",str(tmp)) #正则匹配获取用户的id
self_id = m[0]
print "用户ID为：" + self_id


#page 
totalPageHtml = statusSoup.select(".gray")[-1].contents
totalPage = re.findall(r"(?<=/)\d+(?=[^\d])",str(totalPageHtml))
totalPage = int(totalPage[0])
print "总共有:"+str(totalPage)+"页"

finalFile = open("UserData/"+self_id+".txt","w")
finalFile.write("# \t Name \t School \n")
finalFile.close()

divd=4.0

# spider the info
nowPage = 1
count = 0
while (nowPage<=totalPage):
	statusList = statusSoup.select(".list")[0].children
	#print type(statusList)
	#print statusList
	#print dir(statusList)
	print "当前正在获取第"+str(nowPage)+"页状态信息"
	for child in statusList:
		if (child.string):# Step1: 找时间戳，确定为状态信息
			# print type(child)
			# print child.string
			pass
		else:
			# print child
			# print type(child)
			if(child.select('img')):
				print child.select('img')[0]['alt'].decode('utf-8')
				statusName.append(child.select('img')[0]['alt'])
				print child.select('.gray')[0].get_text().strip().decode('utf-8')
				statusSchool.append(child.select('.gray')[0].get_text().strip())

	if((nowPage/divd)==int(nowPage/divd)):
		#result file
		finalFile = open("UserData/"+self_id+".txt","a")
		for i in range (0,len(statusName)):
		    finalFile.write(str(count+i+1)+" \t "+str(statusName[i])+" \t "+str(statusSchool[i])+"\n")
		finalFile.close()
		statusName= [] #保存状态时间
		statusSchool = [] #保存状态内容
		count=count+int(divd*5)

	nowPage = nowPage+1
	if(nowPage>totalPage):
		break
	try:
		nextPageUrl =str(statusSoup.select(".l")[0].a['href']) #查找下一页URL并跳转 
		req = urllib2.Request(nextPageUrl)
		statusFile = urllib2.urlopen(req).read().encode('utf-8')
	except:
		errorfile=open("UserData/stopurl.txt","w+")
		errorfile.write(nextPageUrl)
		errorfile.close()
	else:
		statusSoup = BeautifulSoup(statusFile)

```
