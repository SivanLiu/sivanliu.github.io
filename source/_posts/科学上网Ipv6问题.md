---

title: VPS 设置 IPv6/IPv4 模式科学上网

date:  2017-12-31 11:42:21

tags:  工具

top: 26

---

想必那些经常科学访问 google 的友友们，肯定会遇到这样的问题：
* 为什么访问 google 页面偶尔会出现验证码输入页面？
* 为什么可以 ping 通 google，但是用命令行（w3m，curl，lynx，links）浏览网页超级慢，甚至打不开？

### 问题的根源：

针对上述问题的原因基本上是由于 google 对你的 ip 地址做了 ipv4 或者 ipv6 过滤，为什么会做过滤呢？必须的吗？当然不是啦，只有 google 检测到你的网段发生异常的时候才会进行过滤，比如说你的 vps，ipv4 或者 ipv6 网段被用来做爬虫爬取 google 内容，但是 google 一般不会把你的 ipv4 和 ipv6 地址全部屏蔽。

### 解决方案：
##### 确定 Ipv4 还是 Ipv6 被封锁了：

1. 找到访问 google 的 ip 地址，当你通过 ss 使用 google 时，出现了验证码页面，上面会有相应的 ip 地址，此时你就可以知道是 ipv4 还是 ipv6 被封掉了。
2. 可以在 vps 上用 curl 命令测试下：
1）Ipv4 测试：

```shell
[root@154255 ~]# curl -4 -s -w 'Testing Website Response Time for :%{url_effective}\n\nLookup Time:\t\t%{time_namelookup}\nConnect Time:\t\t%{time_connect}\nPre-transfer Time:\t%{time_pretransfer}\nStart-transfer Time:\t%{time_starttransfer}\n\nTotal Time:\t\t%{time_total}\n' -o /dev/null http://www.google.com
Testing Website Response Time for :http://www.google.com/
```
结果：

```shell
Lookup Time:		0.030
Connect Time:		0.032
Pre-transfer Time:	0.032
Start-transfer Time:	0.086

Total Time:		0.086
```
2）Ipv6 测试：

```shell
[root@154255 ~]# curl -s --ipv6 -w 'Testing Website Response Time for :%{url_effective}\n\nLookup Time:\t\t%{time_namelookup}\nConnect Time:\t\t%{time_connect}\nPre-transfer Time:\t%{time_pretransfer}\nStart-transfer Time:\t%{time_starttransfer}\n\nTotal Time:\t\t%{time_total}\n' -o /dev/null http://www.google.com
Testing Website Response Time for :http://www.google.com/
```

结果：

```shell
Lookup Time:            0.061
Connect Time:           63.063
Pre-transfer Time:      63.063
Start-transfer Time:    63.116

Total Time:             63.116
```

从上述结果可以看到，ipv6 连接时间很长，说明 ipv6 不正常，已经被封锁了。

##### 设置指定模式（Ipv4 or Ipv6）访问：
1. 强制 Ipv4 访问，一般情况下，Vps 会同时支持 Ipv4 和 Ipv6，此时你只需设置禁用 Ipv6 访问模式即可：
1）编辑/etc/sysctl.conf，在文件末尾加入：

```shell
# disable ipv6
net.ipv6.conf.all.disable_ipv6=1
net.ipv6.conf.default.disable_ipv6=1
net.ipv6.conf.lo.disable_ipv6=1
```
然后执行如下命令，让修改过的模式生效：

```shell
sysctl -p
```

2. 强制使用 Ipv6 模式访问：
如果是 IPv4 地址被封，那么只需要强制 VPS 使用 Ipv6 访问 Google 就行了。
在 VPS 的 hosts 中指定 Google 的Ipv6 <https://raw.githubusercontent.com/lennylxx/ipv6-hosts/master/hosts> 地址即可。

```shell
2404:6800:4008:c00::71 google.com
2404:6800:4008:c02::63 www.google.com
2404:6800:4008:c04::66 aboutme.google.com #plus.l.google.com
2404:6800:4008:c00::54 accounts.google.com #accounts.l.google.com
2404:6800:4008:c03::71 admin.google.com
```
最后重新启动下 ss 服务，应该就可以正常访问 google 了。

### ssh 解决 SSH 超时断开问题：
平时通过终端连接服务器时，当长时间不操作，服务器就会自动断开连接，也就是 SSH 超时断开，下次操作需要重新连接，感觉很麻烦，总结一下解决此问题的方法。

1. 修改 sshd_config 文件:

1）打开 /etc/ssh/sshd_config 配置文件，找到ClientAliveCountMax（单位为分钟）修改你想要的值，执行service sshd reload：

```shell
ClientAliveInterval 30
ClientAliveCountMax 3
```
* ClientAliveInterval：指服务器端向客户端发送检测是否活跃消息的时间间隔。默认是 0，不发送。将其修改为 30，表示每 30 分发送一次消息。
* ClientAliveCountMax：表示服务器发出请求后，服务器可接受的客户端未响应次数，达到该值后，连接会自动断开。默认是 0，使用其默认值即可。

2）重新启动 sshd 服务：

```shell
service sshd restart
```

2. 修改/etc/profile 配置文件，增加 TMOUT：

```shell
TMOUT=1800
```
使修改过的配置生效：

```shell
source /etc/profile
```
如此一来，30 分钟没操作就自动 LOGOUT（SSH超时断开）

常用的就上述两种方法！







 




