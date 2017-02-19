---
author: kimmy
layout: post
title:  "Slicing List"
date:   2017-02-08 23:15:36 +08:00
categories: Python
tags:
- Python
- Notes
---

* content
{:toc}



## L [ i : j ]
returns a new list, containing the objects between i and j, exactly is [ i, j )

{% highlight python %}
n = len(L)
item = L[index]
seq = L[start:stop]
{% endhighlight %}

## L [start:stop:step]
Lists also support slice *steps*, which is the last item!

{% highlight python %}
seq = L[::2]  # step is 2
seq = L[::-3] # step is 3 and sequence is reversed
seq = L[1::2] # get every other item, starting with the second
{% endhighlight %}

## KEYNOTE
---
To evaluate the expression seq[start:stop:step],    
Python calls `seq.__getitem__(slice(start, stop, step))`
