# Javac
javac命令用与编译java源码文件，其语法格式如下：
javac [ options ] [ sourcefiles ] [ @files ]

参数可按任意次序排列：
* options                       命令行选项。
* sourcefiles                 一个或多个要编译的源文件（例如 MyClass.java）。
* @files                           一个或多个对源文件进行列表的文件。

有两种方法可将源代码文件名传递给 javac：
## sourcefiles
直接给出要编译的源文件。如果源文件数量少，可以用这种方式，在命令行上列出文件名即可。文件与文件之间用空格非分开就可以了。
```shell
javac -d classes src\com\robin\Hello.java src\com\robin\People.java src\com\hubin\Util.java
```

其实这里的源码文件每个都是单独的参数，如果文件路径包括有空格，可以用双引号把该文件名括起来。
比如上面的命令可以写成下面的样子：
```shell
javac -d classes "src\com\robin\Hello.java" "src\com\robin\People.java" "src\com\hubin\Util.java"
```

在没使用分号的情况下，对相同路径下的Java源码文件可以使用统配符，比如示例1可以写成：
```shell
javac -d classes src\com\robin\*.java src\com\hubin\Util.java
```

## @files
为缩短或简化javac命令，可以把要编译的java源文件名列在一个文件，文件名之间用空格或回车进行分割。然后在javac命令行中，可以用'@' 字符加上包含有要编译java源文件名的文件名来指定要编译的java源文件。因为javac当遇到以 `@' 字符，它就会对该字符后的文件所列出的所有java源文件进行编译。这种形式适用于java源文件很多的情况。比如，我们把示例1要编译的源文件名包含在src.txt文件中。

src.txt文件
```
src\com\robin\Hello.java src\com\robin\People.java
src\com\hubin\Util.java
```

然后运行如下的javac命令：
```shell
javac -d classes @src.txt
```

当然我们可以在src.txt中用双引号把单个要编译的java源码文件括起来，但是这时路径直接的分隔符“\”就要写成"\\"的形式了。

src.txt文件
```
"src\\com\\robin\\Hello.java" "src\\com\\robin\\People.java"
"src\\com\\hubin\\Util.java"
```
这时javac命令仍然同上。

## 命令选项options

### -d
该选项用于指定生成的class目标文件的目录。如果某个类是一个包的组成部分，则 javac 将把该类文件放入反映包名的子目录中，必要时创建目录。比如，示例1中的class目标目录就放在classes下，Hello.class和People.class位于classes\com\robin下，Util.class位于classes\com\hubin目录下。

>若未指定 -d 选项，则 javac 将把类文件放到与源文件相同的目录中。
>注意： -d 选项指定的目录不会被自动添加到用户类路径中。

### -bootclasspath，-extdirs，-classpath和-cp

JDK在编译一个java源文件时，搜索类文件的方式和顺序如下：
* Bootstrap classes，
    Bootstrap默认的是JDK自带的jar或zip文件，它包括jre\lib下rt.jar等文件，JDK首先搜索这些文件.可以通过-bootclasspath来设置它。文件之间用分号";"进行分割。
* Extension classes，
    Extension默认的是位于jre"lib"ext目录下的jar文件，JDK在搜索完Bootstrap后就搜索该目录下的jar文件.可以通过-extdirs来设置。文件之间用分号";"来进行分割
* User classes
    User classes搜索顺序为当前目录、环境变量 CLASSPATH、-classpath。

### -cp
-cp 和 -classpath 是同义词，参数意义是一样的。classpath参数太长了，所以提供cp作为缩写形式它们用于告知JDK搜索目录名、jar文档名、zip文档名，用分号";"进行分隔。

例如当你自己开发了公共类并包装成一个common.jar包，在使用 common.jar中的类时，就需要用-classpath common.jar 告诉JDK从common.jar中查找该类，否则JDK就会抛出java.lang.NoClassDefFoundError异常，表明未找到类定义。

使用-classpath后JDK将不再使用CLASSPATH中的类搜索路径，如果-classpath和CLASSPATH都没有设置，则JDK使用当前路径(.)作为类搜索路径。

推荐使用-classpath来定义JDK要搜索的类路径，而不要使用环境变量 CLASSPATH的搜索路径，以减少多个项目同时使用CLASSPATH时存在的潜在冲突。例如应用1要使用a1.0.jar中的类G，应用2要使用 a2.0.jar中的类G,a2.0.jar是a1.0.jar的升级包，当a1.0.jar，a2.0.jar都在CLASSPATH中，JDK搜索到第一个包中的类G时就停止搜索，如果应用1应用2的虚拟机都从CLASSPATH中搜索，就会有一个应用得不到正确版本的类G。
```shell
javac -classpath lib\Util.zip -d classes src\com\robin\*.java
或
javac -cp lib\Util.zip -d classes src\com\robin\*.java
```

```shell
javac -classpath classes -d classes src\com\robin\*.java
或
javac -cp classes -d classes src\com\robin\*.java
```
如果需要指定各个JAR文件具体的存放路径，相同路径有多个可使用通配符。

### -sourcepath
-sourcepath java源码文件路径
在编译时，JDK需要两方面的路径，一个是查找java源码文件的路径，一个是查找class（类）文件的路径。关于class文件的路径上文已经已经介绍过，可以通过-bootclasspath，-extdirs，-classpath和-cp来设定。java源码文件的路径则可以通过

-sourcepath来设定，默认情况下-sourcepath和-classpath的路径一样。在编译的过程中，若需要相关java类的则首先在sourcefiles或@files列出的java源码文件中查找并编译，如果没找到，就在-sourcepath指定的路径中查找java源码文件，这时无论找没找到都会继续在类路径中进行查找。如果在sourcepath中找到了java源码文件，但是在类路径中没有找到了相关的类，或找的类位于包文件（jar或zip）中,或找的类并不是在包文件中，但源码文件比该类文件新，这时会对源码文件进行编译，而且编译生成的类文件将会和你指定要进行编译的java源码所生成的类文件位于同一根目录。否则，除了即没找到java源码文件也没找到相关类就编译失败外，直接载入相关类就可以了。因此你得至少要指定一个要编译的java源文件。它并不是指定sourecfiles或@files中指定的要编译的java源码文件的根目录。与类路径一样，java源码路径项用分号 (;) 进行分隔，它们可以是class文件的根目录、JAR 归档文件或 ZIP 归档文件。
```shell
javac -sourcepath src -d classes src\com\robin\Hello.java
javac -cp lib\Util.zip -sourcepath src -d classes src\com\robin\*.java
```

### -source和-target
* -source 版本
当你从sun安装了某个版本的JDK，而其实该JDK却包含多个版本的编译器。-source参数就是指定用哪个版本的编译器对java源码进行编译。如果你的java源码不符合该版本编译器的规范的话，当然就不能编译通过。

* -target 版本
该命令用于指定生成的class文件将保证和哪个版本的虚拟机进行兼容。我们可以通过-target 1.2来保证生成的class文件能在1.2虚拟机上进行运行，但是1.1的虚拟机就不能保证了。因为java虚拟机的向前兼容行，1.5的虚拟机当然也可以运行通过-target 1.2让生成的class文件。

每个版本编译器的默认-target版本是不太一样的，比如：
JDK1.2版本编译器支持-target 1.1,-target 1.2,-target 1.3,-target 1.4,-target 1.5,-target 1.6,它默认的就是1.1
JDK1.4版本编译器只支持-target 1.4 和-target 1.5
JDK1.5版本编译器就只支持-target 1.5

```shell
javac -cp lib\Util.zip -sourcepath src -source 1.2 -target 1.1 -d classes src\com\robin\*.java
javac -cp lib\Util.zip -sourcepath src -source 1.2 -target 1.3 -d classes src\com\robin\*.java
javac -cp lib\Util.zip -sourcepath src -source 1.2 -target 1.4 -d classes src\com\robin\*.java
javac -cp lib\Util.zip -sourcepath src -source 1.2 -target 1.5 -d classes src\com\robin\*.java
javac -cp lib\Util.zip -sourcepath src -source 1.2 -target 1.6 -d classes src\com\robin\*.java
```

```shell
javac -cp lib\Util.zip -sourcepath src -source 1.4 -target 1.4 -d classes src\com\robin\*.java
javac -cp lib\Util.zip -sourcepath src -source 1.4 -target 1.5 -d classes src\com\robin\*.java
```

### -deprecation
如果java源码中使用了的不鼓励使用的类或类的Field或类的方法或类的方法覆盖，那么如果使用了该参数，将显示关于此的的详细信息,否则只有个简单的Note.
```
D:\project\test>javac -cp lib\Util.zip -sourcepath src -source 1.2 -target 1.6 -
d classes src\com\robin\*.java
Note: src\com\robin\Hello.java uses or overrides a deprecated API.
Note: Recompile with -Xlint:deprecation for details.

D:\project\test>javac -cp lib\Util.zip -sourcepath src -deprecation -d classes s
rc\com\robin\*.java
src\com\robin\Hello.java:11: warning: [deprecation] destroy() in java.lang.Threa
d has been deprecated
t.destroy();
               
1 warning
```

### -encoding
设置源文件编码名称，例如UTF-8。若未指定 -encoding 选项，则使用平台缺省的编码方式。

### -g
生成所有的调试信息，包括局部变量。缺省情况下，只生成行号和源文件信息。
    -g:none
    不生成任何调试信息。
    -g:{关键字列表}
    只生成某些类型的调试信息，这些类型由逗号分隔的关键字列表所指定。有效的关键字有：
    source
    源文件调试信息
    lines
    行号调试信息
    vars
    局部变量调试信息

### -nowarn
禁用警告信息。
