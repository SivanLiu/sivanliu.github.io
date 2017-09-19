---

title: 聊聊微信、qq数据库解密

date:  2017-9-19 23:42:21

tags:  Android

top: 15

---

### 最近看到网上有人专门针对如何破解qq、微信数据库，自己也想试试，于是就有了此文，主要讲下如何读取聊天信息, 前提是需要一部 root 过的手机。

#### 概述：
- 微信数据库加密方式；
- 微信多账户数据读取；
- 微信好友消息以及群消息；
- QQ 消息加密方式；
- QQ 讨论组消息；
- QQ 群消息；
- QQ 多账户数据读取；

#### 1.微信数据库加密方式：
微信数据库加密方式已经有高人研究过了，就不细说了，主要是：
```java
String password = [1, 7]md5(imei + uin)
```
其中 imei 是手机的 IMEI 号， 在拨号界面输入 *#06# 即可看到，有手机有两个 IMEI 号，选择第一个 IMEI 号，关于双 IMEI 号如何读取，请自行 google，不用重复造轮子了；uin 是微信为每个用户分配的 id， 可以在如下路径找到
```java
\data\data\com.tencent.mm\shared_prefs\auth_info_key_prefs.xml
```
其中具体的信息如下：
```xml
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<map>
    <boolean name="key_auth_info_prefs_created" value="true" />
    <int name="key_auth_update_version" value="637864511" />
    <int name="_auth_uin" value="*********" />
    <string name="_auth_key">66678f560e18cc8a4388814ed45f1f285193badd1e6f9ca59d63315ac31f9514f5fa66f06992022745c08a66a9ec83e723d34d3e884531e5b0ca2f034510d28c013cef877dccfdea7d718f2aec08c0d</string>
    <string name="server_id">99021802ad4d5a45ecc81c99443700</string>
</map>
```
_auth_uin 的值 value 即为 uin 值，在代码中可以通过解析 xml 来读取 uin 的值；然后将 imei 和 uin 相加计算 md5 的结果，取前 7 位的值即为密码。

#### 2.微信多账户数据读取：
当多个账户在同一个手机上使用时，就会存在多个账户的数据库文件，但是在 auth_info_key_prefs 中只有当前用户的 uin 的值，无法获取到历史的 uin 数据，不过仔细分析下数据库目录的父目录：
```md5
/data/data/com.tencent.mm/MicroMsg/c8201023c5887895fabf25da90b5fab8/EnMicroMsg.db
```
c8201023c5887895fabf25da90b5fab8 可以看出来是个 md5 值，这个已经有人研究过了（https://blog.slinuxer.com/2015/10/%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A9%E8%AE%B0%E5%BD%95）
```java
文件名 = md5(mm + uin)
```
经验证，确实如此，知道了文件名的构成，可以逆推 uin 的值，只不过是个计算的问题。那么话说回来，只有逆推才能得出 uin 的值吗？逆推有些繁琐，那么有没有简单的方法得到登录过的所有的 uin 的值呢? 答案是肯定的，经过一番搜索，把每个目录都翻遍了，终于发现了 app_brand_global_sp.xml.xml 文件，打开一看，大吃一惊，原来就是你，其中的具体信息如下：
```xml
<map>
    <set name = "uin_set">
        <string>-123847323</string>
        <string>-923847323</string>
    </set>
</map>
```
找的就是你，这个只是最新版微信测试的，之前的版本未测试。
有了所有用户的 uin, imei 号也知道了，还有什么不知道呢，剩下的就是撸代码了。

#### 3.微信好友消息及群消息读取：
既然密码知道了，那么就开始解密数据库了，工具 sqlcipher.exe，这个已经有前人编译好的，直接下载用，打开数据库，聊天消息尽收眼底。
- 3.1 好友消息与群消息的区别主要是 message 表中 talker 字段是否包括 @chatroom 字样：
- 3.2 好友，包括群聊昵称、id 在 rcontact 表中可以查到；
- 3.3 在 message 表中可以提取以下几个字段，基本上就可知道，聊天的内容了：
```java
type 消息类型 | isSend (2 : 系统 1 : 本人发送, 0 :　聊天对象发送) | createTime : 时间戳 | talker : 聊天对象 | content : 聊天内容 |
```
#### 4.QQ消息加密方式
qq 数据库并没有全库加密，而是对每条消息进行加密, 加密的方式就是 uin 的 md5 值， uin 就是用户的 qq 号码，每个聊天都会对应一个表，表名命名规则：
```java
mr_friend_New 好友聊天
mr_discussionuin_New 讨论组聊天
mr_troop_New 群聊
```
每张表里面只需要提取以下字段就够了：
```java
selfuin 本人的 uin | senderuin 发消息的 uin | msgtype 消息类型 | msgData 消息内容 | issend 是否为本人发送 | time 时间 
```
其中大部分字段都是加密的，需要对应的 uin 的 md5 来解密。

#### 5.QQ讨论组消息：
所有的讨论组信息都在 discussionInfo 表中，可以找到讨论组的 uin, 然后根据 uin 构造表名称，根据表名称去查对应的表信息

#### 6.群消息；
所有群的相关信息都记录在 TroopInfo 中，而且在 TroopStatisticsInfo 中记录了群的总数，这样一来就可以构造出每个群聊的表名称，根据表名称去查找对应的表信息即可

#### 7.多账户数据读取；
相比微信来说，QQ 多账户记录读取简单很多，在 QQ 数据库目录中的所有的账号数据库都在此，一目了然，并无什么特别的技巧。

基本上就这么些东西了，借助前人的研究做起来还是比较顺畅，如果真要整理出每条聊天消息，还是需要花些功夫的。

好了，就写到这里了，有什么不明白的可以留言！谢谢！

