# Library可分成三种:static、shared与dynamically loaded

## Static libraries
Static链接库用于静态链接，简单讲是把一堆 object 檔用 ar(archiver) 包装集合起来，文件名以 `.a' 结尾。
**优点**是执行效能通常会比后两者快，而且因为是静态链接，所以不易发生执行时找不到 library 或版本错置而无法执行的问题。
**缺点**则是档案较大，维护度较低；例如 library 如果发现 bug 需要更新，那么就必须重新连结执行档。
### 创建
```c++
    ____ hello.c ____
    #include
    void hello(){ printf("Hello "); }

    ____ world.c ____
    #include
    void world(){ printf("world."); }

    ____ mylib.h ____
    void hello();
    void world();
```
```shell
    $ gcc -c hello.c world.c /* 编出 hello.o 与 world.o */
    $ ar rcs libmylib.a hello.o world.o /* 包成 limylib.a */
```
这样就可以建出一个档名为 libmylib.a 的檔。输出的档名其实没有硬性规定，但如果想要配合 gcc 的 '-l' 参数来连结，一定要以 'lib'开头，中间是你要的 library 名称，然后紧接着 `.a' 结尾。

### 使用
```c++
    ____ main.c ____
    #include "mylib.h"
    int main() {
        hello();
        world();
    }
```
使用上就像与一般的 object 档连结没有差别。
```shell
    $ gcc main.c libmylib.a
```
也可以配合 gcc 的 `-l' 参数使用
```shell
    $ gcc main.c -L. -lmylib
```

## Shared libraries
Shared library 会在程序执行起始时才被自动加载。因为链接库与执行档是分离的，所以维护弹性较好。有两点要注意， shared library 是在程序起始时就要被加载，而不是执行中用到才加载，而且在连结阶段需要有该链接库才能进行连结。

首先有一些名词要弄懂， soname 、real name 与 linker name 。
1. soname 用来表示是一个特定 library 的名称，像是 libmylib.so.1 。前面以 'lib' 开头，接着是该 library 的名称，然后是 `.so' ，接着
是版号，用来表名他的界面；如果接口改变时，就会增加版号来维护兼容度。
2. real name 是实际放有 library 程序的文件名，后面会再加上 minor 版号与release 版号，像是 libmylib.so.1.0.0 。
一般来说，版号的改变规则是 ( 印象中在 APress-Difinitive Guide to GCC 中有提到，但目前手边没这本书 ) ，最后缀的 release 版号用于程序内容的修正，接口完全没有改变。中间的 minor 用于有新增加接口，但相旧接口没改变，所以与旧版本兼容。最前面的 version 版号用于原接口有移除或改变，与旧版不兼容时。
3. linker name 是用于连结时的名称，是不含版号的 soname ，如 : libmylib.so 。
通常 linker name 与 real name 是用 ln 指到对应的 real name ，用来提供弹性与维护性。

### 创建
shared library 的制作过程较复杂。
```shell
    $ gcc -c -fPIC hello.c world.c
```
编译时要加上 -fPIC 用来产生 position-independent code 。也可以用 -fpic 参数。 ( 不太清楚差异，只知道 -fPIC 较通用于不同平台，但产生的 code 较大，而且编译速度较慢 ) 。
```shell
    $ gcc -shared -Wl,-soname,libmylib.so.1 -o libmylib.so.1.0.0 hello.o world.o
```
-shared 表示要编译成 shared library
-Wl 用于参递参数给 linker ，因此 -soname 与 libmylib.so.1 会被传给 linker 处理
-soname 用来指名 soname 为 limylib.so.1
-o 指明library 会被输出成 libmylib.so.1.0.0 ( 也就是 real name)
若不指定 soname 的话，在编译结连后的执行档会以连时的 library 档名为 soname ，并载入他。否则是载入 soname 指定的 library 档案。

### 使用
与使用 static library相同
```shell
    $ gcc main.c libmylib.so
```
以上直接指定与 libmylib.so 连结。或用
```shell
    $ gcc main.c -L. -lmylib
```
linker 会搜寻 libmylib.so 来进行连结。如果目录下同时有 static 与 shared library 的话，会以 shared 为主。使用 -static 参数可以避免使用 shared 连结。
```shell
    $ gcc main.c -static -L. -lmylib
```
此时可以用 ldd 看编译出的执行档与 shared 链接库的相依性
```shell
    $ldd a.out
    linux-gate.so.1 => (0xffffe000)
    libmylib.so.1 => not found
    libc.so.6 => /lib/libc.so.6 (0xb7dd6000)
    /lib/ld-linux.so.2 (0xb7f07000)
```

输出结果显示出该执行文件需要 libmylib.so.1 这个 shared library 。
会显示 not found 因为没指定该 library 所在的目录，所找不到该 library 。
因为编译时有指定 -soname 参数为 libmylib.so.1 的关系，所以该执行档会加载 libmylib.so.1 。否则以 libmylib.so 连结，执行档则会变成要求加载 libmylib.so
```shell
    $ ./a.out
    ./a.out: error while loading shared libraries: libmylib.so.1:
    cannot open shared object file: No such file or directory
```

因为找不到 libmylib.so.1 所以无法执行程序。有几个方式可以处理:
1. 把 libmylib.so.1 安装到系统的 library 目录，如 /usr/lib 下
2. 设定 /etc/ld.so.conf ，加入一个新的 library 搜寻目录，并执行 ldconfig
更新快取
3. 设定 LD_LIBRARY_PATH 环境变量来搜寻 library
这个例子是加入当前目录来搜寻要载作的 library
$ LD_LIBRARY_PATH=. ./a.out
Hello world.

## Dynamically loaded libraries
Dynamicaaly loaded libraries 才是像 windows 所用的 DLL ，在使用到时才加载，编译连结时不需要相关的 library 。动态载入库常被用于像 plug-ins的应用。
### 使用
动态加载是透过一套 dl function 来处理。
```c++
    #include
    void *dlopen(const char *filename, int flag);
    //开启加载 filename 指定的 library 。
    void *dlsym(void *handle, const char *symbol);
    //取得 symbol 指定的 symbol name 在 library 被加载的内存地址。
    int dlclose(void *handle);
    //关闭 dlopen 开启的 handle 。
    char *dlerror(void);
    //传回最近所发生的错误讯息。
```
```c++
    ____ dltest.c ____
    #include <stdio.h>
    #include <stdlib.h>
    #include <stddef.h>
    #include <dlfcn.h>
    int main() {
        void *handle;
        void (*f)();
        char *error;
        /* 开启之前所撰写的 libmylib.so 链接库 */
        handle = dlopen("./libmylib.so", RTLD_LAZY);
        if( !handle ) {
            fputs( dlerror(), stderr);
            exit(1);
        }
        /* 取得 hello function 的 address */
        f = dlsym(handle, "hello");
        if(( error=dlerror())!=NULL) {
            fputs(error, stderr);
            exit(1);
        }
        /* 呼叫该 function */
        f();
        dlclose(handle);
    }
```

编译时要加上 -ldl 参数来与 dl library 连结
```shell
    $ gcc dltest.c -ldl
```
结果会印出 Hello 字符串
$ ./a.out
Hello
