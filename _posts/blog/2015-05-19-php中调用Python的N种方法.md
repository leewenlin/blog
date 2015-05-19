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
- 
{% highlight php %}
$ret = popen("python hehe.py","r"); 
$read=''; 
while(!feof($ret)){ 
$read .= fread($ret, 512); 
} 
echo $read;
{% endhighlight %}

{% highlight php %}
system("python hehe.py");
{% endhighlight %}

{% highlight php %}
echo `python hehe.py`;
{% endhighlight %}

{% highlight php %}
echo exec("python hehe.py");
{% endhighlight %}


