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
1. 引入jar包
```xml
    <dependency>
       <groupId>org.quartz-scheduler</groupId>
       <artifactId>quartz</artifactId>
       <version>1.8.6</version>
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
        <bean name="job1" class="org.springframework.scheduling.quartz.JobDetailBean">
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
<bean id="simpleTrigger" class="org.springframework.scheduling.quartz.SimpleTriggerBean">
    <property name="jobDetail" ref="job1" />
    <property name="startDelay" value="0" /><!-- 调度工厂实例化后，经过0秒开始执行调度 -->
    <property name="repeatInterval" value="2000" /><!-- 每2秒调度一次 -->
</bean>
```
```xml
<bean id="cronTrigger" class="org.springframework.scheduling.quartz.CronTriggerBean">
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
