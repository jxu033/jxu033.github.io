---
title: "Jooq解决Postgres JDBC对sql参数限制"
layout: post
date: 2022-04-02 22:44
tag:
- jooq
- postgres
- jdbc
- sql
star: true
category: blog
author: jiaqixu
description: Jooq使用问题
---

### 目录
- [背景](#背景)
- [问题解析](#问题解析)
- [源码解析](#源码解析)
    - [JOOQ执行sql](#jooq执行sql)


### 背景
最近在对atlas企业计划平台项目进行从mysql到postgres的底层数据库替换开发，整个过程由于两者之间的差异(e.g. sql语法)以及项目中使用的orm对两者dialect支持等一些问题会导致项目接口不可用，需要基于数据库替换进行对应的开发工作。

替换开发过程中一个比较有意思的问题是，数据库迁移成pg之后，修改历史接口报了如下错误(备注：和数据集无关的业务我们orm用的是mybatis):
![image](/assets/images/blog/jooq-postgres-jdbc-1.png)								
图-1
![image](/assets/images/blog/jooq-postgres-jdbc-2.png)
图-2
上面的错误显示的是**tried to send an out-of-range integer as a 2-byte value: 33979. **
我们根据堆栈信息我们定位到问题所在，原来Postgres JDBC在发送query到服务端时会进行参数个数校验，如下图
![image](/assets/images/blog/jooq-postgres-jdbc-3.png)
图-3
如果绑定的参数的个数不在2个字节所能表示的有符号整数的范围内(-32768,  32767)，则会抛出图-2所示的异常。图-1中sql绑定的参数个数为33979个，故不在范围内。

所以我们现在知道了报错的原因是**PostgreSQL JDBC中对于SQL语句的参数数量限制为2个字节所能表示的有符号整数的范围(-32768 ~ 32767)**，即**最大的参数个数为32767**. 

这个时候我们开始担心了，那我们项目中例如数据同步，数据修改这些地方 使用jooq拼接的sql都有可能超过这个最大参数。
准备了一个实验数据，大概20列1W行，批量导入的时候也有5000行，那么数据导入到临时表的sql拼接完后必然会有20*5000=10W个参数 > 32767，所以我们预期是会报同样的错误的。运行后，发现数据同步成功，没有报错，并且也确认会走图-3的验证函数，但是发现val为0,  即参数个数为0，和预期的1w个参数不符合。
![image](/assets/images/blog/jooq-postgres-jdbc-4.png)
我又找了一个参数较少的case，如下，此时参数个数如预期一样是4个, 能通过验证。
![image](/assets/images/blog/jooq-postgres-jdbc-5.png)
[
](#wACSf)

奇怪了，这到底是啥情况？**为什么看上去Postgres JDBC对sql参数的限制无效，**我们带着这个问题继续进行探索
### 问题解析

1. 针对上面的2个例子，我们从源头找起，观察2个插入语句的拼装有何不同。发现用的是insertValuesStepN.execute()； 一个是insertValuesStep4.execute()，所以怀疑是不是这个问题，后来实验发现并不是这个原因。
1. 查看源码，详细见[源码解析](#rwKdF)，发现Jooq确实对绑定参数超出限制的sql做了处理，在prepare statement的时候，对于一般大小参数sql，jooq会正常解析处理；对于超出了参数限制的sql，jooq在解析处理时会进行校验并处理，如下
```java
private final Rendered getSQL0(ExecuteContext ctx) {
        Rendered result;

        // [#3542] [#4977] Some dialects do not support bind values in DDL statements
        // [#6474] [#6929] Can this be communicated in a leaner way?
        if (ctx.type() == DDL) {
            ctx.data(DATA_FORCE_STATIC_STATEMENT, true);
            DefaultRenderContext render = new DefaultRenderContext(configuration);
            result = new Rendered(render.paramType(INLINED).visit(this).render(), null, render.peekSkipUpdateCounts());
        }
        else if (executePreparedStatements(configuration().settings())) {
            try {
                DefaultRenderContext render = new DefaultRenderContext(configuration);
                render.data(DATA_COUNT_BIND_VALUES, true);
                result = new Rendered(render.visit(this).render(), render.bindValues(), render.peekSkipUpdateCounts());
            }
            catch (DefaultRenderContext.ForceInlineSignal e) {
                ctx.data(DATA_FORCE_STATIC_STATEMENT, true);
                DefaultRenderContext render = new DefaultRenderContext(configuration);
                result = new Rendered(render.paramType(INLINED).visit(this).render(), null, render.peekSkipUpdateCounts());
            }
        }
        else {
            DefaultRenderContext render = new DefaultRenderContext(configuration);
            result = new Rendered(render.paramType(INLINED).visit(this).render(), null, render.peekSkipUpdateCounts());
        }
          return result;
}
```
```java
@Override
    protected final void visit0(QueryPartInternal internal) {
        int before = bindValues.size();
        internal.accept(this);
        int after = bindValues.size();

        // [#4650] In PostgreSQL, UDTConstants are always inlined as ROW(?, ?)
        //         as the PostgreSQL JDBC driver doesn't support SQLData. This
        //         means that the above internal.accept(this) call has already
        //         collected the bind variable. The same is true if custom data
        //         type bindings use Context.visit(Param), in case of which we
        //         must not collect the current Param
        if (after == before && paramType != INLINED && internal instanceof Param) {
            Param<?> param = (Param<?>) internal;

            if (!param.isInline()) {
                bindValues.add(param);

                Integer threshold = settings().getInlineThreshold();
                if (threshold != null && threshold > 0) {
                    checkForceInline(threshold);
                }
                else {
                    switch (family()) {
                        // [#5701] Tests were conducted with PostgreSQL 9.5 and pgjdbc 9.4.1209
                        case POSTGRES:
                            checkForceInline(32767);
                            break;

                        case SQLITE:
                            checkForceInline(999);
                            break;

                        default:
                            break;
                    }
                }
            }
        }
    }


    private final void checkForceInline(int max) throws ForceInlineSignal {
        if (bindValues.size() > max)
            if (TRUE.equals(data(DATA_COUNT_BIND_VALUES)))
                throw new ForceInlineSignal();
    }
```
抛出DefaultRenderContext.ForceInlineSignal异常后, 由getSQL0函数重新捕获，解析成不带参数的静态statement继续进行处理，这样在最后进行参数校验的时候，由于静态statement的参数为0，就能通过了。

### 源码解析
#### JOOQ执行sql

```sql
    public final int execute() {
        return delegate.execute();
    }
```
```java
public final int execute() {
	...
	// first time statement preparing
	listener.renderStart(ctx);
	rendered = getSQL0(ctx);
	ctx.sql(rendered.sql);
	listener.renderEnd(ctx);
	
	...
	// prepare statement
	listener.prepareStart(ctx);
	prepare(ctx);
	listener.prepareEnd(ctx);
	
	
	...
	// bind variables
	listener.bindStart(ctx);
	if (rendered.bindValues != null)
		using(c).bindContext(ctx.statement()).visit(rendered.bindValues);
	listener.bindEnd(ctx);
	
	result = execute(ctx, listener);
	
	...
}
```
```java
@Override
protected final int execute(ExecuteContext ctx, ExecuteListener listener) throws SQLException {
    returned = null;
    returnedResult = null;
    
    if (returning.isEmpty()) {
        return super.execute(ctx, listener);
    }
    
    else {
    
        ...
            
        case POSTGRES: {
            rs = executeReturningQuery(ctx, listener);
            break;
        }
    }
```
```java
/**
 * Default implementation for query execution using a prepared statement.
 * Subclasses may override this method.
 */
 protected int execute(ExecuteContext ctx, ExecuteListener listener) throws SQLException {
     listener.executeStart(ctx);

     // [#1829] Statement.execute() is preferred over Statement.executeUpdate(), as
     // we might be executing plain SQL and returning results.
     if (!stmt.execute()) {
         result = stmt.getUpdateCount();
         ctx.rows(result);
     }
     listener.executeEnd(ctx);
 }
      
```

execute:
DefaultPreparedStatement.java -> DruidPooledPreparedStatement.java -> PgPreparedStatement.java -> PgStatement.java -> QueryExecutorImpl.java

```java
public synchronized void execute(Query query, ParameterList parameters, ResultHandler handler,
      int maxRows, int fetchSize, int flags) throws SQLException {
    waitOnLock();
    if (LOGGER.isLoggable(Level.FINEST)) {
      LOGGER.log(Level.FINEST, "  simple execute, handler={0}, maxRows={1}, fetchSize={2}, flags={3}",
          new Object[]{handler, maxRows, fetchSize, flags});
    }

    if (parameters == null) {
      parameters = SimpleQuery.NO_PARAMETERS;
    }

    flags = updateQueryMode(flags);

    boolean describeOnly = (QUERY_DESCRIBE_ONLY & flags) != 0;

    ((V3ParameterList) parameters).convertFunctionOutParameters();

    // Check parameters are all set..
    if (!describeOnly) {
      ((V3ParameterList) parameters).checkAllParametersSet();
    }

    boolean autosave = false;
    try {
      try {
        handler = sendQueryPreamble(handler, flags);
        autosave = sendAutomaticSavepoint(query, flags);
        sendQuery(query, (V3ParameterList) parameters, maxRows, fetchSize, flags,
            handler, null);
          
    ...
```
sendQuery ->sendOneQuery -> sendParse 
```java
private void sendParse(SimpleQuery query, SimpleParameterList params, boolean oneShot)
      throws IOException {
    // Already parsed, or we have a Parse pending and the types are right?
    int[] typeOIDs = params.getTypeOIDs();
    if (query.isPreparedFor(typeOIDs, deallocateEpoch)) {
      return;
        
    ...
    pgStream.sendChar(0); // End of statement name
    pgStream.send(queryUtf8); // Query string
    pgStream.sendChar(0); // End of query string.
    pgStream.sendInteger2(params.getParameterCount()); 
    ...
}
```
```java

  /**
   * Sends a 2-byte integer (short) to the back end.
   *
   * @param val the integer to be sent
   * @throws IOException if an I/O error occurs or {@code val} cannot be encoded in 2 bytes
   */
  public void sendInteger2(int val) throws IOException {
    if (val < Short.MIN_VALUE || val > Short.MAX_VALUE) {
      throw new IOException("Tried to send an out-of-range integer as a 2-byte value: " + val);
    }

    int2Buf[0] = (byte) (val >>> 8);
    int2Buf[1] = (byte) val;
    pgOutput.write(int2Buf);
  }
```

