Maven常用插件集合
=================

# maven-compiler-plugin
用途: 设置编译的jdk版本

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.6.0</version>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
    </configuration>
</plugin>
```

# maven-dependency-plugin
用途：用于复制依赖的jar包到指定的文件夹里
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <version>2.10</version>
    <executions>
        <execution>
            <id>copy-dependencies</id>
            <phase>package</phase>
            <goals>
                <goal>copy-dependencies</goal>
            </goals>
            <configuration>
                <outputDirectory>${project.build.directory}/lib</outputDirectory>
            </configuration>
        </execution>
    </executions>
</plugin>
```

# maven-jar-plugin
用途: 打成jar时，设定manifest的参数，比如指定运行的Main class，还有依赖的jar包，加入classpath中
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>2.4</version>
    <configuration>
        <archive>
            <manifest>
                <addClasspath>true</addClasspath>
                <classpathPrefix>/data/lib</classpathPrefix>
                <mainClass>com.zhang.spring.App</mainClass>
            </manifest>
        </archive>
    </configuration>
</plugin>
```

# tomcat7-maven-plugin
用途：部署Java Web项目
```xml
<plugin>
    <groupId>org.apache.tomcat.maven</groupId>
    <artifactId>tomcat7-maven-plugin</artifactId>
    <version>2.2</version>
    <configuration>
        <url>http://59.110.162.178:8080/manager/text</url>
        <username>linjinbin</username>
        <password>linjinbin</password>
    </configuration>
</plugin>
```

## maven-shade-plugin
用途：用于把多个jar包，打成1个jar包。一般Java项目都会依赖其他第三方jar包，最终打包时，希望把其他jar包包含在一个jar包里
>现在基本上都是采用maven来进行开发管理，我有一个需求是需要把通过maven管理的java工程打成可执行的jar包，这样也就是说必需把工程依赖的jar包也一起打包。而使用maven默认的package命令构建的jar包中只包括了工程自身的class文件，并没有包括依赖的jar包。所以根本不能执行。但我们可以通过配置插件来对工程进行打包。

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>2.4.3</version>
    <configuration>
        <createDependencyReducedPom>false</createDependencyReducedPom>
        <!--注意这个属性,如果你用这个插件来deploy,或者发布到中央仓库，这个属性会缩减你的pom文件,会把你依赖的<dependency>干掉-->
    </configuration>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <transformers>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <manifestEntries>
                            <Main-Class>com.meiyou.topword.App</Main-Class>
                            <X-Compile-Source-JDK>${maven.compile.source}</X-Compile-Source-JDK>
                            <X-Compile-Target-JDK>${maven.compile.target}</X-Compile-Target-JDK>
                        </manifestEntries>
                    </transformer>
                </transformers>
                <filters>
                    <filter>
                       <artifact>*:*</artifact>
                       <excludes>
                           <exclude>META-INF/*.SF</exclude>
                           <exclude>META-INF/*.DSA</exclude>
                           <exclude>META-INF/*.RSA</exclude>
                       </excludes>
                    </filter>
                </filters>
            </configuration>
        </execution>
    </executions>
</plugin>
```
