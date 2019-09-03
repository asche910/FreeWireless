# FreeWireless
绕过校园网、公共热点认证，从而实现免费上网


文章显示若存在问题，可[此处查看](https://www.cnblogs.com/asche/p/11449351.html)



## 前言

很多时候，当流量不够用时，看着周围那么多热点又连不上，是不是有点心痒痒呢？那么有没有办法不需要要通过这些热点的认证即可上网呢？当然是有的。
另外在此强调一点，本教程仅用于**学习测试**用途，请勿用于不正当的途径！


## 大体思路

连上那些公共热点，往往都能成功，但是也往往还需要进一步的认证才能够上网。没有认证的时，当我们访问http的网站时，我们的请求会被拦截并跳转至热点（下文就以校园网代表热点了）的登陆认证页面，如图所示。

![image](http://asche.top/asapi/forward?url=https://img2018.cnblogs.com/blog/1470456/201909/1470456-20190902203433234-683910181.png)


但是，如果直接访问https的话，就是响应超时了，原因应该是https的一些加密导致的吧。


但是，我们发现，某些udp的端口还是开放的，毕竟由于他们的特殊作用。先看看下图

![image](http://asche.top/asapi/forward?url=https://img2018.cnblogs.com/blog/1470456/201909/1470456-20190902203811058-1658638662.png)

上面那个是dns解析的，没有认证的情况下可以成功的解析到结果，说明dns的端口53是开放的。

而下面的ping命令却是超时，这就说明了ping的icmp协议被拦截下来了。因此我们就在dns端口53上下功夫。

后面要做的就是在外部服务器上搭建相应的环境，然后在本地也搭建下环境，这样我们就可以将我们的网络请求通过53端口发送到外部服务器上，外部服务器解析之后请求目标服务器，再将结果返回到本地，总体流程大致即这样。

## 详细步骤

### 服务端

这里以我的ubuntu为例，默认gcc环境等依赖安装完成。

ssh登陆服务器后,下载vpnserver

``` 
wget https://github.com/SoftEtherVPN/SoftEtherVPN_Stable/releases/download/v4.28-9669-beta/softether-vpnserver-v4.28-9669-beta-2018.09.11-linux-x64-64bit.tar.gz
```

解压

``` 
tar -zxvf softether-vpnserver-v4.28-9669-beta-2018.09.11-linux-x64-64bit.tar.gz
```

cd到对应目录，make编译

``` 
make
``` 

此时提示输入，一路输入1 即yes就是了。

然后启动该服务
```
./vpnserver start
```

紧接着
```
./vpncmd
```

输入1之后两次回车，后面会提示输入密码，这个密码就当时连接vpnserver的密码吧。

注意：这时如果端口被占用的话，可能会报错，就导致没到密码那一步提前结束了，还是建议为本应用留着那几个端口吧，不行的话可以手动更改目录下的配置文件修改端口。

到这里，服务端的安装完毕。


### 本地配置

首先下载 [SoftEtherVPN](https://github.com/SoftEtherVPN/SoftEtherVPN_Stable/releases/download/v4.28-9669-beta/softether-vpnserver_vpnbridge-v4.28-9669-beta-2018.09.11-windows-x86_x64-intel.exe) ，按我的理解，这个应该是为我们待会的openvpn生成配置文件准备。
下载完成后安装，选择最下面的一个安装
![image](https://img2018.cnblogs.com/blog/1470456/201909/1470456-20190902210622166-1774959716.png)

安装完成后如下图

![image](http://asche.top/asapi/forward?url=https://img2018.cnblogs.com/blog/1470456/201909/1470456-20190902210743223-1221953514.png)

点击新设置，填写相关信息。名称可以随意填一个，主机名就填你之前的服务器地址，下面端口默认端口443（之前服务端启动监听的），右下角密码就是之前说的那个vpnserver密码。然后确定。之后再连接

![image](http://asche.top/asapi/forward?url=https://img2018.cnblogs.com/blog/1470456/201909/1470456-20190902211051780-1327321873.png)

管理虚拟HUB

![image](http://asche.top/asapi/forward?url=https://img2018.cnblogs.com/blog/1470456/201909/1470456-20190902211419325-1851202746.png)

管理用户

![image](http://asche.top/asapi/forward?url=https://img2018.cnblogs.com/blog/1470456/201909/1470456-20190902211502641-158336568.png)


再新建

![image](http://asche.top/asapi/forward?url=https://img2018.cnblogs.com/blog/1470456/201909/1470456-20190902211540186-1445131527.png)

其中用户名和密码待会在openvpn中登陆要用到的。

确认之后会弹框显示成功，紧接着

![image](http://asche.top/asapi/forward?url=https://img2018.cnblogs.com/blog/1470456/201909/1470456-20190902211747096-102358010.png)


![image](http://asche.top/asapi/forward?url=https://img2018.cnblogs.com/blog/1470456/201909/1470456-20190902211811648-1333986281.png)

再确认。之后回到管理界面，点击下图中的

![image](http://asche.top/asapi/forward?url=https://img2018.cnblogs.com/blog/1470456/201909/1470456-20190902211858698-742773328.png)

然后先填写端口，通常53用的比较多，其次67、68这些。然后点击下面那个，生成配置文件之后解压出来。

![image](http://asche.top/asapi/forward?url=https://img2018.cnblogs.com/blog/1470456/201909/1470456-20190902211923569-1447131565.png)


提取出那个含有remote字眼的文件。


然后还要下载[openvpn](https://github.com/asche910/FreeWireless/blob/master/openvpn-install-2.4.7-I607-Win10.exe?raw=true)文件。安装完成之后打开会在任务栏中出现相应的图标，鼠标右键选择导入配置文件，再选中之前解压出来的那个配置文件。然后该图标右键会多了一个选项，选中之后再点击连接。随后会出现用户名密码的认证框，输入之前添加的用户即可。
成功之后如下图所示。

![image](http://asche.top/asapi/forward?url=https://img2018.cnblogs.com/blog/1470456/201909/1470456-20190902213347401-660788147.png)

这时应该就可以未认证免费上网了。速度的话基本取决于你服务器的带宽。比如我的是1Mb速度，实际也就是100多k吧。如果嫌慢可以选择国外的大带宽vps服务。


### 跨平台三端

刚刚windows上的已经介绍了，还有Android和ios端。
Android端的安装包，可以在我[github](https://github.com/asche910/FreeWireless)中下载，安装完成之后，把之前windows上准备好的配置文件拷贝到手机，然后手机上进入文件管理器，找到文件，打开方式选择openvpn即可。

Ios的为[openvpn-connect](https://apps.apple.com/us/app/openvpn-connect/id590379981) 国区商店自然是下载不到的，自备美区账号下载吧。下载完成之后，可以先将配置文件通过qq传送到手机，然后打开方式同样的选择openvpn-connect即可。


## 一些问题

一路上也遇到了不少问题，也简单说下吧。
openvpn那里老是连接不上，报错信息大概是握手失败，多久之后重试之类的。我服务器安全组之前记得放通所有端口了，所以没太在意，后来发现还是这里的问题。原因在于我放通的是TCP端口，UDP还是被我关了，所以打开之后问题解决。

还有就是中午连接成功，也正常上网，可是大概过了几十分钟后，突然又上不了。此时我将端口从53换为67、68皆可以上网，只是速度极慢，看服务器日志显示这些信息

![image](http://asche.top/asapi/forward?url=https://img2018.cnblogs.com/blog/1470456/201909/1470456-20190902215640986-1242248590.png)

感觉是校园网那边检测到dns端口流量异常，因此直接切断了我的这些连接。不过此时nslookup命令同样能工作。

后来到今天晚上，我又试了试53端口，发现又可以成功连接了，而且接下来的几个小时都没有出现啥问题。查看服务器日志，发现还是有不少像之前那样的连接被删除的信息，这里我就有点迷了！

了解到softether还有加密与网络 - VPN over ICMP / DNS的功能，我也试了试，发现打开之后生成的配置文件与之前的比较没啥差别，具体我也仍在探索中。

最后，本文github连接 [https://github.com/asche910/FreeWireless](https://github.com/asche910/FreeWireless) 

同时，参考文章：

https://blog.csdn.net/qq_35422558/article/details/84316063

https://www.bennythink.com/udp53.html



