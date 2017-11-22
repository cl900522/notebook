Java定时任务管理
==================
>在java中我们常用Timer和TimerTask实现定时功能，而在JavaEE项目中可以使用Spring整合Quartz定时器、Spring的Task任务。相比于Spring自带的任务，Quartz非常的强大，能够实现所有想要的定时任务，包括Tomcat服务器开始启动，定时定点定周，集群定时任务等等的任务。

# 简介
## 从实现的技术上来分类，目前主要有三种技术（或者说有三种产品）：

1. Java自带的java.util.Timer类，这个类允许你调度一个java.util.TimerTask任务。使用这种方式可以让你的程序按照某一个频度执行，但不能在指定时间运行。一般用的较少，这篇文章将不做详细介绍。
2. 使用Quartz，这是一个功能比较强大的的调度器，可以让你的程序在指定时间执行，也可以按照某一个频度执行，配置起来稍显复杂，稍后会详细介绍。
3. Spring3.0以后自带的task，可以将它看成一个轻量级的Quartz，而且使用起来比Quartz简单许多。

## 从作业类的继承方式来讲，可以分为两类：
1. 作业类需要继承自特定的作业类基类，如Quartz中需要继承自org.springframework.scheduling.quartz.QuartzJobBean；java.util.Timer中需要继承自java.util.TimerTask。
2. 作业类即普通的java类，不需要继承自任何基类。

>**注**:个人推荐使用第二种方式，因为这样所以的类都是普通类，不需要事先区别对待。

## 从任务调度的触发时机来分，这里主要是针对作业使用的触发器，主要有以下两种：
1. 每隔指定时间则触发一次，在Quartz中对应的触发器为：org.springframework.scheduling.quartz.SimpleTriggerBean
2. 每到指定时间则触发一次，在Quartz中对应的调度器为：org.springframework.scheduling.quartz.CronTriggerBean
>**注**：并非每种任务都可以使用这两种触发器，如java.util.TimerTask任务就只能使用第一种。Quartz和spring task都可以支持这两种触发条件。

# 实现
## J2SE Timer
1. 创建Timer任务管理器；
2. 创建TimerTask子类对象，实现run函数；
3. 调用Timer的schedule**系列函数；

```java
    Timer timer = new Timer();
    TimerTask task = new TimerView();
    timer.schedule(task, new Date());
    task = new TimerView();
    timer.schedule(task, 5000);
```
## Quartz
### Quartz Only
1. 引入jar包
```xml
    <dependency>
       <groupId>org.quartz-scheduler</groupId>
       <artifactId>quartz</artifactId>
       <version>2.2.2</version>
    </dependency>
```
2. 构建job，schedule，trigger
```java
    JobDetail job1 = JobBuilder.newJob(Job1.class).withIdentity("dummyJobName1", "group1").build();
    SimpleScheduleBuilder simpleSchedule = SimpleScheduleBuilder.simpleSchedule().withIntervalInSeconds(5).repeatForever();
    // CronScheduleBuilder cronSchedule = CronScheduleBuilder.cronSchedule("0/5 * * * * ?");
    Trigger trigger = TriggerBuilder.newTrigger().withIdentity("dummyTriggerName1", "group1").withSchedule(simpleSchedule).build();
    Scheduler scheduler = new StdSchedulerFactory().getScheduler();
    scheduler.start();
    scheduler.scheduleJob(job1, trigger);
```

### Quartz With Spring

1. 引入jar包
```xml
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-support</artifactId>
        <version>${org.springframework.version}</version>
    </dependency>
    <dependency>
       <groupId>org.quartz-scheduler</groupId>
       <artifactId>quartz</artifactId>
       <version>2.2.2</version>
    </dependency>
```

2. 构建job对象，这里可以使用两种方式生成
    1. 实现子类QuartzJobBean

    ```java
        import org.quartz.JobExecutionContext;
        import org.quartz.JobExecutionException;
        import org.springframework.scheduling.quartz.QuartzJobBean;
        public class Job1 extends QuartzJobBean {

            private int timeout;
            private static int i = 0;
            //调度工厂实例化后，经过timeout时间开始执行调度
            public void setTimeout(int timeout) {
                this.timeout = timeout;
            }

            /**
            * 要调度的具体任务
            */
            @Override
            protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
                System.out.println("定时任务执行中…");
            }
        }
    ```

    通过spring bean注入

    ```xml
        <bean name="job1" class="org.springframework.scheduling.quartz.JobDetailFactoryBean">
            <property name="jobClass" value="com.gy.Job1" />
            <property name="jobDataAsMap">
            <map>
                <entry key="timeout" value="0" />
            </map>
            </property>
        </bean>
    ```

    2. 也可针对普通类使用方法名称反射构建job对象

    ```xml
        <!-- 添加调度的任务bean 配置对应的class-->
        <bean id="myPrintSchedule" class="andy.test.quartz.schedule.MyPrintSchedule" />

        <!--配置调度具体执行的方法-->
        <bean id="myPrintDetail"
            class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
            <property name="targetObject" ref="myPrintSchedule" />
            <property name="targetMethod" value="printSomething" />
            <property name="concurrent" value="false" />
        </bean>
    ```

3. 构建执行触发器
```xml
<bean id="simpleTrigger" class="org.springframework.scheduling.quartz.SimpleTriggerFactoryBean">
    <property name="jobDetail" ref="job1" />
    <property name="startDelay" value="0" /><!-- 调度工厂实例化后，经过0秒开始执行调度 -->
    <property name="repeatInterval" value="2000" /><!-- 每2秒调度一次 -->
</bean>
```
```xml
<bean id="cronTrigger" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
    <property name="jobDetail" ref="myPrintSchedule" />
    <!-- 每天12:00运行一次 -->
    <property name="cronExpression" value="0 0 12 * * ?" />
    <!-- 不重复计数，只执行一次 -->
    <property name="cronExpression">
            <property name="repeatCount" value="0" />
    </property>
</bean>
```

4. 配置调度工厂
```xml
<bean class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
    <property name="triggers">
        <list>
            <ref bean="simpleTrigger" />
            <ref bean="cronTrigger" />
        </list>
    </property>
</bean>
```
### 分部署部署
对于单机的任务调度，使用Quartz十分方便。但是在分布式情况下，对于集群中每台机器都会执行任务，从而造成了重复执行任务的问题。Quart不仅支持单机任务调度，同时也支持集群中的任务调度。原理如下：在集群中，各个不同的机器公用同一个调度器，调度器按照一定的算法选择集群中某一台机器执行任务。
```properties
#Configure Main Scheduler Properties
#==============================================================
#配置集群时，quartz调度器的id，由于配置集群时，只有一个调度器，必须保证每个服务器该值都相同，可以不用修改，只要每个ams都一样就行
org.quartz.scheduler.instanceName = Scheduler1
#集群中每台服务器自己的id，AUTO表示自动生成，无需修改
org.quartz.scheduler.instanceId = AUTO
#==============================================================
#Configure ThreadPool
#==============================================================
#quartz线程池的实现类，无需修改
org.quartz.threadPool.class = org.quartz.simpl.SimpleThreadPool
#quartz线程池中线程数，可根据任务数量和负责度来调整
org.quartz.threadPool.threadCount = 5
#quartz线程优先级
org.quartz.threadPool.threadPriority = 5
#==============================================================
#Configure JobStore
#==============================================================
#表示如果某个任务到达执行时间，而此时线程池中没有可用线程时，任务等待的最大时间，如果等待时间超过下面配置的值(毫秒)，本次就不在执行，而等待下一次执行时间的到来，可根据任务量和负责程度来调整
org.quartz.jobStore.misfireThreshold = 60000
#实现集群时，任务的存储实现方式，org.quartz.impl.jdbcjobstore.JobStoreTX表示数据库存储，无需修改
org.quartz.jobStore.class = org.quartz.impl.jdbcjobstore.JobStoreTX
#quartz存储任务相关数据的表的前缀，无需修改
org.quartz.jobStore.tablePrefix = QRTZ_
#连接数据库数据源名称，与下面配置中org.quartz.dataSource.myDS的myDS一致即可，可以无需修改
org.quartz.jobStore.dataSource = myDS
#是否启用集群，启用，改为true,注意：启用集群后，必须配置下面的数据源，否则quartz调度器会初始化失败
org.quartz.jobStore.isClustered = false
#集群中服务器相互检测间隔，每台服务器都会按照下面配置的时间间隔往服务器中更新自己的状态，如果某台服务器超过以下时间没有checkin，调度器就会认为该台服务器已经down掉，不会再分配任务给该台服务器
org.quartz.jobStore.clusterCheckinInterval = 20000
#==============================================================
#Non-Managed Configure Datasource
#==============================================================
#配置连接数据库的实现类，可以参照IAM数据库配置文件中的配置
org.quartz.dataSource.myDS.driver = com.mysql.jdbc.Driver
#配置连接数据库连接，可以参照IAM数据库配置文件中的配置
org.quartz.dataSource.myDS.URL = jdbc:mysql://localhost:3306/test
#配置连接数据库用户名
org.quartz.dataSource.myDS.user = yunxi
#配置连接数据库密码
org.quartz.dataSource.myDS.password = 123456
#配置连接数据库连接池大小，一般为上面配置的线程池的2倍
org.quartz.dataSource.myDS.maxConnections = 10
```

## Spring task
1. 设计任务类（pojo类）
给定时任务类添加@Component注解，给任务方法添加@Scheduled(cron = “0/5 * * * * ?”)注解，并让Spring扫描到该类。
```java
@Scheduled(cron = "0/5 * * * * ?")
public void work1(){
    System.out.println(Thread.currentThread().getName()+" "+"work1: 每5秒执行一次");
}
```
2. xml配置任务
```xml
<task:scheduled-tasks>
    <task:scheduled ref="monitorTask" method="loadIptvAccountSizeCache" fixed-rate="1800000" />
    <task:scheduled ref="monitorTask" method="clearCacheAtNight" cron="0 00 5 * * *" />
    <task:scheduled ref="monitorTask" method="trunOfDevices" cron="0 30 5 * * *" />
    <task:scheduled ref="monitorTask" method="fillInAreaInfoForLogs" cron="0 00 4 * * *" />
</task:scheduled-tasks>
```

3. spring优化
```xml
    <task:executor id="executor" pool-size="5" />
    <task:scheduler id="scheduler" pool-size="5" />
    <task:annotation-driven executor="executor" scheduler="scheduler" />
```
如果定时任务很多，可以配置executor线程池，这里executor的含义和java.util.concurrent.Executor是一样的，pool-size的大小官方推荐为5~10。scheduler的pool-size是ScheduledExecutorService线程池，默认为1。假如我设置了8个任务，每个任务都是每5秒钟执行一次，把下面的代码再复制7份再改一改，看看打印结果。

## cron 写法

"0/10 * * * * ?" 每10秒触发
"0 0 12 * * ?" 每天中午12点触发
"0 15 10 ? * *" 每天上午10:15触发
"0 15 10 * * ?" 每天上午10:15触发
"0 15 10 * * ? *" 每天上午10:15触发
"0 15 10 * * ? 2005" 2005年的每天上午10:15触发
"0 * 14 * * ?" 在每天下午2点到下午2:59期间的每1分钟触发
"0 0/5 14 * * ?" 在每天下午2点到下午2:55期间的每5分钟触发
"0 0/5 14,18 * * ?" 在每天下午2点到2:55期间和下午6点到6:55期间的每5分钟触发
"0 0-5 14 * * ?" 在每天下午2点到下午2:05期间的每1分钟触发
"0 10,44 14 ? 3 WED" 每年三月的星期三的下午2:10和2:44触发
"0 15 10 ? * MON-FRI" 周一至周五的上午10:15触发
"0 15 10 15 * ?" 每月15日上午10:15触发
"0 15 10 L * ?" 每月最后一日的上午10:15触发
"0 15 10 ? * 6L" 每月的最后一个星期五上午10:15触发
"0 15 10 ? * 6L 2002-2005" 2002年至2005年的每月的最后一个星期五上午10:15触发
"0 15 10 ? * 6#3" 每月的第三个星期五上午10:15触发
每隔5秒执行一次：/5 * * * ?
每隔1分钟执行一次：0 /1 * * ?
每天23点执行一次：0 0 23 * * ?
每天凌晨1点执行一次：0 0 1 * * ?
每月1号凌晨1点执行一次：0 0 1 1 * ?
每月最后一天23点执行一次：0 0 23 L * ?
每周星期天凌晨1点实行一次：0 0 1 ? * L
在26分、29分、33分执行一次：0 26,29,33 * * * ?
每天的0点、13点、18点、21点都执行一次：0 0 0,13,18,21 * * ?
