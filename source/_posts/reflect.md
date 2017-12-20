---

title: 浅谈 Java 反射

date:  2017-12-16 17:42:21

tags:  java

top: 23

---

    在程序中，我们经常会用到发射，但有木有想到过为什么要使用反射，其实现原理是什么，有哪些使用场景？

### 1. 反射原理
反射机制就是在程序运行状态中，对于任意的一个类，你都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种**动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。**

简明扼要一句话：**反射能够在程序运行时获取任意一个类的属性和方法**。

### 2. 如何使用反射
Java 反射是可以让我们在运行时获取类的函数、属性、父类、接口等 Class 内部信息的机制。通过反射还可以让我们在运行期实例化对象，调用方法，通过调用 get/set 方法获取变量的值，即使方法或属性是私有的也可以通过反射的形式调用。
#### 2.1 核心类：

| 代码描述 | 解释 |
| --- | --- |
|java.lang.Class | 类 |
|java.lang.reflect.Constructor | 构造方法 |
|java.lang.reflect.Field | 类的属性 |
|java.lang.reflect.Method | 类的方法 |
|java.lang.reflect.Modifier | 类的修饰符 |

其中 Modifier 取值范围如下： public, protected, private, abstract, static, final, transient, volatile, synchronized, native, strictfp, interface。

Constructor， Field， Method 这三个类都继承 AccessibleObject，该对象有一个非常重要的方法 setAccessible(boolean flag)，借助该方法，能直接调用非 Public 的属性与方法。


#### 2.2 核心方法：
* **成员属性**：

| 方法| 解释 |
| --- | --- |
|getFields() | 获得类的 public 类型的属性 |
|getDeclaredFields()| 获得类的所有属性 |
|getField(String name) | 获取类中 public 类型的特定属性 |
|getDeclaredField(String name) | 获取类的特定属性 |

* **成员方法**：

| 方法| 解释 |
| --- | --- |
| getMethods() | 获得类的特定方法 |
| getDeclaredMethods()| 获得类的所有方法 |
| getMethod(String name, Class[] parameterTypes) | 获得类的特定方法 |
| getDeclaredMethod(String name, Class[] parameterTypes) | 获得类的特定方法 |

* **构造方法**：

| 方法| 解释 |
| --- | --- |
| getConstructors() | 获得类的public类型的构造方法 |
| getDeclaredConstructors()| 获得类的所有构造方法 |
| getConstructor(Class[] parameterTypes) | 获得类中 public 类型的特定构造方法 |
| getDeclaredConstructor(Class[] params) | 获得类的特定构造方法 |

#### 2.3 反射类：
* getClass():

```java
    String str = "test";
    Class cls = str.getClass();
```
* getSuperClass():

```java
    String str = "test";
    Class cls = str.getClass().getSuperclass();
```

* Class.forName(String name):

```java
    try {
            String str = "test";
            Class cls = Class.forName("java.lang.String");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
```

* .class 属性:

```java
    String str = "test";
    Class cls = java.lang.String.class;
```

* 基本类型包装类的 TYPE 语法():

```java
    Class cls = Integer.TYPE;
```
#### 2.4 私有类以及类的私有方法或属性：

* 私有类：

```java
package com.company;

class Test {
    private int obj = 10;
    
    private Test() {

    }

    private int add(int x, int y) {
        return x + y;

    }
}

```

```java
        Class cls = Class.forName("com.company.Test");
        Constructor constructor = cls.getDeclaredConstructor();
        constructor.setAccessible(true);
        Object instance = constructor.newInstance();
```

* 私有方法：

```java
    Method method = cls.getDeclaredMethod("add", int.class, int.class);
    method.setAccessible(true);
    int sum = (int) method.invoke(instance, 1, 2);
```


* 私有属性：

```java
    Field field = cls.getDeclaredField("obj");
    field.setAccessible(true);
    int value = (int) field.get(instance);
```

#### 2.5 反射内部类：
内部类分 static 内部类与非 static 内部类，两者的反射方式有一定的区别：构造方法有所差异。

示例：

```java
class Test {
    private int obj = 10;

    private Test() {

    }

    private int add(int x, int y) {
        return x + y;

    }

    private class Inner {
        private int innerObj = 20;

        private Inner() {

        }

        private int sub(int x, int y) {
            return x - y;
        }
    }

    private static class staticInner {
        private String innerField = "StaticInner Field";
        private static String innerStaticField = "StaticInner static Field";

        private staticInner() {

        }

        private int Sub(int x, int y) {
            return x - y;
        }

        private static int staticSub(int x, int y) {
            return x - y;
        }

    }
}
```
将 staticInner 和 Inner 对应的构造器、字段、方法打印出来，可看到：

* static 内部类的默认构造函数： private com.company.Test$staticInner()

* 非 static 内部类的默认构造函数： private com.company.Test$Inner(com.company.Test)

由此可知，非静态内部类相对于静态内部类来说多了一个参数com.company.Test，也就是说非 static 内部类保持了外部类的引用。

##### 2.5.1 静态内部类：
静态内部类构造器、字段、方法调用：

```java
            Class staticInnerCls = Class.forName("com.company.Test$staticInner");
            Constructor staticInnerCons = staticInnerCls.getDeclaredConstructors()[0];
            staticInnerCons.setAccessible(true);
            Object staticInnerInstance = staticInnerCons.newInstance();

            Method staticInnerMethod = staticInnerCls.getDeclaredMethod("Sub", int.class, int.class);
            staticInnerMethod.setAccessible(true);
            int staticResult = (int) staticInnerMethod.invoke(staticInnerInstance, 4, 2);

            Method staticMethod = staticInnerCls.getDeclaredMethod("staticSub", int.class, int.class);
            staticMethod.setAccessible(true);
            int staticMethodResult = (int) staticMethod.invoke(staticInnerCls, 5, 2);

            Field staticField = staticInnerCls.getDeclaredField("innerStaticField");
            staticField.setAccessible(true);
            String staticValue = (String) staticField.get(staticInnerCls);

            Field staticInnerField = staticInnerCls.getDeclaredField("innerField");
            staticInnerField.setAccessible(true);
            String staticInnerValue = (String) staticInnerField.get(staticInnerInstance);

```

##### 2.5.2 非静态内部类：
反射构造器、字段、方法如下：

```java
            Class innerCls = Class.forName("com.company.Test$Inner");
            Constructor innerConstructor = innerCls.getDeclaredConstructors()[0];
            innerConstructor.setAccessible(true);
            Object inngerObj = innerConstructor.newInstance(instance);

            Method innerMethod = innerCls.getDeclaredMethod("sub", int.class, int.class);
            innerMethod.setAccessible(true);
            int innerResult = (int) innerMethod.invoke(inngerObj, 3, 2);

            Field innerField = innerCls.getDeclaredField("innerObj");
            innerField.setAccessible(true);
            int innerValue = (int) innerField.get(inngerObj);
```


### 3. 反射功能：
* 在运行时判断任意一个对象所属的类；
* 在运行时构造任意一个类的对象；
* 在运行时判断任意一个类所具有的成员变量和方法；
* 在运行时调用任意一个对象的方法；
* 生成动态代理。

### 4. 反射的利弊
为什么要用反射机制？与直接创建对象相比， 有什么区别？此处就需要说明下动态与静态编译的概念。
#### 4.1 静态编译和动态编译：
* 静态编译：在编译时确定类型，绑定对象。 
* 动态编译：运行时确定类型，绑定对象。动态编译最大限度发挥了java的灵活性，体现了多态的应用，有以降低类之间的藕合性。

#### 4.2 利弊：
* 优点：可以实现动态创建对象和编译，体现出很大的灵活性；比如，一个大型的软件，不可能一次就把把它设计的很完美，当这个程序编译后，发布了，当发现需要更新某些功能时，我们不可能要用户把以前的卸载，再重新安装新的版本，假如这样的话，这个软件肯定是没有多少人用的。采用静态的话，需要把整个程序重新编译一次才可以实现功能的更新，而采用反射机制的话，它就可以不用卸载，只需要在运行时才动态的创建和编译，就可以实现该功能。
* 缺点：性能影响。使用反射基本上是一种解释操作，我们可以告诉JVM，我们希望做什么并且它满足我们的要求。这类操作总是慢于只直接执行相同的操作。


