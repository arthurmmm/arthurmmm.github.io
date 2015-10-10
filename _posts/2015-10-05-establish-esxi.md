---
layout: post
title:  '笔记本搭建ESXi服务器'
date:   2015-10-05 12:31 +0800
comments: true
imagedir: /images/2015-10-05/
category: ESXi
---

让闲置的笔记本发挥余热，搭建ESXi Server成为家庭服务器吧！

## 准备工作

**硬件：**  

* CPU: Intel i5 520 M
* 内存：4GB
* 网卡: Realtek 8118/8168

**安装介质：**  

* 4GB U盘一枚  

**ESXi版本：**  

* ESXi 5.1 ISO 镜像（VMware Hypervisor）

## 为什么不安装更新的ESXi 5.5或者6.0?

其实一开始是打算安装6.0的，然而在安装过程中碰到了许多问题...  

**【安装时报错`No compatible network adapter found`】**

解决方案参考[这篇文章](http://bbs.51cto.com/thread-1165256-1-1.html) ，简单来讲：  

* 从[这个网站](https://app.yinxiang.com/OutboundRedirect.action?dest=https%3A%2F%2Fvibsdepot.v-front.de%2Fwiki%2Findex.php%2FList_of_currently_available_ESXi_packages%23NIC_drivers) 下载对应的网卡驱动
* 下载并使用[ESXi-Customizer-v2.7.2](http://pan.baidu.com/s/1eQ2f822) 将网卡驱动写入ISO镜像  

**【安装时提示内存不足4GB】**

坑爹啊（╯‵□′）╯︵┴─┴~~  
虽然说这台笔记本号称是有'4G内存'，然而却被集显吃掉了部分内存，实际只有3.8G。  
**安装ESXi 5.5及以上版本是严格要求有4G以上可用内存的！！**  

这个是硬伤..有尝试过更换一条8G内存，然而并不兼容..安装过程中花屏了过不去只能放弃(’╥ω╥‘)  

## 下载ESX 5.1

首先登录到[VMware官网](http://www.vmware.com/cn) ，点击`下载`->`所有下载、驱动程序和工具` 进入到搜索页面  
或者直接点击[这个链接](https://my.vmware.com/cn/web/vmware/downloads)  
在该页面搜索'hypervisor 5.1'就可以找到对应的ISO镜像啦！  
　  
　*PS:  
　你需要注册一个VMWARE帐号来获得免费使用的licence  
　由于现在的最新版本是ESXi 6.x，直接点`下载`->`vSphere Hypervisor`是找不到5.x的版本的！*  

## 使用UltraISO制作安装U盘

UltraISO可以直接[百度下载](https://www.baidu.com/s?wd=ultraiso)  
**注意：ESX 5.1自带了很多网卡驱动（比如常见的Realtek 8118/8168），你不需要再自制ISO了，如果按照6.0的方式自制ISO反而会报`No compatible network adapter found`的错误！**  

在UltraISO中选择`文件`->`打开`来载入ISO镜像，随后选择`启动`->`写入硬盘镜像`来制作安装U盘。*记得要把`硬盘驱动器`改为你的U盘哦！*  

## 安装ESXi系统  

将做好的U盘插入笔记本，开机时长按<F12>进入boot menu，选择U盘引导进入安装向导  
向导部分还是很简单的，一路根据提示选择，可以参考[这篇文章](http://www.osyunwei.com/archives/6586.html)  
安装完成后又遇到了一个问题：  

**【系统无法正常引导，仅提示boot up failed！】** 

一开始怀疑是主板不兼容，然而如果真是不兼容的话应该在安装过程中就失败了。  
在查阅各路论坛后发现是ESXi引导的问题，ESXi默认采用GPT分区，必须在BIOS中使用UEFI模式才能引导，这台老爷机可没有这配置...  
最后的解决方案如下：

* 在一开始那个`Loading ESXi installer`的界面键入Shift+O  
* 下方会出现一个输入框，并且已经自动键入了`runweasel`指令  
* 在`runweasel`后添加`formatwithmbr`，即最后输入框中的指令变成`runweasel formatwithmbr`，回车继续安装  

这次安装完成后就可以进入ESXi系统啦！ \(^o^)/  
　  
　*PS:顺手查了一下[GPT和MBR的区别](http://fyzx.ankang.gov.cn/Article/Class26/201408/1484.html)*

## 安装OpenSUSE 13.2  

装完Hypervisor开始部署VM~
首先使用vShpere client连接到ESXi服务器，创建VM，分配CPU、内存等信息。  
随后把VM开机，载入client端上的ISO，再用`CTRL+ALT+INSERT`重启虚机即可进入安装界面，一路NEXT...  
如果需要使用SSH连接虚机，可以在安装时直接打开SSH选项  
[![]({{site.baseurl}}{{page.imagedir}}00.JPG)]({{site.baseurl}}{{page.imagedir}}00.JPG)


