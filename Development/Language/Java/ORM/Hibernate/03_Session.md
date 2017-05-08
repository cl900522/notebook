Session
===========

Hibernate 允许对关联对象、属性进行延迟加载，但是必须保证延迟加载的操作限于同一个 Hibernate Session 范围之内进行。如果 Service 层返回一个启用了延迟加载功能的领域对象给 Web 层，当 Web 层访问到那些需要延迟加载的数据时，由于加载领域对象的 Hibernate Session 已经关闭，这些导致延迟加载数据的访问异常。

而Spring为我们提供的**OpenSessionInViewFilter**过滤器为我们很好的解决了这个问题。OpenSessionInViewFilter的主要功能是使每个请求过程绑定一个 Hibernate Session，即使最初的事务已经完成了，也可以在 Web 层进行延迟加载的操作。OpenSessionInViewFilter 过滤器将 Hibernate Session 绑定到请求线程中，它将自动被 Spring 的事务管理器探测到。所以 OpenSessionInViewFilter 适用于 Service 层使用HibernateTransactionManager 或 JtaTransactionManager 进行事务管理的环境，也可以用于非事务只读的数据操作中。

所谓的**OpenSessionInView**模式，把session的周期交给servlet filter来管理，每当有request进来，就打开一个session，response结束之后再关闭它，这样可以让session存在于整个请求周期中。
