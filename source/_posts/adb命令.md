---

title: 常用的 adb 命令

date:  2017-9-27 21:42:21

tags:  Android

top: 16

---

### 本篇博客就来记录下平日工作里经常使用的 adb 命令

#### 1.电量分析
```java
adb shell dumpsys batterystats --reset 重置电池数据收集

adb shell dumpsys batterystats --enable full-wake-history 开始统计电量

adb bugreport > bugreport.txt 导出电量使用数据
```

#### 2.app_process 启动特权进程
- 创建或使用一个已有的 Android 应用程序项目, 增加一个包含有 main 方法的类, 将该类编译成 class, dx 转换成 dex, 将 dex 文件 push 到 /data/local/tmp 中，再到 /data/local/tmp 中执行 app_process
```java
javac -source 1.8 -target 1.8 /Users/yafeng/process/Hello.java
dx --dex --output=/Users/yafeng/process/Hello.dex  /Users/yafeng/process/Hello.class
```
- 将 dex 文件 push 到 /data/local/tmp 中:
```java
adb push /Users/yafeng/process/Hello.dex /data/local/tmp 
```
- 使用 adb shell 在手机上执行对应的 app_process 命令启动进程:
```java
adb shell app_process -Djava.class.path=Hello.dex /data/local/tmp Hello
```

#### 3.linux 系统上连接上 adb， 但提示没有权限执行命令，则可以入下操作:
- 进入到 sdk/platform-tools 目录中；
- 切换到 root 用户；
- 执行 ./adb kill-server 即可， 然后再执行 ./adb start-server 即可；


#### 4.查看 apk 的具体信息：
```java
adb shell dumpsys package <packagename> 获得手机里面某个apk的应用信息、版本信息
aapt dump badging xxx.apk 查看apk信息
aapt list -a test.apk 获取apk中的资源属性的值
aapt d badging apkName aapt解析apk
adb shell pm list packages 获取apk包名
adb shell pm path packageName 获取apk的路径
adb shell dumpsys dbinfo [packagename] 查看应用数据库
adb shell rm -f /data/system/password.key 删除锁屏密码
```

#### 5.设备相关：
```java
adb shell cat /proc/cpuinfo 查看CPU架构
adb shell getprop 获取 ROM 中的所有属性
adb reboot recovery 重启到Recovery模式
adb reboot bootloader 重启到fastboot模式
adb shell am start -a android.settings.APPLICATION_DEVELOPMENT_SETTINGS 打开开发者页面(vivio: adb shell am start -n com.android.settings/com.vivo.settings.DevelpmentSettingsActivity2)
adb shell dumpsys window w | grep \/ | grep name= | cut -d = -f 3 | cut -d \) -f 1 显示当前activity
adb shell dumpsys activity top |grep ACTIVITY 查看栈顶Activity
adb shell "dumpsys activity | grep Focuse" 查看当前应用的包名
adb shell am start -a android.settings.APPLICATION_DETAILS_SETTINGS -d package:packageName  打开应用详情
adb shell am start -a android.intent.action.CALL -d tel:number 拨打电话
adb shell am start -a android.intent.action.VIEW -d  url 打开指定网页
adb shell ls /sdcard/logs/abc_*.txt  | tr -d '\r' | xargs -n1 -I {} adb pull {} d:/abc 正则拷贝多个文件到电脑
adb shell cd dirName 进入某个目录
adb shell ls -a 查看当前目录下的文件
adb pull <remote> <local> 从手机复制文件出来
adb push <local> <remote> 向手机发送文件
adb shell rename path/oldfilename path/newfilename 重命名文件
adb shell rm -r <folder> 删除文件夹及其下面所有文件
adb shell mv path/file newpath/file 移动文件
adb shell cat <file> 查看文件内容
adb shell makedir <filename> 创建一个文件夹
adb shell cat /data/misc/wifi/*.conf 查看wifi密码
```

#### 6.导出堆栈信息：
```java
adb shell "cp /data/anr/traces.txt /sdcard/"
adb pull /sdcard/traces.txt .
```

#### 7.进程：
```java
adb shell ps |grep fj 查看进程名包含 fj 的进程
adb shell am force-stop packageName 杀掉应用进程
```

#### 8.截屏、录屏
```java
adb shell screencap -p /sdcard/test.png
adb shell screenrecord /sdcard/test.mp4 
需要停止时按 Ctrl-C，默认录制时间和最长录制时间都是 180 秒
```

#### 9.日志：
```java
adb logcat -c 清除log缓存
adb logcat -v time | grep ActivityManager 查看Activity跳转
adb logcat -v time | grep AndroidRuntime 查看崩溃信息
adb logcat -v time | grep "D\/Dalvik" 查看Dalvik信息，比如GC
adb logcat -v time | grep "I\/art" 查看art信息，比如GC
adb bugreport 查看bug报告
```

#### 10.查看签名信息：
```java
keytool -printcert -file xxx.cert 查看 cert 信息
keytool -printcert -jarFile xxx.apk 查看 apk 签名信息
```

### 11.手动删除 app， 只针对 root 过的手机:
- adb root后adb remount
- data/app 用户程序安装的目录，有删除权限。
- data/data 存放应用程序的数据
- adb pull /data/system/packages.xml 和 /data/system/packages.list 到本地电脑，删除这两个文件中的xxx.apk相关的项，并且删除与我们包名相同的项。

#### 12.端口占用解决方法：
- 5037 为 adb 默认端口，若 5037 端口被占用，查看占用端口的进程 PID：
```java
netstat -aon|findstr 5037
```
- 通过 PID 查看所有进程:
```java
tasklist /fi "PID eq pidNumber"
```
- 杀死占用端口的进程:
```java
taskkill /pid pidNumber /f
```

大概就以上这么多，后续会不断补充！欢迎指导！