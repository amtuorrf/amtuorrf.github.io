---
title: 使用python-selenium.webdriver库爬取netcraft网站上的域名
author: AmtuOrRF
categories: [Misc]
tags: [信息收集]
---
### 剧情
1. 小强最近发现一个可以搜索子域名的网站, 他试着搜索了一下`edu.cn`, 结果得到了很多 三级域名、四级域名。

![[Pasted image 20220717144616.png]]
![[Pasted image 20220717144729.png]]

2. 小强现在想使用并不熟练的python requests库来爬取网站上的子域名信息。过程中他发现, requests 请求的并不是他想要的内容，网站似乎做了防爬.

```python
# cat getsubdomain.py 
import requests

url = "https://searchdns.netcraft.com/"
t1 = requests.get(url)
print(t1.text)

# python3 getsubdomain.py > 1.html
```

![[Pasted image 20220717145708.png]]

3. 小强在网上查询了许多资料，且学会了一下东西，接下来跟着小强一步一步操作，帮助小强完成任务.


## 配置
操作系统环境 : windows 10
浏览器 : Google Chrome
python3.9+


1. 首先下载安装 `python` 第三方库 `selenium`

```cmd
PS C:\Users\GameG\test> pip.exe install selenium
```

不出意外的话应该可以安装成功



2. 配置浏览器相关东西.

查看Google Chrome版本
![[Pasted image 20220717151033.png]]

Version： 103.0.5060.114

在这个网站下载一个小软件，版本就选择和Google Chrome 差不多的.

[chromedriver.storage.googleapis.com/index.html](http://chromedriver.storage.googleapis.com/index.html)

我选了这这个.

![[Pasted image 20220717151336.png]]

下载 chromedriver_win32.zip 文件.

![[Pasted image 20220717151403.png]]

将文件解压，把解压后的文件放到 python 的安装目录.

C:\Users\用户名\AppData\Local\Programs\Python\Python310\

![[Pasted image 20220717151608.png]]

然后添加到环境变量。


![Desktop View](/assets/img/wz-images/Pasted image 20220717151910.png){: w="700"}
3. 测试脚本

不出意外的话应该可以了

```python
from selenium import webdriver
from selenium.webdriver import ActionChains
from selenium.webdriver.common.by import By
import time
driver = webdriver.Chrome()
#driver.implicitly_wait(3)

# 要收集的域名
domain = "edu.cn"

driver.get(f"https://searchdns.netcraft.com/?restriction=site+contains&host={domain}&position=limited")
#a = driver.find_element_by_css_name('result')

def main():
    a = driver.find_elements(By.CLASS_NAME, 'results-table__host')

	# 保存获取的子域名
    f = open('subdomain.txt','a+')
    for i in a:
        print(i.text)
        f.write(i.text+'\r')
    f.close()
    time.sleep(1)
    driver.execute_script("window.scrollTo(0, 1100)")
    time.sleep(2)
    driver.find_element(By.CLASS_NAME,'fa-chevron-circle-right').click()

for i in range(100):
    time.sleep(1)
    main()
```

