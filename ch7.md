## 提高效率：多线程与并行爬虫

对网络爬虫来说，一个绕不过去的问题就是**效率**。对单线程的爬虫，一方面效率很慢，另一方面，持续访问容易引起网络服务器反爬虫的制约。因此，利用多线程提高爬虫效率成为我们不二的选择。

幸运的是python目前多线程支持已经比较完善，甚至有人已经开发出针对网络爬虫的多线程程序包，方便程序员和研究者进行相关的任务工作。这里，我简单介绍一下利用多线程thread 和队列Queue 进行多线程的编写以及如何把网络爬虫纳入多线程框架。

### 多线程参考资料

[w3cchool多线程入门](http://www.runoob.com/python/python-multithreading.html)


### 简单多线程的尝试
python利用thread类来进行线程的声明和运行，下面引用w3cschool一个简单的例子来说明多线程问题

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

import thread
import time

# 为线程定义一个函数
def print_time( threadName, delay):
   count = 0
   while count < 5:
      time.sleep(delay)
      count += 1
      print "%s: %s" % ( threadName, time.ctime(time.time()) )

# 创建两个线程
try:
   thread.start_new_thread( print_time, ("Thread-1", 2, ) )
   thread.start_new_thread( print_time, ("Thread-2", 4, ) )
except:
   print "Error: unable to start thread"

while 1:
   pass
   
``` 

python主要通过threading 和thread模块对线程进行操作，里面最为重要的函数为run()(存储工作函数)和start()(启动线程活动)。下面的例子就是threading 进行线程的操作

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

import threading
import time

exitFlag = 0

class myThread (threading.Thread):   #继承父类threading.Thread
    def __init__(self, threadID, name, counter):
        threading.Thread.__init__(self)
        self.threadID = threadID
        self.name = name
        self.counter = counter
    def run(self):                   #把要执行的代码写到run函数里面 线程在创建后会直接运行run函数 
        print "Starting " + self.name
        print_time(self.name, self.counter, 5)
        print "Exiting " + self.name

def print_time(threadName, delay, counter):
    while counter:
        if exitFlag:
            thread.exit()
        time.sleep(delay)
        print "%s: %s" % (threadName, time.ctime(time.time()))
        counter -= 1

# 创建新线程
thread1 = myThread(1, "Thread-1", 1)
thread2 = myThread(2, "Thread-2", 2)

# 开启线程
thread1.start()
thread2.start()


```
### 加锁和同步
多线程会遇到同步性的问题，多个线程对同一个数据存储空间进行修改会出现不可预料的后果，为保证数据正确性，需要“加锁”和同步操作

锁有两种状态——锁定和未锁定。每当一个线程比如"set"要访问共享数据时，必须先获得锁定；如果已经有别的线程比如"print"获得锁定了，那么就让线程"set"暂停，也就是同步阻塞；等到线程"print"访问完毕，释放锁以后，再让线程"set"继续。


```python

import threading
import time

class myThread (threading.Thread):
    def __init__(self, threadID, name, counter):
        threading.Thread.__init__(self)
        self.threadID = threadID
        self.name = name
        self.counter = counter
    def run(self):
        print "Starting " + self.name
       # 获得锁，成功获得锁定后返回True
       # 可选的timeout参数不填时将一直阻塞直到获得锁定
       # 否则超时后将返回False
        threadLock.acquire()
        print_time(self.name, self.counter, 3)
        # 释放锁
        threadLock.release()

def print_time(threadName, delay, counter):
    while counter:
        time.sleep(delay)
        print "%s: %s" % (threadName, time.ctime(time.time()))
        counter -= 1

threadLock = threading.Lock()
threads = []

# 创建新线程
thread1 = myThread(1, "Thread-1", 1)
thread2 = myThread(2, "Thread-2", 2)

# 开启新线程
thread1.start()
thread2.start()

# 添加线程到线程列表
threads.append(thread1)
threads.append(thread2)

# 等待所有线程完成
for t in threads:
    t.join()
print "Exiting Main Thread"


```

### 线程队列
线程队列，向我们提供一个很好的框架利用多线程来处理任务序列，可以把队列想象成一个工厂的流水线，在一端放上原材料，即我们的任务；随着传送带运行，旁边的工人们（多线程）按序从中拿取任务进行完成，完成后的任务序列仍然保持原本的顺序，方便我们后续操作（保序性）。

Queue模块中的常用方法:

* Queue.qsize() 返回队列的大小
* Queue.empty() 如果队列为空，返回True,反之False
* Queue.full() 如果队列满了，返回True,反之False
* Queue.full 与 maxsize 大小对应
* Queue.get([block[, timeout]])获取队列，timeout等待时间
* Queue.get_nowait() 相当Queue.get(False)
* Queue.put(item) 写入队列，timeout等待时间
* Queue.put_nowait(item) 相当Queue.put(item, False)
* Queue.task_done() 在完成一项工作之后，Queue.task_done()函数向任务已经完成的队列发送一个信号
* Queue.join() 实际上意味着等到队列为空，再执行别的操作

```python

#!/usr/bin/python
# -*- coding: UTF-8 -*-

import Queue
import threading
import time

exitFlag = 0

class myThread (threading.Thread):
    def __init__(self, threadID, name, q):
        threading.Thread.__init__(self)
        self.threadID = threadID
        self.name = name
        self.q = q
    def run(self):
        print "Starting " + self.name
        process_data(self.name, self.q)
        print "Exiting " + self.name

def process_data(threadName, q):
    while not exitFlag:
        queueLock.acquire()
        if not workQueue.empty():
            data = q.get()
            queueLock.release()
            print "%s processing %s" % (threadName, data)
        else:
            queueLock.release()
        time.sleep(1)

threadList = ["Thread-1", "Thread-2", "Thread-3"]
nameList = ["One", "Two", "Three", "Four", "Five"]
queueLock = threading.Lock()
workQueue = Queue.Queue(10)
threads = []
threadID = 1

# 创建新线程
for tName in threadList:
    thread = myThread(threadID, tName, workQueue)
    thread.start()
    threads.append(thread)
    threadID += 1

# 填充队列
queueLock.acquire()
for word in nameList:
    workQueue.put(word)
queueLock.release()

# 等待队列清空
while not workQueue.empty():
    pass

# 通知线程是时候退出
exitFlag = 1

# 等待所有线程完成
for t in threads:
    t.join()
print "Exiting Main Thread"

```

### 爬虫的多线程框架

爬虫进入多线程，需要对爬虫框架进行一定的分析与分解：

* 获得信息链接
* 进行相关定位
* 获取信息
* 保存数据

把爬虫嵌入到多线程框架，无论是利用传统的urllib还是高级的mechanize 甚至是selenium。操作的关键就在于线程的**工作函数** 上述的步骤都是在工作函数中完成的。此外，对于多线程以及爬虫来说，更为关键的地方在于错误处理与错误恢复。爬虫最为消耗时间的地方就是错误处理过程了。这一方面需要技巧、经验与艺术的结合。下面的例子是针对landchina.com土地交易信息的抓取。供大家参考：



```python

### 引用程序包####
import sys as sys
import codecs
import time, re
import glob,os,copy

from datetime import date, timedelta,datetime
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait, Select
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.keys import Keys
from selenium.common.exceptions import NoSuchElementException, NoAlertPresentException
import selenium.common.exceptions as S_exceptions



reload(sys)
sys.setdefaultencoding('utf-8')

### 初始化和相关变量声明 ###

list_content=[
        u"行政区",
        u"电子监管号",
        u"项目名称", 
        u"项目位置" ,
        u"面积(公顷)", 
        u"土地来源" ,
        u"土地用途" , 
        u"供地方式" ,  
        u"土地使用年限" ,
        u"行业分类", 
        u"土地级别", 
        u"成交价格(万元)" ,
        u"土地使用权人" , 
        u"约定容积率下限" ,
        u"约定容积率上限", 
        u"约定交地时间" ,
        u"约定开工时间",  
        u"约定竣工时间",
        u"批准单位",
        u"合同签定日期" ]

dic_content={
u"行政区":"mainModuleContainer_1855_1856_ctl00_ctl00_p1_f1_r1_c2_ctrl",
u"电子监管号":"mainModuleContainer_1855_1856_ctl00_ctl00_p1_f1_r1_c4_ctrl",
u"项目名称":"mainModuleContainer_1855_1856_ctl00_ctl00_p1_f1_r17_c2_ctrl", 
u"项目位置":"mainModuleContainer_1855_1856_ctl00_ctl00_p1_f1_r16_c2_ctrl" ,
u"面积(公顷)":"mainModuleContainer_1855_1856_ctl00_ctl00_p1_f1_r2_c2_ctrl", 
u"土地来源":"mainModuleContainer_1855_1856_ctl00_ctl00_p1_f1_r2_c4_ctrl" ,
u"土地用途":"mainModuleContainer_1855_1856_ctl00_ctl00_p1_f1_r3_c2_ctrl" , 
u"供地方式": "mainModuleContainer_1855_1856_ctl00_ctl00_p1_f1_r3_c4_ctrl",  
u"土地使用年限":"mainModuleContainer_1855_1856_ctl00_ctl00_p1_f1_r19_c2_ctrl" ,
u"行业分类":"mainModuleContainer_1855_1856_ctl00_ctl00_p1_f1_r19_c4_ctrl" , 
u"土地级别":"mainModuleContainer_1855_1856_ctl00_ctl00_p1_f1_r20_c2_ctrl" , 
u"成交价格(万元)":"mainModuleContainer_1855_1856_ctl00_ctl00_p1_f1_r20_c4_ctrl" ,
u"土地使用权人" :"mainModuleContainer_1855_1856_ctl00_ctl00_p1_f1_r9_c2_ctrl", 
u"约定容积率下限":"mainModuleContainer_1855_1856_ctl00_ctl00_p1_f2_r1_c4_ctrl" ,
u"约定容积率上限":"mainModuleContainer_1855_1856_ctl00_ctl00_p1_f2_r1_c2_ctrl", 
u"约定交地时间" : "mainModuleContainer_1855_1856_ctl00_ctl00_p1_f1_r21_c4_ctrl",
u"约定开工时间": "mainModuleContainer_1855_1856_ctl00_ctl00_p1_f1_r22_c2_ctrl",  
u"约定竣工时间":"mainModuleContainer_1855_1856_ctl00_ctl00_p1_f1_r22_c4_ctrl",
u"批准单位" :"mainModuleContainer_1855_1856_ctl00_ctl00_p1_f1_r14_c2_ctrl",
u"合同签定日期":"mainModuleContainer_1855_1856_ctl00_ctl00_p1_f1_r14_c4_ctrl", 
}


import re,sys
from time import ctime,sleep
import Queue
import threading
import time


#### 多线程的类框架 ####
#thread
threadLock = threading.Lock()

class spiderThread (threading.Thread):
    def __init__(self,Work_fun,threadID,task_content,workqueue,exitFlag):
        threading.Thread.__init__(self)
        self.threadID = threadID
        self.task_content = task_content
        self.workqueue = workqueue
        self.exitFlag=exitFlag
        self.Work_fun=Work_fun
                
    def run(self):  
        while not self.exitFlag[0]:
            threadLock.acquire()
            if not self.workqueue.empty() :
                Data_dict = self.workqueue.get()
                ## 加入错误信息记录和处理
                ### 如果无法打开服务器端的网站，那么线程中断结束
                ## 加入日志记录
                
                
                print str(self.threadID)+" is processing " 
                result=Work_fun(Data_dict)
                ## 判断 result 状态进行相关的出错记录和
                print str(self.threadID)+" is done "                
#                self.workqueue.task_done()                
                threadLock.release()
            else :
                threadLock.release()
            sleep(1)
        

    
    def set_exitFlag(self,Flag):
        self.exitFlag[0]=Flag[0]
        print "thread",self.threadID, "exitFlag is "+str(self.exitFlag[0])+"\n"
    
    
    


# Queue
class Queue_Frame():
    def __init__(self,exitFlag=[0]):
        self.workQueue = Queue.Queue()
        self.threads = []
        self.exitFlag=exitFlag
    
    def init_thread(self,Work_fun,task_content,num_thread):
        print "init thread"
        for tID in range(1,num_thread+1):
            thread =spiderThread(Work_fun,tID, task_content, self.workQueue,self.exitFlag)
            thread.start()
            self.threads.append(thread)
            print "thread",tID,"is in the Queue"
    
    def put_workQueue_run(self,list_thread):
        threadLock.acquire()
        for num in range(0,len(list_thread)) :
            self.workQueue.put(list_thread[num])
        threadLock.release()
        
        # 处理一下中途中断退出的问题
        # 以及加入chunk分块如queue的情况
        
        while not self.workQueue.empty():
            pass
        sleep(1)        
        
        
        
        self.exitFlag[0]=1
        for tID in range(0,num_thread):
            self.threads[tID].set_exitFlag(self.exitFlag)
        
        for t in self.threads:
            t.join()
            
        print "finish and exit"
        
    def test_print(self):
        print "Queue start"
                    
                    
### 多线程工作函数 ###

def webs_record(driver,link,count_url):
    
    ## 加入打开存储txt文件的判断工作，以及判断是哪个月的
    ## 加入处理反爬虫电子狗的情况
    
    result=[]
        
    flag=0
    count=0
    while flag==0:

        try:
            driver.get(link_line)
            WebDriverWait(driver,60).until(EC.presence_of_element_located((By.ID,"mainModuleContainer_1855_1856_ctl00_ctl00_p1_f1_r14_c4_ctrl")))
            time.sleep(1)
            flag=1
            
        except:
            count=count+1
            if "a70.htm" in str(driver.current_url):
                driver=login_internet(driver)
                time.sleep(10)
                driver.get(link_line)
                
                
        # 成功获得信息页面
        if flag==1:
            break
        # 连续打开多次没有成功
        if count>=3:
            print "can not open this page error, save link "
            failed_record(link_line,fail_name)
            
            break

    # 信息的抓取
    if count < 3:
        
        temp_list_data={}

        for name, ID_element in dic_content.items():
            try:
                temp_list_data[name]=driver.find_element_by_id(ID_element).text
            except:
                temp_list_data[name]=""
            print name, u":" ,  
            try:
                print temp_list_data[name].decode('utf-8','ignore')
            except :
                pass

        data_list={}
        data_list=copy.deepcopy(temp_list_data)
        # for j in range(len(list_content)):
            # temp = data_list[list_content[j]]
            # sheet.write(i+1,j,temp.decode("utf-8"))
        
        data_export = codecs.open(".\\info\\"+temp_file+"export.txt",'a+',"utf-8-sig")
        
        writeline="\t".join(data_list[list_content[j]] for j in range(len(list_content)))
        writeline=writeline+"\t"+str(link_line).rstrip()
        writeline=writeline+"\n"
        # writeline =ID_num+","+xzq+","+dzjgh+","+xmwz+","+mj_gq+","+tdly+","+tdyt+","+gdfs+","+tdsynx+","+hyfl+","+tdjb+","+cjjg+","+tdsyqr+","+rjlsx+","+rjlxx+","+htqdrq+","+"\n"
        data_export.write(writeline.encode("utf-8-sig"))
        print u"<--------------!-------------->"
        count_url=count_url+1
        result=['1',count_url]
    else:
        count_url=count_url+1
        result=['0',count_url,link_line]

    
    return result
