---
layout: post
title:  '配置和安装OpenSUSE 13.2虚拟机'
imagedir:  /images/2015-10-06-00/
date:   2015-10-06 19:53 +0800
comments: true
category: Linux
---

这里记录一下在ESXi上部署OpenSUSE 13.2虚拟机的过程。   
关于搭建ESXi的细节可以参考[这里]({{ site.baseurl}}/2015/10/establish-esxi/)  

## 安装OpenSUSE  

首先使用vShpere client连接到ESXi服务器，创建VM，分配CPU、内存等信息。  
随后把VM开机，载入client端上的ISO，再用`CTRL+ALT+INSERT`重启虚机即可进入安装界面，一路NEXT...  
如果需要使用SSH连接虚机，可以在安装时直接打开SSH选项  
![]({{site.baseurl}}{{page.imagedir}}00.JPG)

## 配置zypper国内源：

参考我的另外一篇文章[快速为OpenSUSE配置国内源]({{ site.baseurl}}/2015/10/replace-zypper-repo/)  

## 安装GIT：  

由于我的各类配置都同步在GIT上，所以这里先安装GIT以便导入配置，也记录一下安装和配置GIT的过程：  

1] 使用zypper安装GIT程序包：  

```
$ zypper install git
```
  
2] 配置GIT用户名和邮箱：  

```
$ git config --global user.name "arthurmmm"
$ git config --global user.email "arthur_mao@live.com"
```   

3] 生成ssh密钥：   

```
$ ssh-keygen -t rsa -C "arthur_mao@live.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
//不用更改目录和密码，敲3次回车即可
```  

4] 登陆`github`，展开右上角箭头找到并进入`setting`页面  
![]({{site.baseurl}}{{page.imagedir}}01.JPG)  

5] 进入右侧`SSH Keys`页面，点击`Add SSH key`，把`/root/.ssh/id_rsa.pub`中的内容复制到`Key`中再点击`Add key`   

6] 测试  

```
$ ssh git@github.com
Warning: Permanently added the RSA host key for IP address '192.30.252.128' to the list of known hosts.
PTY allocation request failed on channel 0
Hi arthurmmm! You've successfully authenticated, but GitHub does not provide shell access.
Connection to github.com closed.
```

7] 导入github上的配置文件  

*PS：这些配置文件仅适用于我自家的环境哟*

```
$ ./setup.sh
>> Creating NETSTORAGE mount point...
>> Installing user profile...
>> Installing system profile...
>> Finish!

** Please reboot to activate all settings ...
```