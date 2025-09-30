---
title: 游标
date: 2025-09-29 11:05:10
tags:  
---

在数据库编程中，**游标（Cursor）** 是一个核心概念，它充当应用程序与数据库之间的 “桥梁”，用于执行 SQL 语句并处理结果。无论是查询（SELECT）、插入（INSERT）、更新（UPDATE）还是删除（DELETE）操作，几乎都需要通过游标来完成。

下面从多个角度详细介绍游标：

### **一、游标的本质**

游标可以理解为：

- 数据库连接中**临时创建的 “指针” 或 “缓冲区”**
- 用于**执行 SQL 语句**并**暂存结果集**
- 提供了逐行访问结果的能力（尤其适合处理大量数据时）

简单说，游标就像数据库操作的 “执行者” 和 “搬运工”—— 它负责把你的 SQL 指令传递给数据库，再把执行结果带回给应用程序。

### **二、游标的核心作用**

1. **执行 SQL 语句**所有 SQL 操作（查询、增删改）都必须通过游标执行，例如：

cursor.execute("SELECT * FROM users")  # 执行查询 cursor.execute("UPDATE posts SET status=1 WHERE id=1")  # 执行更新

1. **处理查询结果**对于 SELECT 等返回结果集的操作，游标提供了多种获取数据的方式：

- - cursor.fetchone()：获取结果集中的**一行数据**
  - cursor.fetchmany(n)：获取**n 行数据**
  - cursor.fetchall()：获取**所有剩余数据**

示例：

cursor.execute("SELECT name, age FROM users") while True:    row = cursor.fetchone()  # 逐行获取，适合大数据量    if not row:        break    print(row)  # (张三, 25)

1. **参数化查询**游标支持通过占位符（如%s）传递参数，避免 SQL 注入风险：

\# 安全的参数传递方式 cursor.execute("SELECT * FROM users WHERE age > %s", (18,))

### **三、游标的生命周期**

一个完整的游标使用流程通常是：

1. **创建游标**：通过数据库连接对象生成

cursor = connection.cursor()  # 基于连接创建游标

1. **执行操作**：用游标执行 SQL 语句

cursor.execute(sql, params)  # params是可选参数

1. **处理结果**（如为查询操作）：通过 fetch 方法获取数据

results = cursor.fetchall()

1. **释放资源**：关闭游标（避免资源泄露）

cursor.close()

在实际开发中，通常会用try...finally确保游标一定会被关闭：

cursor = connection.cursor() try:    # 执行操作 finally:    cursor.close()  # 无论是否出错，都关闭游标

### **四、游标的常见类型与特性**

不同数据库驱动支持的游标类型可能不同，以 Python 的mysql-connector为例：

1. **默认游标**：返回元组形式的结果

cursor = connection.cursor() cursor.execute("SELECT name, age FROM users") print(cursor.fetchone())  # ('张三', 25) 元组形式

1. **字典游标**：返回字典形式的结果（键为字段名）

\# 创建时指定dictionary=True cursor = connection.cursor(dictionary=True) cursor.execute("SELECT name, age FROM users") print(cursor.fetchone())  # {'name': '张三', 'age': 25} 字典形式

1. **服务器端游标**：适合处理超大结果集（避免一次性加载全部数据到内存）需在创建时指定buffered=False（具体参数因驱动而异）。

### **五、为什么不能长期复用游标？**

1. **状态绑定**：游标会与特定 SQL 操作绑定，复用可能导致结果混乱。例如，前一个查询未读取完的结果会影响下一个操作。
2. **资源限制**：游标是有限资源，长时间不关闭会占用数据库连接的资源配额，尤其在连接池环境中可能导致连接耗尽。
3. **线程安全问题**：游标通常不是线程安全的，多线程共享游标会导致不可预期的错误。

因此，**“随用随创建，用完即关闭”** 是游标使用的最佳实践。

### **六、总结**

游标是数据库编程中不可或缺的组件，它的核心价值在于：

- 作为 SQL 执行的 “入口”
- 提供灵活的结果处理方式
- 支持安全的参数传递

理解游标的工作原理和使用规范，能帮助你写出更高效、安全的数据库操作代码。