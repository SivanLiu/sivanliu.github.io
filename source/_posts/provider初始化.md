---

title: Android 多个 provider 初始化顺序

date:  2017-12-16 12:42:21

tags:  Android

top: 22

---

**缘起**：
    利用 provider 来初始化你的 provider， 这个相信大家已经不太陌生了，下面简要说下。

## 1. provider 初始化 library:

    在日常开发过程中， 经常会遇到 Library 都需要传入 Context 参数以完成初始化，此时这个 Context 参数一般会从 Application 对象的 onCreate 方法中获取。于是，很多 library 都会提供一个 init 方法，在Application Object中完成调用.

```java
public class MyApplication extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        Library.init(this);
    }
}
```

至此， 相信大家可以看到依赖第三方 library 的时候，你必须得写一个自定义的 Application， 这样的用户体验可不太好。从 Android Debug Database， Firebase 中受到启发， 我们可以在 library 中创建一个 ContentProvider 来， 然后在 Manifest 中完成必要的注册信息， 在应用 Application 起来的时候会自动去初始化 library 中定义的 ContentProvider，现在就可以从 ContentProvider 中的 onCreate 里面得到 context 来初始化自己的 library。

```java
public class FirstProvider extends ContentProvider {
    @Override
    public void attachInfo(Context context, ProviderInfo info) {
        super.attachInfo(context, info);
    }
    @Override
    public boolean onCreate() {
        context = getContext();
        // 这里可以初始化你需要的代码
        return true;
    }
    ......
}
``` 
这里顺便说一下Application， ContentProvider 初始化的顺序：

```java
Application.attachBaseContext(super before) -> Application.attachBaseContext(super after) -> ContentProvider.attachInfo(super before) -> ContentProvider.onCreate() -> ContentProvider.attachInfo(super after) -> Application.onCreate(super before) -> Application.onCreate(super after)
```

## 2. 多个 provider 初始化顺序：
 
1. 如果 library 中有业务需要用到多个自定义 provider，如 A、B、C， 但是自定义用来初始化的 provider 为D， 由于初始化流程在 provider D 中开始的， 那么 A、B、C 就必须在 D 之后起来才行，那么D ->A、B、C 顺序怎么来定义呢？也就是如何保证 D 最先初始化。

2. 为了验证多个 provider 初始化顺序，demo 中定义了三个 provider， 具体如下：
   
   ```java
   public class FirstProvider extends ContentProvider {
    private static final String TAG = "FirstProvider";

    @Override
    public void attachInfo(Context context, ProviderInfo info) {
        Log.e(TAG, "attachInfo 0 info = " + info.authority);
        super.attachInfo(context, info);
        Log.e(TAG, "attachInfo 1 info = " + info.authority);
    }

    @Override
    public boolean onCreate() {
        Log.e(TAG, "onCreate  pid = " + Process.myPid());
        return true;
    }
    ......
}
   ```
如此定义了 FirstProvider，SecondProvider 以及 ThirdProvider，然后在 Manifest 中注册：

```java
        <provider
                android:authorities="com.sivan.FirstProvider"
                android:exported="false"
                android:name=".FirstProvider" />

        <provider
                android:authorities="com.sivan.ThirdProvider"
                android:exported="false"
                android:name=".ThirdProvider" />

        <provider
                android:authorities="com.sivan.SecondProvider"
                android:exported="false"
                android:name=".SecondProvider" />
```

最后得到的日志信息如下：
```java
MyApplication: attachBaseContext 0 pid = 8257
MyApplication: attachBaseContext 1 pid = 8257
ThirdProvider: attachInfo 0 com.sivan.ThirdProvider
ThirdProvider: onCreate pid =8257
ThirdProvider: attachInfo 1 com.sivan.ThirdProvider
SecondProvider: attachInfo 0 info = com.sivan.SecondProvider
SecondProvider: onCreate  pid = 8257
SecondProvider: attachInfo 1 info = com.sivan.SecondProvider
FirstProvider: attachInfo 0 info = com.sivan.FirstProvider
FirstProvider: onCreate  pid = 8257
FirstProvider: attachInfo 1 info = com.sivan.FirstProvider
MyApplication: onCreate 0 pid = 8257
MyApplication: onCreate 1 pid = 8257
```
可以看出启动顺序为：Third -> Second -> First
从中基本上可以猜测出，provider 初始化的顺序是根据 android:name 相关，为了验证该结论， 将 FirstProvider name 修改为 TFirstProvider， 再看下日志信息：

```java
MyApplication: attachBaseContext 0 pid = 9508
MyApplication: attachBaseContext 1 pid = 9508
ThirdProvider: attachInfo 0 info = com.sivan.FirstProvider
ThirdProvider: onCreate  pid = 9508
ThirdProvider: attachInfo 1 info = com.sivan.FirstProvider
TFirstProvider: attachInfo 0 com.sivan.ThirdProvider
TFirstProvider: onCreate pid =9508
TFirstProvider: attachInfo 1 com.sivan.ThirdProvider
SecondProvider: attachInfo 0 info = com.sivan.SecondProvider
SecondProvider: onCreate  pid = 9508
SecondProvider: attachInfo 1 info = com.sivan.SecondProvider
MyApplication: onCreate 0 pid = 9508
MyApplication: onCreate 1 pid = 9508
```
可以看出初始化流程：ThirdProvider -> TFirstProvider -> SecondProvider

所以我们需要根据 android:name 来调整自定义用来初始化的 provider， 要保证 D 在 A、B、C 之前来初始化。



