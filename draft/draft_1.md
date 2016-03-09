title: 基于python3的杭电HDU ACM AC自动机
date: 2016-03-08 23:55:27
modified: 2016-03-08 23:55:27
author: Scarb
postid: $POSTID
slug: $SLUG
nicename: 
attachments: 76,77,78,79,80,81,82,83,84,85
posttype: post
poststatus: publish
tags: python, crawler，acm
category: gadget

[TOC]
## 0. 先放几个图……
RunnerUp为我的ID，之后有可能会改掉。
[RunnerUp资料页](http://acm.hdu.edu.cn/userstatus.php?user=RunnerUp)
----------
![RunnerUp][img1]
![No.9][img2]
----------

## 1. 起因……
最近在学习python和python爬虫技术，写了几个弱智爬虫，正想找一个小项目练手。
在久违的登陆HDOJ做了一道题后看了一下排行榜，看到了制作AC自动机的大大们，于是自己也萌生了一个念头。
用python爬虫技术做一个HDOJ的AC自动机，登上Ranklist。
我浏览了一下几位先辈的博客，发现他们用不同的语言写出了自动机，
有C#、C++、node.js
一般都是用POST方法模拟提交。而[beautifulzzzz](http://acm.hdu.edu.cn/userstatus.php?user=beautifulzzzz1)用C#模拟键盘动作进行搜索答案并且提交
于是我开始边学边写自己的python爬虫……

## 2. 规划和思路
一开始规划成了2个模块，一个答案爬取模块和一个OJ操作模块
----------
答案爬取模块

 - 用百度搜索题号
 - 进入搜索结果url，抓取题解
 - 进入问题的discuss，抓取题解
 - 将题解作为一个list返回
 
 HDOJ操作模块
 
 - 登陆账号
 - 判断某题是否AC
 - 提交代码(用不同的编译器)
 
 ----------
 在做题时如果做不出来想要题解，最方便的方法就是看一下discuss，里面一般来说会有题解。所以我就选取抓discuss中的代码。
 后来的试验正面discuss中的代码正确率为50%左右，对于爬虫来说算是比较高的正确率了。还有一些错误的代码和编译出错的代码。
 一开始我写了一个discuss代码爬虫。
 光用discuss的代码可以AC 1000+的题，但也仅限于此，并不能排到排行榜的前列。于是还是得使用百度进行搜索。
 这里选择爬取csdn博客中的代码，csdn博客中不进题解多，而且代码非常好爬取。
 
## 3. 模块
最后一共写了6个模块
分别是
 1. login(self, username, password): 用于登陆OJ
 2. submit(self, problemID, code, language=0): 用于提交代码
 3. getsolved(self, username): 获取用于所有AC的题
 4. getdiscuss(self, problemID): 返回一个list，为从discuss中爬取的代码
 5. getbaidu(self, problemID): 返回一个list，为从百度搜索，从csdn博客中爬取的代码
 6. autorun(self, start=1000, end=5639, interval=5): 开始运行AC自动机，start和end分别是开始和结束的题号，interval是每提交一题见个的时间
 
![ACM_Module][img3]
代码量包含了注释也只有200行，可以体会到python的方便之处。果然“人生苦短，我用python”这句话是有一定的道理的……
 
## 4. 终于开始写代码啦！
在这里选用requests第三方库进行POST和GET的操作，BeautifulSoup第三方库进行html内容的解析。
可以直接用pip install安装上面两个库

先创建一个class， 我命名为'ACM_MODULE'
有一个私有成员，是一个session
添加了浏览器的头
```python
class ACM_Module(object):
    def __init__(self):
        object.__init__(self)
        self.session = requests.Session()
        headers = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/48.0.2564.116 Safari/537.36'
        }
        self.session.headers.update(headers)
```

### 4.1 login模块
HDOJ的登录页面地址为 http://acm.hdu.edu.cn/userloginex.php?action=login
接下来就要用POST模拟登录。
登录的同时并且抓包
![登录抓包][img4]
可以看到有三个data，分别是
username userpass login，我们需要模拟这三个data
```python
        data = {
            'username': username,
            'userpass': password,
            'login': 'Sign In',
        }
```
还有一些headers需要模拟
```python
        headers = {
            'host': 'acm.hdu.edu.cn',
            'origin': 'http://acm.hdu.edu.cn',
            'referer': 'http://acm.hdu.edu.cn/'
        }
```
然后用类中的session POST一下，即模拟了登录。这个session已经登录了。
`        r = self.session.post(url, data=data, headers=headers)`

### 4.2 submit模块
用submit模块来模拟提交代码。
手动提交并且抓包
![提交抓包][img5]
String Parameters是url中的代码，从而得到提交页面的url是
http://acm.hdu.edu.cn/submit.php?action=submit
需要模拟的data为Form Data中的几个数据，其中problemid是题目编号，language是提交的编译器种类，usercode是提交的代码
还需要模拟一个headers(这里我没有试过不加这个headers会怎么样，可能不加也可以)Connect-Type
最后用类中的session POST即可

```python
    def submit(self, problemID, code, language=0):
        """
        提交
        :param problemID: 题号
        :param language: 提交语言
        0 - G++
        1 - GCC
        2 - C++
        3 - C
        4 - Pascal
        5 - Java
        6 - C#
        :param code: 需要提交的代码
        """
        url = 'http://acm.hdu.edu.cn/submit.php?action=submit'
        code = code.encode('utf-8').decode()
        data = {
            'check': '0',
            'problemid': str(problemID),
            'language': str(language),
            'usercode': code
        }
        headers = {
            'Connect-Type': 'application/x-www-form-urlencoded'
        }
        se = self.session
        print('submitting problem: ', problemID)
        r = self.session.post(url, data=data, headers=headers)
```
### 4.3 getdiscuss模块
此模块用于返回从一个问题discuss中获得的答案的list

写完了登陆和提交模块之后，接下来当然就是爬取答案的模块啦！
HDOJ的discuss答案较多，正确率也较高，更重要的是代码非常好爬取。
来看一下一个题的discuss页面的源码
![discuss页面][img6]
问题的url比较规律，为http://acm.hdu.edu.cn/discuss/problem/list.php?problemid= 加上题号，拼接一下即可
`url = 'http://acm.hdu.edu.cn/discuss/problem/list.php?problemid=%s' % problemID`
接下来要解析html获取问题题解的url，这里就要用到正则表达式和BeautifulSoup的html.parser解析器(这个解析器为python自带解析器)
解析出deep=0的留言url，保存到一个list中。
不用担心这个留言里面没有答案，当我们得到这题的留言url之后，我们访问这些url，然后搜索'#include'如果包含则判断为代码，否则无动作
```python
        soup = BeautifulSoup(r.text, 'html.parser')
        res = []
        res = soup.find_all('a', href=re.compile('\./post/reply\.php\?postid=\d+&messageid=1&deep=0'))
        for item in res:
            item = item['href']
            item = item[1:]     # 将点截掉
            solutionurls.append('http://acm.hdu.edu.cn/discuss/problem%s' % item)  # 拼接url

```
对于list中的item，先拼接成完整的url，然后访问。
我们看一下discuss界面的html源码。
![discuss源码][img7]
可以看到内容都被保存在`<pre id=disshow>`的tag中，我们find这个tag，然后获取它的text
对text尝试匹配'#include'，如果匹配到了，那么加入到最后的结果中
```python
        for url in solutionurls:
            r = self.session.get(url)
            soup = BeautifulSoup(r.text, 'html.parser')
            disshow = soup.find('pre', id='disshow').text

            diss = re.search(re.compile(r'#include.+', re.S), disshow)     # re.S 开启多行模式匹配###中间可能有空格什么的
            if diss:
                solutions.append(diss.group())
        return solutions
```
### 4.4 getsolved
实际上有了上面3个模块以后程序已经可以运行做题了。不过这样做题的正确率会比较低，因为仅仅只是爬取discuss的所有代码并且提交一遍。
如何提高正确率？那就是避免重复提交已经AC的题。这样，判断题目是否AC的模块就显得尤为重要。
判断是否AC我有两个思路
 1. 从用户的userstatus页面获取所有已经AC的题目，然后判断某个题目是否在这个list中
 2. 从用户的status页面获取当前最新题目动态
第一个看似比较简单，于是选择了第一个方法……
来看一下userstatus页面的html
![userstatus][img8]
可以看到有一大串js代码，我们可以分割出题号并且存在一个list中
```python
    def getsolved(self, username):
        """
        获得一个用户已经解决的问题的所有题号
        :return: 返回一个含有解决问题题号的list
        :param username: 用户id
        """
        url = 'http://acm.hdu.edu.cn/userstatus.php?user=%s' % username
        solved = []
        r = self.session.get(url)
        # 解析出含有所有已完成题目号的字符串solvedstr
        soup = BeautifulSoup(r.text, 'html.parser')
        result = soup.find('p', align='left')
        solvedstr = result.text.split(';')
        # 从solvedstr中解析出一个list，含有所有完成题目号码
        for item in solvedstr:
            if item:
                item = re.search(r'\d{4}', item)
                solved.append(item.group(0))
        return solved
```

做完前4个模块以后我运行了一下，AC了大概1100的题，正确率为50%左右。显然还需要爬取更多的题解才能登上排行榜第一页。

### 4.5 getbaidu
这个模块用于抓取百度搜索结果中csdn博客中的代码，这是由于很多ACMer把题解发在csdn博客，而且csdn的代码比较容易抓取。
如果追求更高的正确率可以在一些专门提供题解的网站进行定向抓取，这样获得的题解准确度会更高。
我们看一下百度搜索结果页面
![baidusearchhtml][img9]
可以看到当前页面的url为 https://www.baidu.com/s?wd=hdu%201001
%20表示空格，那么我们就可以推导出百度搜索某个关键字的url，用题号将上面url的1001替换即可。
然后新建一个requests的session'baidusession'添加headers，模拟baidu搜索
```python
        url = r'http://www.baidu.com/s?wd=hdu%20' + str(problemID)

        baidusession = requests.Session()
        headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/48.0.2564.116 Safari/537.36'}
        baidusession.headers.update(headers)
        r = baidusession.get(url)
```
此外百度的搜索结果url搜加了密，样式大致为 https://www.baidu.com/link?url=XXXXXXXXXXX
但是每个搜索结果下面都有一小段绿的不完整的url，我们可以通过这个来判断搜索结果是否为csdn博客。如果是，那么保存这个页面的url
```python
        soup = BeautifulSoup(r.text, 'html.parser')
        res = soup.find_all('a', attrs={'target': '_blank', 'class': 'c-showurl',
                                  'style': 'text-decoration:none;'})
        for item in res:
            if re.match('blog.csdn.net', item.text):
                solutionurls.append(item['href'])
```
有了博客的url之后，我们一一访问这些url。
看一下csdn博客的源码
![csdnhtml][img10]
可以看到代码保存在一个tag中，name="code" class="cpp"
我们用BeautifulSoup解析出这个tag，获得代码。
为了提高正确率，我们可以验证一下博客页面的头是否包含该题号
(因为在实际运行的过程中我发现有些题目百度不到正确结果，经常百度出来几个别的题目的题解，正确率大打折扣，后来添加了这个验证)
如果通过验证，则把代码加入list
```python
        for item in solutionurls:
            r = baidusession.get(item)

            soup = BeautifulSoup(r.text, 'html.parser')
            code = soup.find(attrs={'name': 'code', 'class': 'cpp'})
            if code:
                # 先验证博客标题，如果标题包含题号，则继续
                title = soup.find('span', class_='link_title')
                pos = (title.text).find(str(problemID))
                if pos == -1:     # 若果不包含题号，break
                    break
                solutions.append(code.text)

        print(problemID, 'solutions finded: ', len(solutions))
        return solutions
```
### 4.6 autorun
这个模块比较简单，封装了最后的运行方法。
仅限于getbaidu里面的题解，并没有把getdiscuss写进去(因为我之前已经运行过一次getdiscuss，后来就改成getbaidu的了)
里面还有一个判断语言的机制，就是在代码中查询是否有'iosteam'来判断是否是C++代码等等……
然而实际运行的时候好像并不能大幅提高正确率，并没有什么卵用。
如果有兴趣可以试试直接autorun一下可以做到什么程度。
```python
    def autorun(self, start=1000, end=5639, interval=5):

        language = 0
        for problemID in range(start, end):
            # 先判断是否已经ac
            if str(problemID) not in c.getsolved(user):
                print(problemID, 'is not AC, start solving it...')
                # 解决这道没有AC的题目
                answers = c.getbaidu(problemID)
                if answers:
                    for answer in answers:
                        if str(problemID) not in c.getsolved(user):
                            # 判断语言
                            if answer.find('iostream') != -1:
                                language=2
                            elif answer.find('cstdio') != -1:
                                language=2
                            elif answer.find('stdio.h') != -1:
                                language=0
                            else:
                                print('language=???')
                                continue
                            print('language=', language)
                            c.submit(problemID, answer, language=language)
                            time.sleep(interval)
                        else:
                            break
```
## 5. 最后……
### 5.1 改进方向
其实还有一些改进的方向，比如爬取status页面，返回实时的题目信息。
还有抓取题目详情页的信息，甚至转化成更美观的Markdown页面……
这样就可以实现在本地命令行上做题，不用去网站。
### 5.2 源码地址-GitHub
我将注释更详细的源码发在GitHub上，欢迎各种Fork
[源码地址-GitHub](https://github.com/HScarb/HDUACM_ACer.git)
### 5.3 ???
……好像也没啥好说的了。
这个OJ号
账号RunnerUp
密码runnerup
如果有兴趣的话可以用这个号继续刷题……

以上


[img1]: http://114.215.140.250/wp-content/uploads/2016/03/2016-03-08_235738.png
[img2]: http://114.215.140.250/wp-content/uploads/2016/03/2016-03-08_235948.png
[img3]: http://114.215.140.250/wp-content/uploads/2016/03/2016-03-09_111521.png
[img4]: http://114.215.140.250/wp-content/uploads/2016/03/2016-03-09_105637.png
[img5]: http://114.215.140.250/wp-content/uploads/2016/03/2016-03-09_112405.png
[img6]: http://114.215.140.250/wp-content/uploads/2016/03/2016-03-09_113744.png
[img7]: http://114.215.140.250/wp-content/uploads/2016/03/2016-03-09_121653.png
[img8]: http://114.215.140.250/wp-content/uploads/2016/03/2016-03-09_123937.png
[img9]: http://114.215.140.250/wp-content/uploads/2016/03/2016-03-09_224545.png
[img10]: http://114.215.140.250/wp-content/uploads/2016/03/2016-03-09_225150.png