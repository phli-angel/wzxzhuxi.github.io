---
layout: post
title: .NET Core 数据库操作方法
author: phli
date: 2025-11-15
categories: [工业控制]
tags: [.NET Core, C#, SQL Server, Database, ADO.NET]
excerpt: 在 .NET Core 项目中操作 SQL Server 数据库的实用方法,涵盖 SQL 执行、参数化查询、事务处理和存储过程调用
---

# .NET Core 数据库操作方法

大家好!今天给大家分享一下在 .NET Core 项目中操作 SQL Server 数据库的一些实用方法。这篇文章会涵盖从基础的 SQL 执行、参数化查询,到事务处理和存储过程调用等场景,相信会对大家的日常开发有所帮助。

## 封装格式化SQL语句方法

### 非读操作

先来看最常用的增删改操作。这个方法可以用来执行 INSERT、UPDATE、DELETE 这类不返回结果集的 SQL 语句。

```c#
public static int Update(string sql)
{
    SqlConnection conn = new SqlConnection(connString);
    SqlCommand cmd = new SqlCommand(sql, conn);

    try
    {
        conn.Open();
        return cmd.ExecuteNonQuery();
    }
    catch (Exception e)
    {
        string errorInfo = "调用 public static int Update(string sql) 方法发生错误:" + e.Message;
        WriteLog(errorInfo);
        throw new Exception(errorInfo);
    }
    finally
    {
        conn.Close();
    }
}
```

这个方法会返回受影响的行数,比如你删了 3 条数据,它就返回 3。

### 单条读操作

有时候我们只想查一个值,比如统计总数、获取最大值之类的,这时候用 `ExecuteScalar()` 就很方便了。

```c#
public static object GetSingleResult(string sql)
{
    SqlConnection conn = new SqlConnection(connString);
    SqlCommand cmd = new SqlCommand(sql, conn);

    try
    {
        conn.Open();
        return cmd.ExecuteScalar();
    }
    catch (Exception e)
    {
        string errorInfo = "调用 public static object GetSingleResult(string sql) 方法发生错误:" + e.Message;
        WriteLog(errorInfo);
        throw new Exception(errorInfo);
    }
    finally
    {
        conn.Close();
    }
}
```

它会返回查询结果的第一行第一列,如果啥也没查到就返回 null。简单直接!

### 多条读操作

当需要读取大量数据时,`SqlDataReader` 是个不错的选择。它的特点是只能向前读取,但性能很高。

```c#
public static SqlDataReader GetReader(string sql)
{
    SqlConnection conn = new SqlConnection(connString);
    SqlCommand cmd = new SqlCommand(sql, conn);

    try
    {
        conn.Open();
        return cmd.ExecuteReader(CommandBehavior.CloseConnection);
    }
    catch (Exception e)
    {
        string errorInfo = "调用 public static SqlDataReader GetReader(string sql) 方法发生错误:" + e.Message;
        WriteLog(errorInfo);
        throw new Exception(errorInfo);
    }
}
```

**小提示**: 这里用了 `CommandBehavior.CloseConnection`,这样当你关闭 Reader 的时候,数据库连接也会自动关闭,省得忘记了。

### 使用DataSet读操作

如果你需要在内存中缓存数据,或者要处理多个表的关系,`DataSet` 就派上用场了。虽然它比 Reader 重一些,但胜在灵活。

```c#
public static DataSet GetDataSet(string sql)
{
    SqlConnection conn = new SqlConnection(connString);
    SqlCommand cmd = new SqlCommand(sql, conn);
    SqlDataAdapter da = new SqlDataAdapter(cmd);
    DataSet ds = new DataSet();
    
    try
    {
        conn.Open();
        da.Fill(ds);
        return ds;
    }
    catch (Exception e)
    {
        string errorInfo = "调用 public static DataSet GetDataSet(string sql) 方法发生错误:" + e.Message;
        WriteLog(errorInfo);
        throw new Exception(errorInfo);
    }
    finally
    {
        conn.Close();
    }
}
```

## 封装有参数SQL语句执行的各种方法

前面的方法虽然能用,但有个大问题——容易被 SQL 注入攻击！所以强烈建议大家用参数化查询。不仅安全,性能也更好。

### 非读操作

来看看怎么用参数:

```c#
public static int Update(string sql, SqlParameter[] parameters)
{
    SqlConnection conn = new SqlConnection(connString);
    SqlCommand cmd = new SqlCommand(sql, conn);
    
    try
    {
        conn.Open();
        cmd.Parameters.AddRange(parameters);
        return cmd.ExecuteNonQuery();
    }
    catch (Exception e)
    {
        string errorInfo = "调用 public static int Update(string sql, SqlParameter[] parameters) 方法发生错误:" + e.Message;
        WriteLog(errorInfo);
        throw new Exception(errorInfo);
    }
    finally
    {
        conn.Close();
    }
}
```

**实际使用起来是这样的**:
```c#
string sql = "UPDATE Users SET Name = @name WHERE Id = @id";
SqlParameter[] parameters = {
    new SqlParameter("@name", "张三"),
    new SqlParameter("@id", 1)
};
int rows = Update(sql, parameters);
```

看到没?用 `@` 符号做占位符,然后把实际的值通过参数传进去,这样就安全多了。

### 单条读操作

```c#
public static object GetSingleResult(string sql, SqlParameter[] parameters)
{
    SqlConnection conn = new SqlConnection(connString);
    SqlCommand cmd = new SqlCommand(sql, conn);

    try
    {
        conn.Open();
        cmd.Parameters.AddRange(parameters);
        return cmd.ExecuteScalar();
    }
    catch (Exception e)
    {
        string errorInfo = "调用 public static object GetSingleResult(string sql, SqlParameter[] parameters) 方法发生错误:" + e.Message;
        WriteLog(errorInfo);
        throw new Exception(errorInfo);
    }
    finally
    {
        conn.Close();
    }
}
```

### 多条读操作

```c#
public static SqlDataReader GetReader(string sql, SqlParameter[] parameters)
{
    SqlConnection conn = new SqlConnection(connString);
    SqlCommand cmd = new SqlCommand(sql, conn);

    try
    {
        conn.Open();
        cmd.Parameters.AddRange(parameters);
        return cmd.ExecuteReader(CommandBehavior.CloseConnection);
    }
    catch (Exception e)
    {
        string errorInfo = "调用 public static SqlDataReader GetReader(string sql, SqlParameter[] parameters) 方法发生错误:" + e.Message;
        WriteLog(errorInfo);
        throw new Exception(errorInfo);
    }
}
```

### 使用DataSet读操作

```c#
public static DataSet GetDataSet(string sql, SqlParameter[] parameters)
{
    SqlConnection conn = new SqlConnection(connString);
    SqlCommand cmd = new SqlCommand(sql, conn);
    SqlDataAdapter da = new SqlDataAdapter(cmd);
    DataSet ds = new DataSet();
    
    try
    {
        conn.Open();
        cmd.Parameters.AddRange(parameters);
        da.Fill(ds);
        return ds;
    }
    catch (Exception e)
    {
        string errorInfo = "调用 public static DataSet GetDataSet(string sql, SqlParameter[] parameters) 方法发生错误:" + e.Message;
        WriteLog(errorInfo);
        throw new Exception(errorInfo);
    }
    finally
    {
        conn.Close();
    }
}
```

## 事务处理

### 启用事务提交多条SQL语句(带参数)

事务这东西很重要!比如你在做订单系统,需要同时往订单主表和明细表插数据,万一中间出错了怎么办?这时候事务就能保证"要么全成功,要么全失败",不会出现数据不一致的情况。

```c#
/// <summary>
/// 启用事务提交多条SQL语句(带参数)
/// </summary>
/// <param name="mainSql">主表SQL语句</param>
/// <param name="mainParam">主表SQL语句对应的参数</param>
/// <param name="detailSql">明细表SQL语句</param>
/// <param name="detailParam">明细表SQL语句对应的参数</param>
/// <returns>操作是否成功</returns>
/// <exception cref="Exception">操作失败时抛出异常</exception>
public static bool UpdateByTran(string mainSql, SqlParameter[] mainParam, 
                                string detailSql, List<SqlParameter[]> detailParam)
{
    SqlConnection conn = new SqlConnection(connString);
    SqlCommand cmd = new SqlCommand();
    cmd.Connection = conn;
    
    try
    {
        conn.Open();
        cmd.Transaction = conn.BeginTransaction();
        
        // 先处理主表
        if (mainSql != null && mainSql.Length != 0)
        {
            cmd.CommandText = mainSql;
            cmd.Parameters.AddRange(mainParam);
            cmd.ExecuteNonQuery();
        }

        // 再处理明细表,可能有多条
        foreach (SqlParameter[] param in detailParam)
        {
            cmd.CommandText = detailSql;
            cmd.Parameters.Clear();
            cmd.Parameters.AddRange(param);
            cmd.ExecuteNonQuery();
        }

        cmd.Transaction.Commit();  // 全部成功,提交!
        return true;
    }
    catch (Exception e)
    {
        if (cmd.Transaction != null)
        {
            cmd.Transaction.Rollback();  // 出错了,回滚!
        }

        string errorInfo = "调用 UpdateByTran 方法发生错误:" + e.Message;
        WriteLog(errorInfo);
        throw new Exception(errorInfo);
    }
    finally
    {
        if (cmd.Transaction != null)
        {
            cmd.Transaction = null;
        }
        conn.Close();
    }
}
```

**典型场景**: 比如用户下单,你得在订单表里插入一条主订单记录,然后在订单明细表里插入多条商品记录。用事务就能确保这些操作要么一起成功,要么一起失败,不会出现只插了主订单但明细没插成功的尴尬情况。

## 封装调用存储过程的各种方法

有些复杂的业务逻辑,写成存储过程会更高效。虽然现在很多项目不太用存储过程了,但在某些场景下它还是很有用的。

### 非读操作

```c#
public static int UpdateByProcedure(string spName, SqlParameter[] parameters)
{
    SqlConnection conn = new SqlConnection(connString);
    SqlCommand cmd = new SqlCommand(spName, conn);

    try
    {
        conn.Open();
        cmd.CommandType = CommandType.StoredProcedure;  // 注意这里要指定类型
        cmd.Parameters.AddRange(parameters);
        return cmd.ExecuteNonQuery();
    }
    catch (Exception e)
    {
        string errorInfo = "调用 public static int UpdateByProcedure(string spName, SqlParameter[] parameters) 方法发生错误:" + e.Message;
        WriteLog(errorInfo);
        throw e;
    }
    finally
    {
        conn.Close();
    }
}
```

### 单条读操作

```c#
public static object GetSingleResultByProcedure(string spName, SqlParameter[] parameters)
{
    SqlConnection conn = new SqlConnection(connString);
    SqlCommand cmd = new SqlCommand(spName, conn);

    try
    {
        conn.Open();
        cmd.CommandType = CommandType.StoredProcedure;
        cmd.Parameters.AddRange(parameters);
        return cmd.ExecuteScalar();
    }
    catch (Exception e)
    {
        string errorInfo = "调用 public static object GetSingleResultByProcedure(string spName, SqlParameter[] parameters) 方法发生错误:" + e.Message;
        WriteLog(errorInfo);
        throw new Exception(errorInfo);
    }
    finally
    {
        conn.Close();
    }
}
```

### 多条读操作

```c#
public static SqlDataReader GetReaderByProcedure(string spName, SqlParameter[] parameters)
{
    SqlConnection conn = new SqlConnection(connString);
    SqlCommand cmd = new SqlCommand(spName, conn);

    try
    {
        conn.Open();
        cmd.CommandType = CommandType.StoredProcedure;
        cmd.Parameters.AddRange(parameters);
        return cmd.ExecuteReader(CommandBehavior.CloseConnection);
    }
    catch (Exception e)
    {
        string errorInfo = "调用 public static SqlDataReader GetReaderByProcedure(string spName, SqlParameter[] parameters) 方法发生错误:" + e.Message;
        WriteLog(errorInfo);
        throw new Exception(errorInfo);
    }
}
```

### 启用事务调用带参数的存储过程

有时候一个存储过程要调用多次,这时候也需要事务保护:

```c#
/// <summary>
/// 启用事务调用带参数的存储过程
/// </summary>
/// <param name="procedureName">存储过程名称</param>
/// <param name="paramArray">存储过程参数数组</param>
/// <returns>操作是否成功</returns>
/// <exception cref="Exception">操作失败时抛出异常</exception>
public static bool UpdateByTran(string procedureName, List<SqlParameter[]> paramArray)
{
    SqlConnection conn = new SqlConnection(connString);
    SqlCommand cmd = new SqlCommand();
    cmd.Connection = conn;
    
    try
    {
        conn.Open();
        cmd.CommandType = CommandType.StoredProcedure;
        cmd.CommandText = procedureName;
        cmd.Transaction = conn.BeginTransaction();

        foreach (SqlParameter[] param in paramArray)
        {
            cmd.Parameters.Clear();
            cmd.Parameters.AddRange(param);
            cmd.ExecuteNonQuery();
        }

        cmd.Transaction.Commit();
        return true;
    }
    catch (Exception e)
    {
        if (cmd.Transaction != null)
        {
            cmd.Transaction.Rollback();
        }

        string errorInfo = "调用 UpdateByTran(string procedureName, List<SqlParameter[]> paramArray) 方法发生错误:" + e.Message;
        WriteLog(errorInfo);
        throw new Exception(errorInfo);
    }
    finally
    {
        if (cmd.Transaction != null)
        {
            cmd.Transaction = null;
        }
        conn.Close();
    }
}
```

## 最佳实践

写了这么多代码,下面分享几个实用小技巧:

### 连接字符串管理
别把连接字符串写死在代码里!应该放在 `appsettings.json` 配置文件中:

```c#
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyDB;User Id=sa;Password=****;"
  }
}
```

这样改环境的时候直接改配置文件就行了,不用动代码。

### 资源释放优化
数据库连接是很宝贵的资源,用完一定要关!不过手动写 `try-finally` 有点麻烦,用 `using` 语句就方便多了:

```c#
using (SqlConnection conn = new SqlConnection(connString))
{
    conn.Open();
    // 你的数据库操作
}  // 出了这个大括号,连接自动关闭
```

### 异步操作支持
现在是 2025 年了,异步编程已经很成熟了。如果你的项目是 Web API 或者高并发场景,强烈建议用异步方法:

```c#
public static async Task<int> UpdateAsync(string sql, SqlParameter[] parameters)
{
    using (SqlConnection conn = new SqlConnection(connString))
    {
        SqlCommand cmd = new SqlCommand(sql, conn);
        cmd.Parameters.AddRange(parameters);
        
        await conn.OpenAsync();
        return await cmd.ExecuteNonQueryAsync();
    }
}
```

异步方法不会阻塞线程,能大大提升系统的并发处理能力。

## 注意事项

最后再强调几个容易踩的坑:

- **SQL注入防护**: 这个真的很重要!千万别用字符串拼接的方式写 SQL,老老实实用参数化查询。我见过太多因为这个被黑的案例了。

- **事务隔离级别**: 事务有不同的隔离级别(读未提交、读已提交、可重复读、序列化),默认是读已提交。根据你的业务需求选择合适的级别,不要一上来就用最高级别,会影响性能。

- **连接池管理**: ADO.NET 默认开启了连接池,这是个好东西,能重用连接提高性能。但你还是要记得及时关闭连接,不然连接池里的连接会被占满。

- **异常处理**: 出错了一定要记日志!不然线上出问题了根本不知道哪里错了。我的代码里都有 `WriteLog` 方法,就是干这个用的。

- **性能优化**: 如果要批量插入大量数据(比如几万条),用普通的 INSERT 会很慢。这时候应该用 `SqlBulkCopy`,速度能提升好几个数量级。

## 写在最后

这些方法都是我在实际项目中总结出来的,基本能覆盖大部分场景了。当然,现在很多项目都在用 EF Core 或 Dapper 这样的 ORM 框架,但了解底层的 ADO.NET 还是很有必要的,毕竟万变不离其宗嘛。

希望这篇文章对你有帮助!如果有什么问题,欢迎留言交流~ 😊

---

**作者**: Phli  
**日期**: 2025-11-15  
**版本**: 1.0

