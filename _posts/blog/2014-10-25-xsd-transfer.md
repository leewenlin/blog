---
layout: post
title: 翻译XSD文件小脚本
modified:
categories: blog
excerpt: 
tags: [python, xsd]
comments: true
share: true
---

工作需要写的一个翻译xsd文件的小脚本，第一次写python，其实挺简单的，就是正则加上调google的翻译，后来怕google翻译不全面，又给加上了一个调有道的。

{% highlight python %}
import httplib,os,re,json,urllib,urllib2
from urllib import urlencode
def out(text):
    p = re.compile(r'","')
    m = p.split(text)
    return m[0][4:]

def gtrans(text):
    text=urlencode({'text':text})
    h=httplib.HTTP('translate.google.cn')
    h.putrequest('GET', '/translate_a/t?client=t&hl=zh-CN&sl=en&tl=zh-CN&ie=UTF-8&oe=UTF-8&'+text)
    h.endheaders()
    h.getreply()
    f = h.getfile()
    lines = f.readlines()
    f.close()
    return out(lines[0])

def ytrans(text):
    data={}
    formj={"type":"EN2ZH_CN","i":text,"doctype":"json","xmlVersion":"1.6","keyfrom":"fanyi.web","ue":"UTF-8","typoResult":"true"}
    data=urllib.urlencode(formj)
    req=urllib2.Request('http://fanyi.youdao.com/translate?smartresult=dict&smartresult=rule&smartresult=ugc',data)
    response=urllib2.urlopen(req)
    page= response.read()
    data=json.loads(page)
    return data['translateResult'][0][0]['tgt'].encode('UTF-8')

def reg(text):
    pattern = re.compile(r"(.*(?<=>))?(Note:\s)?(?P[^<>]*[\w]+[a-z]+[^<>]*)((?=<).*)?")
    match = pattern.match(text.strip())
    if match:
        hehe = match.group("hehe")
        return addtrans(text,hehe)
    else:
        return text

def addtrans(text,bt):
    return text.replace(bt,"***"+bt+"***"+gtrans(bt))
    '''if gtrans(bt)==ytrans(bt):
        tr=gtrans(bt)
    else:
        tr=gtrans(bt)+"[["+ytrans(bt)+"]]"
    return text.replace(bt,"***"+bt+"***"+tr)'''

def ffilter(text):
    pattern = re.compile(r"[a-i]|[A-I]")
    return pattern.match(text)

def makefiles():
    path="E:\\DESK\\fanyi"
    newpath=path+"\\google"
    l = os.listdir(path)
    os.chdir(path)
    for f in l:
        if(f[-4:]!=".xsd"):
            continue
        fh=open(f)
        print f+" start~"
        newf=open(newpath+"\\"+f,"w")
        newf.truncate()
        for line in fh.readlines():
            '''print "."'''
            if line!= None:
                newf.writelines(reg(line))
        newf.close()
        print f+" finish~"
    print "all finish"

if __name__=='__main__':
    makefiles()
{% endhighlight %}
