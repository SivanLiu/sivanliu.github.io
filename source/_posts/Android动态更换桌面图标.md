---

title: Android 动态更换 Launcher 中的 icon

date:  2018-1-6 11:42:21

tags:  Android icon

top: 27

---

每当双 11、12 来临之际，Android 手机 Launcher 中的淘宝、天猫图标就会变成双 11、12 主题的图标。大家有木有想过，这个功能是怎么实现的呢？下面来分析下该功能的实现原理

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

基本上就这些点儿了，下面附上 Demo 效果：

![效果](https://github.com/SivanLiu/sivanliu.github.io/blob/hexo_src/source/_posts/changeIcon.gif)




