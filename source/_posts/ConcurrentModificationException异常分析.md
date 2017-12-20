---

title: Java fast-fail 和 fast-safe 机制

date:  2017-12-20 23:42:21

tags:  java

top: 24

---

在平时开发过程中，相信大家在使用 ArrayList、HashMap、TreeMap 等容器类的时候，肯定碰到过 java.util.ConcurrentModificationException 异常，今天来捋一捋该异常到底是什么原因导致的，该如何规避它们！

提到 ConcurrentModificationException，大家应该都会想到 fail-fast 和 fail-safe，两者都是 Iterator 对ConcurrentModification 的处理机制，区别在于 fail-fast 会立即抛出 ConcurrentModificationException，而 fail-safe 并不会触发异常但是会导致数据的不一致性。

### fail-fast 机制
**Demo:**

```java
private static class FirstThread extends Thread {
        public void run() {
            Iterator<String> iterator = list.iterator();
            while (iterator.hasNext()) {
                String s = iterator.next();
                System.out.println("FirstThread iterator :" + s);
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }


    private static class SecondThread extends Thread {
        public void run() {
            int i = 0;
            while (i < 5) {
                System.out.println("SecondThread run：" + String.valueOf(i));
                if (i == 2) {
                    list.remove(String.valueOf(i));
                }
                i++;
            }
        }
    }
    
public class Main {
    private static List<String> list = new ArrayList<>();
    public static void main(String[] args) {
        for (int i = 0; i < 8; i++) {
            list.add(String.valueOf(i));
        }
        
        new FirstThread().start();
        new SecondThread().start();
        }
    }
```

**结果抛出异常**:

```java
FirstThread iterator :0
SecondThread run：0
SecondThread run：1
SecondThread run：2
SecondThread run：3
SecondThread run：4
Exception in thread "Thread-0" java.util.ConcurrentModificationException
	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:901)
	at java.util.ArrayList$Itr.next(ArrayList.java:851)
	at com.company.Main$FirstThread.run(Main.java:213)
```

**分析：**

从上述示例可以看出，fail-fast 产生的原因就在于 FristSecond 在对 list 进行迭代时，同时 SecondThread 也对该 list 进行了修改，此时 Iterator 就会抛出 ConcurrentModificationException 异，从而产生 fail-fast。

看看抛出异常的源码：

```java
final void checkForComodification() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
}
```
其中 modCount 是当前 List 大小，expectedModCount 是Iterator 创建时的大小，每次调用 next()、remove() 时都会调用 checkForComodification，该方法主要就是检测 modCount == expectedModCount ? 若不等则抛出ConcurrentModificationException 异常，从而产生fail-fast机制。所以要弄清楚 fail-fast 机制就必须要用弄明白什么时候会发生 modCount != expectedModCount。

在 ArrayList 源码中 modCount 和 expectedModCount 是这样定义的：

```java
int expectedModCount = ArrayList.this.modCount;
protected transient int modCount = 0;
```
expectedModCount 的值就是最初的 modCount 值，不会改变的，modCount 是 AbstractList 中定义的一个全局变量，在 ArrayList 中凡是执行了 add、remove、clear(只要涉及到改变了 ArrayList 元素的个数的方法)方法都会导致 modCount 值的改变。

在上述示例中，FirstThread 在遍历 list 的某个时候（expectedModCount = modCount = N），SecondThread 线程启动，此时 SecondThread 删掉了一个元素，现在 modCount 的值发生改变（modCount + 1 = N + 1）。FirstThread 继续遍历执行 next 方法时，调用 checkForComodification 方法发现expectedModCount  = N  ，而 modCount = N + 1，两者不等，就抛出 ConcurrentModificationException 异常，从而产生了 fail-fast 机制。

**解决方法：**使用 CopyOnWriteArrayList 来替换 ArrayList。
原因：
1.CopyOnWriterArrayList 无论是从数据结构、定义都和ArrayList 一样。它和 ArrayList 一样，同样是实现 List 接口，底层使用数组实现。在方法上也包含 add、remove、clear、iterator等方法。

2.CopyOnWriterArrayList 不会产生ConcurrentModificationException 异常，即不会产生 fail-fast机制。

### fail-safe 机制
fail-safe 机制中任何对集合结构的修改都会在一个复制的集合上进行，因此不会抛出 ConcurrentModificationException。但fail-safe机制存在两个问题：
1. 复制集合会产生大量的无效对象，开销大；
2. 无法保证读取的数据是目前原始数据结构中的数据

### fail-fast 与 fail-safe 对比

|| 抛出 CME 异常 | 复制对象 | 耗内存 | 举例 |
|:---:|:---:|:---:|:---:|:---:|
|fail-fast|是|否|否|HashMap,Vector,ArrayList,HashSet|
|fail-safe|否|是|是|CopyOnWriteArrayList, ConcurrentHashMap|

**如有不正之处，还望多多指教！**


