Mybatis
=========================

# 使用配置


# 启动流程


# 调用流程
## 获取mapper
org.mybatis.spring.SqlSessionTemplate#getMapper
org.apache.ibatis.session.Configuration#getMapper
org.apache.ibatis.binding.MapperRegistry#getMapper
org.apache.ibatis.binding.MapperProxyFactory#newInstance->生成代理实例对象
org.apache.ibatis.binding.MapperProxy#invoke->函数执行
org.apache.ibatis.binding.MapperMethod#execute
org.apache.ibatis.binding.MapperMethod.MethodSignature#convertArgsToSqlCommandParam ->转换入参
org.apache.ibatis.session.defaults.DefaultSqlSession#selectList -> 查询列表
org.apache.ibatis.executor.CachingExecutor#query
org.apache.ibatis.executor.BaseExecutor#query
org.apache.ibatis.executor.BaseExecutor#queryFromDatabase
org.apache.ibatis.executor.SimpleExecutor#doQuery
org.apache.ibatis.session.Configuration#newStatementHandler  创建handler，嵌入plugin


