---
author: kimmy
layout: post
title:  "How Do I ENJOY My Whole 30 Hours"
date:   2017-02-22 21:37:00 +08:00
categories: Linux
tags:
- Notes
- DeepLearning
- Linux
---


* content
{:toc}



## The Original Reason

Here comes a brand-new, `I mean the inside :-)`  workstation, and absolutely a POWERFUL one.
+ Take a look at its plain selfie, clean and pretty.
  ![COOl](http://wx2.sinaimg.cn/mw690/9f1c5669gy1fcy9jxsybsj21kw23u7wi.jpg)
+ But we really should appreciate the great wisdom.
+ ![still COOL](http://wx3.sinaimg.cn/mw690/9f1c5669gy1fcy9rf69uij20o90c7wge.jpg)

## 2017/2/20 22:00 —— 2017/2/21 21:00

I got stuck in the infinite login loop on Ubuntu16.04 when attempted to install NVIDIA Graphic Driver.<br>
During glommy trails, difficulity itself just makes me more itch to overcome.<br>
Except for several times of reloading OS, that's the most desperated minutes. :0

### Foremost， Offical Documentation

+ [CUDA Installation Guide](http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#axzz4ZJkxrvVt)
<br><br>
+ [Installing TensorFlow](https://www.tensorflow.org/install/)
<br><br>
+ [Caffe Installation Instructions](http://caffe.berkeleyvision.org/installation.html)
<br>

Just don't refer to others' blogs or notes at first, <br>
but might be better when you actually crashed into some problems, <br>
then search solutions in stackoverflow, github, specific forum and so on.

### Build OpenCV

NOTE<br>
+ Uninstall the `OpenCV 3.1` packages within Python
+ If compiled with cuda8.0, download `OpenCV 3.1` from [here](https://github.com/opencv/opencv), because the official `OpenCV 3.1` does not support CUDA 8.0.)

{% highlight bash %}
mkdir build
cd build/
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local -D WITH_TBB=ON \
      -D WITH_V4L=ON -D WITH_QT=ON -D WITH_OPENGL=ON \
      -D WITH_CUBLAS=ON -DCUDA_NVCC_FLAGS="-D_FORCE_INLINES" ..   # ".." indicates the makefile dir
make -j $(($(nproc) + 1))   # (48 + 1) COOL
{% endhighlight %}

### Build Caffe
{% highlight bash %}

# install boost dependencies
cd boost_1_63_0/
# add executive permission
find . -name "*.sh" -exec chmod a+x "{}" \;
sudo bash ./bootstrap.sh



# ./bjam make boost
bjam address-model=64 cxxflags=-fPIC --with-thread stage --with-date_time stage        --with-chrono stage --with-system stage --with-filesystem stage --with-regex stage --with-python stage include=/home/frisch/anaconda2/include/python2.7


# compile caffe, plz :)
cmake ..
make all
make install
make runtest

{% endhighlight %}

### Bad Login Loop

`CTRL-ALT-F1` login tty1

{% highlight bash %}
# Uninstall any previous drivers
sudo apt-get remove nvidia-*

# Uninstall the drivers from the .run file
sudo apt-get autoremove
sudo nvidia-uninstall
{% endhighlight %}

This should remove the login loop, so now reboot and login normally.

`CTRL-ALT-F1` login tty1, if use lightdm

{% highlight bash %}
sudo service lightdm stop

sudo ./NVIDIA-Linux-x86_64-375.39.run \
            -no-x-check -no-nouveau-check -no-opengl-files
# `-no-x-check`: shutdown X service
# `-no-nouveau-check`: forbidden nouveau
# `-no-opengl-files`: only build driver

#***************#
# DO NOT reboot.#
#***************#
sudo service lightdm restart
{% endhighlight %}

VERY VERY IMPORTANT:<br><br>
`CTRL-ALT-F7` to login IMMEDIATELY after installation

## Notes
---

### 1.  check hardware info

{% highlight bash %}

#---------------------------------------
#
# /proc
#

# cpu/memory information
cat /proc/cpuinfo
cat /proc/meminfo

# linux/kernel information
cat /proc/version

# scsi/sata devices
cat /proc/scsi/scsi

# partitions
cat /proc/partitions

#---------------------------------------
#
# cpu
#

# summary infomation about cpu and processing units
lscpu

# in detail
cat /proc/cpuinfo

#---------------------------------------
#
# RAM
#

# check the amount of used, free and total
free

# in detail
cat /proc/meminfo

#---------------------------------------
#
# disks
#

# list block devices
lsblk

# list out the partition information
# mainly used to **partition** the hard drive
fdisk -l

# report **filesystem** disk space usage
df

# find disk usage of **files** and directories
du

#---------------------------------------
#
# network cards
#

# show info about wired network
lspci | grep -i 'eth'

# list out all network interfaces
ifconfig

# specify an interface
ethtool eth0

#---------------------------------------
#
# others
#

# pci buses and details about the devices connected to them
# here just list the graphics card info
lspci | grep -i vga

# scsi( small computer system interface)/sata devices
lsscsi

{% endhighlight %}

### 2. find error via the `log` and `error` files

{% highlight bash %}
cat /tmp/cuda_install_XXXX.log
cat ~/.xsession-errors
{% endhighlight %}

### 3. change image source to domestic sites
    whenever reinstall the OS or packages manager like `conda` and `pip`

### 4. about X server in Linux

{% highlight bash %}
sudo service lightdm stop    # start
sudp dpkg-reconfigure lightdm
{% endhighlight %}

### 5. use `ssh` better than physical transfer via trans memory disk

{% highlight bash %}
mount /dev/sdc1 /mnt/disk_name
umount /mnt/disk_name

{% endhighlight %}

### 6. change the host name

{% highlight bash %}
sudo vim /etc/hostname
sudo vim /etc/hosts    # edit the second line
{% endhighlight %}

### 7. switch user
switch to root user<br>
`sudo su`
switch to normal user<br>
+ `su username`
+ `exit`
+ CTRL + D


### 8. check gpu support asked by nvidia
see complete [installation guide](http://docs.nvidia.com/cuda/cuda-installation-guide-linux/#axzz4VZnqTJ2A).

#### `lspci | grep -i nvidia`
+ verify that the GPU is CUDA-capable.
+ `update-pciids` if see nothing.

#### `lsb_release`
+ verify the version of Linux.
+ prints certain LSB (Linux Standard Base) and Distribution information.
+ ` -a` option displays all the information.
+ `uname -m && cat /etc/*release` is the same function.

#### `nvidia-smi`
+ is short for **system manmagement interface**.
+ `man nvidia-smi` would list the help information.
+  `persistence mode`
    + A flag that indicates whether persistence mode is enabled for the GPU.  Value is either "Enabled" or "Disabled".
    + When persistence mode is  enabled the  NVIDIA driver remains loaded even when no active clients,
        such as X11 or nvidia-smi, exist.
    + This minimizes the driver load latency associated with running dependent apps, such as CUDA programs.
