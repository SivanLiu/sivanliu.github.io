---

title: Sqlcipher静态编译

date:  2017-9-4 22:42:21

tags:  c/c++

top: 12

---

##### 简而言之，按照自己的需求，在linux中，将sqlcipher编译成不依赖系统环境的二进制文件，源码编译出来的是动态库，需要依赖系统环境，这样会导致一个问题

```java
clang: /lib64/libc.so.6: version `GLIBC_2.14' not found (required by clang)
```
google了下，看到该问是由于编译环境的gcc版本高于运行环境的gcc版本，解决方案大概有以下两种：

###### 1. 修改生成的可执行文件，并且在执行的时候动态依赖改写的memcpy库；

###### 2.在源码编译的时候指定memcpy的版本，需要修改configure和makefile.in文件

按照顺序来吧，先说第一种解决方案，请参考：

<http://www.lightofdawn.org/wiki/wiki.cgi/NewAppsOnOldGlibc>

<http://www.jianshu.com/p/308a4e803c81>

###### 第二篇基本上是第一篇的翻译简洁版，修改二进制文件的方式与第一篇有所不同，英文好的童鞋可以直接看英文的，很详细，说下我解决的过程，主要分为三步：

- 1）查看编译后的sqlcipher可执行文件的版本信息： 

  ```shell
  readelf -V /path/sqlcipher 
  ```

- 2） 更改依赖： 

  ```shell
  for addr in 0x2f18; do printf '\x02' | dd conv=notrunc of=/home/users/kinva/tools/lib/python2.7/site-packages/annoy/annoylib.so  bs=1 seek=$((addr)); 
  ```

- 3) 将memcpy.c编译成memcpy.o文件，在执行sqlcipher时候，动态依赖： 

  ```shell
  LD_PRELOAD=./libs/memcpy.o ./sqlcipher test.db
  ```

  到此为止，第一种方案就大功告成了。

##### 再来说说第二种方案吧，请参考：

<http://m.blog.csdn.net/nullzhou/article/details/51123435>

原来基本上差不多，都是修改依赖，提供依赖函数，姑且也分为三步：

- 1.编写memcpy.c文件；

-  2.修改makefile.in文件： 

   ```shell
   TLIBS = @LIBS@ -L./libs -static $(LIBS)  

   -o $@ $(TOP)/src/shell.c $(TOP)/src/memcpy.c libsqlcipher.la \
   ```

    其中./libs为libcrypto.a的位置


- 3.修改configure:  

  ```shell
  CFLAGS="$CFLAGS -Wl,-wrap,memcpy"
  ```

  其中memcpy为添加的memcpy.c的名字



编译选项：方案1，2都相同：

```shell
./configure --disable-shared --enable-tempstore=yes CFLAGS="-DSQLITE_HAS_CODEC" LDFLAGS="./libs/libcrypto.a"
```

接下来可以编译了，完美解决。 

写的比较糙，后续有时间再详细点儿，有不明白的可以留言。
