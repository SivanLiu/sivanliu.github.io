---

title: Shadowsocks 搭建

date:  2017-9-19 22:42:21

tags:  工具

top: 13

---

### 由于诸多原因，使用中的好些 vpn 都被 GFW 干掉了，无奈只能自己动手搭建一个了，好处是再也不用四处搜索 vpn 了，唯一的缺点就是需要 RMB，没有付出，哪有回报，总是值得的。

#### 1.准备工作：
- 选择适合自己 VPS；
- 部署 shadowsocks 服务；
- 安装 shadowsocks 客户端；
- 开始 Internet 自由之旅；

#### 2.选择 VPS：
个人觉得选择 VPS 还是尽量选择国外比较有名的提供商吧，至少可以保证稳定性，价格也比较合理，甚至还比国内的便宜，如 Vultr、Digital Ocean、Linode、bandwagonhost、BudgetVM。对比了下，最终选择了 BudgetVM 最便宜的一个 $25：RAM(512), VSwap(256MB), Storage(50GB), Transfer(2TB)。这个配置对于搭建 ss 来说绰绰有余，而且流量也够用了。

#### 3.部署 ss 服务：
此过程为了方便，选择的是脚本一键安装 <https://teddysun.com/486.html>，很简单，什么配置、优化都帮你做好了，还支持自定义多用户配置，炒鸡方便。

### 4.安装 ss 客户端：
ss 客户端基本上都有现成的：
1）<https://sourceforge.net/projects/shadowsocksgui/files/dist/>  
2）<https://github.com/shadowsocks>  
选择适合自己系统的版本下载、安装  
我用的是 Debian，无奈装了个 linux 版本的不行，只有自己手动编译个 deb 包了，最后果然好使  
iphone: 可在 appstore 中下载相应的客户端， 如黑洞（全局代理），wingy（自动代理，需要切换到美国区下载）  
对于谷歌浏览器，可以安装 SwitchyOmega 插件进行设置，教程网上很多，按照操作即可

### 5.Internet 自由之旅：
如果按照以上操作还不能翻越强大的 GFW， 那最大的可能性客户端配置的有问题，可以重新配置试试。  
最后可以尽情地享受 Internet 自由之旅了！

