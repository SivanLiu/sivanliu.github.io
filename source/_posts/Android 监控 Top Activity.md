---

title: Android 获取 Top Activity 方案总结

date:  2017-12-23 8:42:21

tags:  Android

top: 25

---


在实际需求中，往往会碰到需要检测当前正在运行的应用的 packageName 或者获取 Top Activity 等等，现在就来捋一捋该需求的实现方案。

### 1. getRunningTasks:
5.0 以下版本可以正常使用，但 5.0 以上就不行了，Google 以泄露用户信息的理由禁用了该方法， getRunningTasks 将只返回应用自身和启动器相关信息

```java
public static boolean isTopActivy(String name, Context context) {
        if (context == null || name == null) {
            return false;
        }

        ActivityManager manager = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
        List<ActivityManager.RunningTaskInfo> taskInfos = manager.getRunningTasks(Integer.MAX_VALUE);
        String topActivityName = null;
        if (manager != null && taskInfos != null) {
            topActivityName = (taskInfos.get(0).topActivity).toString();
        }

        if (null == topActivityName) {
            return false;
        }

        return name.equals(name);
    }
```

### 2. getRunningAppProcesses
在 5.0 系统可正常使用，但到 5.1 系统上就不行，调用该方法只能返回应用本身的相关信息，而且官方也说了只是用于 “debugging and management user interfaces”

```java
public static boolean isFrontApp(Context context, String packageName) {
        ActivityManager mActivityManager = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);

        if (Build.VERSION.SDK_INT == Build.VERSION_CODES.LOLLIPOP) {
            List<ActivityManager.RunningAppProcessInfo> tasks = mActivityManager.getRunningAppProcesses();
            if (null != tasks && tasks.size() > 0) {
                return packageName.equals(tasks.get(0).processName);
            }
        }

        return false;
    }
```

### 3. ActivityLifecycleCallbacks
大家都知道在 Activity 的生命周期中，如果 Activity 调用了 onResume 方法，那么它在前台，如果 Activity 调用了onPause()，那么该 Activity 就到了后台。那么紧紧靠这两个就够了吗？当然不行，还需要考虑到 onStart() 和 onStop() 回调，为什么呢？因为 onStart() 和 onStop() 是 Activity 可见状态切换的回调；onResume() 和 onPause() 是 Activity 活动状态的回调。
如从 A -> B 的时候，跳转流程：
A.onPause() -> B.onStart() -> B.onResume() -> A.onStop()
基于此类特征，可以定义一个全局计数器，来记录前台页面的数量，在所有 Activity.onStart() 中计数器 +1，在所有 Activity.onStop() 中计数器 -1。计数器数目大于0，说明应用在前台；计数器数目等于 0，说明应用在后台。计数器从 1 变成 0，说明应用从前台进入后台；计数器从 0 变成 1，说明应用从后台进入前台。
还有另外两类情况：
1）没有配置 configChanges 和 screenOrientation，屏幕旋转造成 Activity 重启；
2）手动调用 Activity.recreate() 重启 Activity；

此类情形下，Activity 会完整的走完销毁流程，之后重走创建流程，这时候计数器逻辑会出现问题。因此添加一个缓冲计数器来处理。

```java
public class MyApplication extends Application {
    private ActivityLifecycleCallbacksImpl mActivityLifecycleCallbacksImpl;

    @Override
    public void onCreate() {
        super.onCreate();
        mActivityLifecycleCallbacksImpl = new ActivityLifecycleCallbacksImpl();
        this.registerActivityLifecycleCallbacks(mActivityLifecycleCallbacksImpl);
    }

    private class ActivityLifecycleCallbacksImpl implements ActivityLifecycleCallbacks {
        // 前台 Activity 的数目
        private int mFrontActivityCount = 0;
        // 缓冲计数器，记录 configChanges 的状态
        private int tmpCount = 0;

        @Override
        public void onActivityCreated(Activity activity, Bundle savedInstanceState) {

        }

        @Override
        public void onActivityStarted(Activity activity) {
            if (mFrontActivityCount <= 0) {
                // TODO 处理从后台恢复到前台的逻辑
            }
            
            if (tmpCount < 0) {
                tmpCount++;
            } else {
                mFrontActivityCount++;
            }
        }

        @Override
        public void onActivityResumed(Activity activity) {

        }

        @Override
        public void onActivityPaused(Activity activity) {

        }

        @Override
        public void onActivityStopped(Activity activity) {
            // 遇到 configChanges 情况，操作缓冲计数器
            if (activity.isChangingConfigurations()) {
                tmpCount--;
            } else {
                mFrontActivityCount--;
                if (mFrontActivityCount <= 0) {
                    // TODO 处理从前台进入到后台的逻辑
                }
            }
        }

        @Override
        public void onActivitySaveInstanceState(Activity activity, Bundle outState) {

        }

        @Override
        public void onActivityDestroyed(Activity activity) {

        }
    }
}
```

### 4. UsageStatsManager
利用 UsageStatsManager，并调用 queryUsageStats 方法来获得启动的历史记录，在这之前需要设置权限“Apps withusage access”。但是这个 queryUsageStats 只能查询一段时间内的使用状态，如果时间间隔较短，并且一段时间不使用手机，获得的列表就可能为空。

```xml
<uses-permission android:name="android.permission.PACKAGE_USAGE_STATS"
            tools:ignore="ProtectedPermissions" />
```

```java
public String getFrontPackageName(Context context) {
        String topPackageName = null;
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            UsageStatsManager mUsageStatsManager = (UsageStatsManager) context.getSystemService(Context.USAGE_STATS_SERVICE);
            long time = System.currentTimeMillis();
            // We get usage stats for the last 10 seconds
            List<UsageStats> stats = mUsageStatsManager.queryUsageStats(UsageStatsManager.INTERVAL_DAILY, time - 1000 * 10, time);
            // Sort the stats by the last time used
            if (stats != null) {
                SortedMap<Long, UsageStats> mySortedMap = new TreeMap<>();
                for (UsageStats usageStats : stats) {
                    mySortedMap.put(usageStats.getLastTimeUsed(), usageStats);
                }
                if (mySortedMap != null && !mySortedMap.isEmpty()) {
                    topPackageName = mySortedMap.get(mySortedMap.lastKey()).getPackageName();
                    return topPackageName;
                }
            }
        }
        return null;
    }
```

### 5. 辅助功能（AccessibilityService)
使用辅助功能监控当前 Active Window，然后利用回调函数 onAccessibilityEvent 过滤事件类型 TYPE_WINDOW_STATE_CHANGED 获知当前 window 改变，
通过调用 PackageManager.getActivityInfo() 获取当前应用的 packageName，兼容性比较好，但必须要求用户手动在手机的设置页面开启 Android's accessibility settings 才能收到 AccessibilityEvent事件，当用户在设置页面开启 AccessibilityService 的时候，有些第三方应用会弹出一个悬浮窗盖住屏幕，导致用户无法点击确认按钮。AccessibilityService 是被动触发的，无法主动获知当前应用的信息。

```java
public class MyAccessibilityService extends AccessibilityService {
        private static String mPackageName = null;

        @Override
        public void onAccessibilityEvent(AccessibilityEvent event) {
            //过滤下事件类型
            if (event.getEventType() == AccessibilityEvent.TYPE_WINDOW_STATE_CHANGED) {

                //TODO 可以根据需要做适当处理
                mPackageName = event.getPackageName().toString();
            }
        }

        @Override
        public void onInterrupt() {

        }
    }
```
服务注册：

```xml
        <service
                android:name=".MyAccessibilityService"
                android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE">

            <intent-filter>
                <action android:name="android.accessibilityservice.AccessibilityService"/>
            </intent-filter>

            <meta-data
                    android:name="android.accessibilityservice"
                    android:resource="@xml/accessibility_service_config"/>

        </service>
```

```xml
    <accessibility-service xmlns:android="http://schemas.android.com/apk/res/android"
            android:description="@string/accessibility_service_description"
            android:accessibilityEventTypes="typeAllMask"
            android:accessibilityFeedbackType="feedbackGeneric"
            android:notificationTimeout="100"
            android:accessibilityFlags="flagDefault"
            android:canRetrieveWindowContent="true"
            android:packageNames="cm.sivan.demo"
            android:settingsActivity="cm.sivan.demo.MainActivity"  />
```

### 6. 读取/proc目录下的信息
jaredrummler 的[作品](https://github.com/jaredrummler/AndroidProcesses)，被很多流行应用使用了，比如 ES File Explorer, Clean Master, Security Master, CM Launcher 3D, Virus Cleaner 等应用，但是在 Android 7.0 (Nougat)不好用了，目前 jaredrummler 已经废弃了该库，感兴趣的可以看看，还是有很多值得学习的地方。

```java
// Get a list of running apps
List<AndroidAppProcess> processes = AndroidProcesses.getRunningAppProcesses();

for (AndroidAppProcess process : processes) {
  // Get some information about the process
  String processName = process.name;

  Stat stat = process.stat();
  int pid = stat.getPid();
  int parentProcessId = stat.ppid();
  long startTime = stat.stime();
  int policy = stat.policy();
  char state = stat.state();

  Statm statm = process.statm();
  long totalSizeOfProcess = statm.getSize();
  long residentSetSize = statm.getResidentSetSize();

  PackageInfo packageInfo = process.getPackageInfo(context, 0);
  String appName = packageInfo.applicationInfo.loadLabel(pm).toString();
}
```
相信看到这里，对于想要获取前台应用包名或者 Top Activity 的需求，你一定会有相应的解决方案了，以后有更好的方法还会继续更新。

|方案| 权限要求 | 特点 |
|---|---|---|
|getRunningTasks|否|4.0 可判断其他应用是否位于前台，5.0以后被废弃（只能获取自己应用的信息）|
|getRunningAppProcesses|否|当p存在后台常驻的Service时失效|
|ActivityLifecycleCallbacks|否|简单、需要适配|
|UsageStatsManager|要|可判断其他应用是否位于前台，用户手动给予权限|
|AccessibilityService|要|可判断其他应用是否位于前台，用户手动打开辅助功能|
|读取/proc目录下的信息|否|可判断其他应用是否位于前台|

参考：
1. http://blog.takwolf.com/2017/06/29/android-application-foreground-and-background-switch-listener/index.html

**如有不正之处，还望批评指正！**





