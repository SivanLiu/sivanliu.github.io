---

title: Android 动态更换 Launcher 中的 icon

date:  2018-1-7 11:42:21

tags:  Android icon

top: 27

---

大家有木有想过，在平时开过程中，需要冬天切换 Launcher icon, 这个功能是怎么实现的呢？下面来分析下该功能的实现原理

### 原理：
1.Manifest 中 activity 组件 activity-alias 组件：

* 对于 Android，所有的 activity 都是一个组件，可以对每个组件进行管理。 
通过 android.intent.action.MAIN 可以指定程序的入口。

```xml
//指定应用最先启动的 Activity
<action android:name="android.intent.action.MAIN" />

// 决定应用是否在桌面（Launcher）上显示
<category android:name="android.intent.category.LAUNCHER" />
```

* 通过 activity-alias 属性，可以创建应用多个不同的入口

```xml
<activity-alias 
                android:enabled=["true" | "false"]
                android:exported=["true" | "false"]
                android:icon="drawable resource"
                android:label="string resource"
                android:name="string"
                android:permission="string"
                android:targetActivity="string" >
    <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
</activity-alias>
```
其中大部分属性与 <Activity> 标签的属性一致，大概解释下：

|属性名称|类型|作用|
|---|---|---|
|android:enabled|boolean |是否开启别名设置，默认值为 true|
|android:exported|boolean|是否支持其他应用通过这个别名访问目标 Activity，默认值为 true|
|android:icon|drawable|Launcher 显示的图标|
|android:label|string|Launcher 显示的标签|
|android:name|string|Activity 别名|
|android:permission|权限|别名的使用约束|
|android:targetActivity|string|指定别名能够启动的目标 Activity|

android.intent.action.MAIN 表示别名设置是整个应用的入口，应用启动时第一个创建的就是这个 Activity；
android.intent.category.LAUNCHER 表示别名设置将出现在桌面 Launcher 应用上；
默认的，需要将 android:enabled 属性设为 false，否则运行时将会在桌面上出现两个相同功能但不同显示的应用图标和名称。

2.PackageManager 类进行启用/禁用组件；

Android apk 的 icon 是相对于整个包而言的，既然是修改 apk 包，那么系统有木有管理整个包的类呢？幸运的是，系统给我们提供了 PackageManager 类，通过 PM 可以实现对所有的系统组件进行管理。

```java
// 设置某组件是否可以启动
public abstract void setComponentEnabledSetting(ComponentName componentName,
            int newState, int flags);

// 获取某组件启动与否的状态      
public abstract int getComponentEnabledSetting(ComponentName componentName);
```

### 实现：
基本上就这些点儿了，下面附上 Demo 效果：

![](https://github.com/SivanLiu/sivanliu.github.io/blob/hexo_src/source/_posts/changeIcon.gif?raw=true)


Demo 源码地址：
<http://download.csdn.net/download/lyg468088/10192798>

### 网购类应用更新 Launcher icon 方案:
每当大型购物日来临，许多购物类应用都会修改，新增节日类元素， 比如京东、淘宝、唯品会、支付宝，小编刚才把他们都反编译看看，发现该类应用并没有采用上述方式更新图标，而是在每次节假日来临前都会发布一个版本，更新了图标，并不是像很多童鞋在博客中说到他们都用这种方式来动态更新 icon，估计他们并没有去反编译尝试看看？因此误导了很多初学者认为都是采用此类方式动态替换 Launcher icon，所以这里就带出了一个问题：为什么不采用这种方式呢？小编认为大概有如下原因：

* 由于在禁用组件后的刷新效率受到 ROM 影响，影响用户体验；
* 在正在切换 icon 的时候，马上点击该应用会提示说未安装该应用；
* 升级版本中 activity-alias 的 name 需要与之前保持一致；



