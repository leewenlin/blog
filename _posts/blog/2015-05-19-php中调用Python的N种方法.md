---
layout: post
title: php中调用Python的N种方法
modified:
categories: 
excerpt: 
tags: [php,python]
comments: true
share: true
---

- 方法一:system()
{% highlight php %}
system("python hehe.py");
{% endhighlight %}

- 方法二:popen()
{% highlight php %}
$ret = popen("python hehe.py","r"); 
$read=''; 
while(!feof($ret)){ 
$read .= fread($ret, 512); 
} 
echo $read;
{% endhighlight %}

- 方法三:exec()
{% highlight php %}
echo exec("python hehe.py");
{% endhighlight %}

- 方法四:``
{% highlight php %}
echo `python hehe.py`;
{% endhighlight %}




