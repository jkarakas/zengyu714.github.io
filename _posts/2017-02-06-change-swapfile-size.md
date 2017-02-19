---
layout: post
title:  Change Swapfile Size
date:   2017-02-06 23:47:14 +08:00
categories: Linux
tags:
- Raspbian
- Linux
- Vim
---

* content
{:toc}


## Problem
---------------------------------------------
RASPBERRY PI 3 MODEL B has 1GB RAM, and error occurs when compiling YCM
`sudo python ./install.py`.

{% highlight bash %}
...

'BoostParts/CMakeFiles/BoostParts.dir/libs/python/src/numeric.cpp.o' failed
make[3]: *** [BoostParts/CMakeFiles/BoostParts.dir/libs/python/src/numeric.cpp.o] Error 4
CMakeFiles/Makefile2:75: recipe for target 'BoostParts/CMakeFiles/BoostParts.dir/all' failed
CMakeFiles/Makefile2:209: recipe for target 'ycm/CMakeFiles/ycm_support_libs.dir/rule' failed

...
{% endhighlight %}


## Solution 1
---------------------------------------------
 Add more swap memory

{% highlight bash %}
free -h         # check how much memory available, similar with 'top' command in some ways
sudo swapon -s  # Display swap usage summary by device
{% endhighlight %}


Raspian uses a configuration script located at `/etc/dphys-swapfile` to manage swap.<br>
The standard image includes this turned on by default.


{% highlight bash %}
sudo vim /etc/dphys-swapfile
#-------------------------------------
# CONF_SWAPSIZE=1024
# ------------------------------------

# ------------------------------------------------------------------
# stop and start the service that manages the swapfile own Rasbian
#
sudo /etc/init.d/dphys-swapfile stop
sudo /etc/init.d/dphys-swapfile start
#-------------------------------------------------------------------
free -m # verify the memory condition
{% endhighlight %}

## Solution 2
---------------------------------------------
 Setting the `YCM_CORES` environment variable to 1
{% highlight bash %}
YCM_CORES=1 ./install.py --clang-completer
{% endhighlight %}
