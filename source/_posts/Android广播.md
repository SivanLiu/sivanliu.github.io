---

title: 聊一聊 Android 广播

date:  2017-10-16 20:42:21

tags:  Android

top: 16

---

  Android 开发过程中，广播随处可见，那广播到底是个什么玩意儿呢？官网是这么描述的：
  ```java
  Android apps can send or receive broadcast messages from the Android system and other Android apps, similar to the publish-subscribe design pattern. 
  ```
 应用可以发送或者接收系统或者其他应用的广播消息，类似于发布——订阅者模式。 


1. 普通广播: 通过 Context.sendBroadcast() 方式来异步发送，广播发出去之后，所有的广播接收者几乎是同一时间收到消息的。他们之间没有先后顺序可言，而且这种广播是没法被截断的
- 1. 注册/反注册：
在 Mainfest 中注册： 
```java
<receiver android:name="cm.android.receiver.MyBroadcastReceiver">
    <intent-filter>
        <action android:name="cm.android.broadcast" />
    </intent-filter>
</receiver>
```

在代码中注册：
```java
MyBroadcastReceiver myBroadcastReceiver = new MyBroadcastReceiver();
registerReceiver(myBroadcastReceiver, new IntentFilter("cm.android.broadcast"));
```

反注册：在 Manifest 中注册的广播无法反注册，只有在代码中注册才可以反注册。
```java
unregisterReceiver(myBroadcastReceiver);
```

- 2. 发送：
```java
send.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                final Intent intent = new Intent();
                intent.setAction("cm.android.broadcast");
                sendBroadcast(intent);
            }
        });
```

- 3. 接收：
```java
public class MyBroadcastReceiver extends BroadcastReceiver {
    private static final String action = "cm.android.broadcast";

    @Override
    public void onReceive(Context context, Intent intent) {
        if (action.equals(intent.getAction())) {
            Log.e("onReceive", " intent : " + intent);
        }
    }
}
```


2. 有序广播: 有序广播通过 Context.sendOrderedBroadcast() 来发送，串行处理；广播发出之后各应用根据应用的优先级依次接收广播，优先级高的应用接收广播之后修改的数据也可以传递给后来的接受者，优先级高的应用也可以调用 abrotbroascast 方法停止该广播的向下传播，优先级靠应用的 android:prioioty 属性控制，该值的取值区间为 -1000 到 1000，值越大优先级越高。当广播接收器接收到广播后，也可以用abortBroadcast() 函数来让系统拦截下来该广播，并将该广播丢弃，使该广播不再传送到别的广播接收器接收

- 1. 注册：
```java
<receiver android:name="cm.android.receiver.OrderBroadcastReceiver$FirstBroadcast">
            <intent-filter android:priority="3000">
                <action android:name="cm.android.order.broadcast" />
            </intent-filter>
        </receiver>

        <receiver android:name="cm.android.receiver.OrderBroadcastReceiver$MiddleBroadcast">
            <intent-filter android:priority="2000">
                <action android:name="cm.android.order.broadcast" />
            </intent-filter>
        </receiver>

        <receiver android:name="cm.android.receiver.OrderBroadcastReceiver$LastBroadcast">
            <intent-filter android:priority="1000">
                <action android:name="cm.android.order.broadcast" />
            </intent-filter>
        </receiver>
```

- 2. 发送：
```java
send.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                final Intent intent = new Intent();
                intent.setAction("cm.android.order.broadcast");
                sendOrderedBroadcast(intent, null);
            }
        });
```

- 3. 拦截：
```java
abortBroadcast();
```

- 4. 终止：
```java
send.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                final Intent intent = new Intent();
                intent.setAction("cm.android.order.broadcast");
                sendOrderedBroadcast(intent, null, new OrderBroadcastReceiver.LastBroadcast(), 
                new Handler(), 0, null, null);
            }
        });
```

3. Sticky 广播：通过 Context.sendStickyBroadcast() 发送，以该方式发送的广播会一直存在于系统的消息容器里面，等待对应的处理器去处理，如果暂时没有处理器处理这个消息则一直在消息容器里面处于等待状态，sticky 广播的 Receiver 如果被销毁，那么下次重建时会自动接收到消息数据。  
发送 Sticky 广播需要权限：
```java
<uses-permission android:name="android.permission.BROADCAST_STICKY" />
```  
Sticky 广播只保留最后一条广播，并且一直保留下去，这样即使已经有广播接收器处理了该广播，当再有匹配的广播接收器被注册时，此广播仍会被接收。如果你只想处理一遍该广播，可以通过removeStickyBroadcast() 函数来实现，其注册和发送过程与普通广播类似，不做冗余介绍了。

4. 广播的安全性：
广播应用非常广泛，也比较简单，但其安全性问题不容忽视。第三者很容易通过反编译获取到我们应用中的广播，然后频繁的向你的App中发送广播，导致你的应用崩溃，那么如何避免应用中注册的广播响应其他应用发送的广播呢？

- 私有广播接收器设置exported='false'，并且不配置intent-filter。(私有广播接收器依然能接收到同UID的广播)
```java
<receiver android:name=".PrivateReceiver" android:exported="false" />
```

- 对接收来的广播进行验证;

- 内部 app 之间的广播使用 protectionLevel='signature' 验证;

- 返回结果时需注意接收 app 是否会泄露信息;

- 发送的广播包含敏感信息时需指定广播接收器，使用显示意图或者 setPackage(String packageName);

- sticky broadcast粘性广播中不应包含敏感信息；

- 有序广播建议设置接收权限，避免恶意应用设置高优先级抢收此广播后并拦截该广播。