---
layout: post
title:  Change Swapfile Size
date:   2017-02-06 23:47:14 +08:00
categories: Notes
tags:
- Raspbian
- Linux
- Vim
---

* content
{:toc}



# Change Swapfile Size

**Problem**
---------------------------------------------
RASPBERRY PI 3 MODEL B has 1GB RAM, and error occurs when compiling YCM `sudo python ./install.py`.

> ...  
> ...  
> 'BoostParts/CMakeFiles/BoostParts.dir/libs/python/src/numeric.cpp.o' failed
> make[3]: *** [BoostParts/CMakeFiles/BoostParts.dir/libs/python/src/numeric.cpp.o] Error 4
> CMakeFiles/Makefile2:75: recipe for target 'BoostParts/CMakeFiles/BoostParts.dir/all' failed
> CMakeFiles/Makefile2:209: recipe for target 'ycm/CMakeFiles/ycm_support_libs.dir/rule' failed  
> ...  
> ...


**Solution1**
---------------------------------------------
 add more swap memory  
```bash     
free -h # check how much memory available, similar with 'top' command in some ways
sudo swapon -s #
```

>Raspian uses a script dphys-swapfile to manage swap. The standard image includes this turned on by default. The configuration files is located at **/etc/dphys-swapfile**

```bash  
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
```    

**Solution2**
---------------------------------------------
 setting the YCM_CORES environment variable to 1  
```bash
YCM_CORES=1 ./install.py --clang-completer
```
