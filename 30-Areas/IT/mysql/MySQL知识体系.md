# MySQL 全栈知识体系

这份文档涵盖了从 MySQL 入门到高性能优化的完整知识路径。

## 📍 阶段一：基础入门 (The Basics)

### 1. 核心概念

- **RDBMS**: 关系型数据库管理系统。
- **SQL**: 结构化查询语言 (DDL, DML, DQL, DCL)。
- **表 (Table)**: 行 (Row) 和列 (Column) 组成的二维数据结构。

### 2. DDL (数据定义语言)

用于定义数据库对象（数据库、表、字段）。

- `CREATE DATABASE / TABLE`
- `ALTER TABLE` (ADD, MODIFY, CHANGE, DROP)
- `DROP TABLE`
- `TRUNCATE TABLE` (清空表数据，保留结构)

### 3. 数据类型 (Data Types)

- **数值型**: `INT`, `BIGINT`, `DECIMAL`(金融金额), `FLOAT`, `DOUBLE`
- **字符串**: `CHAR`(固定长度), `VARCHAR`(可变长度), `TEXT`
- **日期时间**: `DATETIME`, `TIMESTAMP`, `DATE`
- **二进制**: `BLOB`

### 4. DML (数据操作语言)

- `INSERT INTO`: 插入数据
- `UPDATE`: 更新数据 (⚠️ 记得加 WHERE)
- `DELETE FROM`: 删除数据

### 5. DQL (数据查询语言) - 基础

- `SELECT *`: 查询所有列
- `WHERE`: 条件过滤 (`=, <>, >, <, BETWEEN, IN, LIKE, IS NULL`)
- `ORDER BY`: 排序 (`ASC`, `DESC`)
- `LIMIT`: 分页查询 (`LIMIT offset, count`)

### 6. DCL (数据控制语言) & 用户管理

用于定义访问权限和安全级别。

- **创建用户**: `CREATE USER 'username'@'host' IDENTIFIED BY 'password';`
- **授权**: `GRANT ALL PRIVILEGES ON database.* TO 'username'@'host';`
- **刷新权限**: `FLUSH PRIVILEGES;`
- **撤销权限**: `REVOKE ALL PRIVILEGES ON ... FROM ...;`

### 7. 约束 (Constraints)

保证数据的完整性和一致性。

- **PRIMARY KEY**: 主键 (非空且唯一)。
- **NOT NULL**: 非空。
- **UNIQUE**: 唯一。
- **DEFAULT**: 默认值。
- **FOREIGN KEY**: 外键 (用于限制两个表的关系)。

---

## 🚀 阶段二：进阶提升 (Advanced Guide)

### 1. 聚合与分组

- **聚合函数**: `COUNT()`, `SUM()`, `AVG()`, `MAX()`, `MIN()`
- **GROUP BY**: 对结果集进行分组。
- **HAVING**: 分组后的条件过滤 (WHERE 是分组前过滤)。
  > **执行顺序**: FROM -> WHERE -> GROUP BY -> HAVING -> SELECT -> ORDER BY -> LIMIT

### 2. 多表查询 (Joins)

- **INNER JOIN**: 内连接，两边都有才显示。
- **LEFT JOIN**: 左连接，包括左表所有行，右表没有的显示 NULL。
- **RIGHT JOIN**: 右连接。
- **UNION / UNION ALL**: 合并多个查询结果集 (注意列数必须一致)。

### 3. 常用函数

- **字符串**: `CONCAT`, `SUBSTRING`, `LENGTH`, `UPPER/LOWER`
- **日期**: `NOW()`, `DATE_FORMAT()`, `DATEDIFF()`
- **逻辑**: `IFNULL()`, `CASE WHEN ... THEN ... ELSE ... END`

### 4. 事务 (Transaction)

- **四大特性 (ACID)**: 原子性 (Atomicity)、一致性 (Consistency)、隔离性 (Isolation)、持久性 (Durability)。
- **操作**: `START TRANSACTION` / `BEGIN`, `COMMIT`, `ROLLBACK`.
- **并发问题**: 脏读、不可重复读、幻读。
- **隔离级别**:
  - `READ UNCOMMITTED`
  - `READ COMMITTED` (Oracle 默认)
  - `REPEATABLE READ` (MySQL 默认)
  - `SERIALIZABLE`

---

## ⚡ 阶段三：高级优化 (Performance & Architecture)

### 1. 存储引擎 (Storage Engines)

| 特性         | InnoDB (默认)     | MyISAM (旧版)       |
| :----------- | :---------------- | :------------------ |
| **事务**     | 支持 (ACID)       | 不支持              |
| **锁粒度**   | 行级锁 (Row Lock) | 表级锁 (Table Lock) |
| **外键**     | 支持              | 不支持              |
| **MVCC**     | 支持              | 不支持              |
| **崩溃恢复** | 优秀              | 差                  |

### 2. 索引 (Indexing) - 数据库的目录

- **数据结构**: B+Tree (非叶子节点存索引，叶子节点存数据，叶子节点之间双向链表)。
- **分类**:
  - **主键索引 (Primary)**: 唯一且非空，聚集索引 (Clustered Index)。
  - **唯一索引 (Unique)**: 唯一，允许为空。
  - **普通索引 (Normal)**: 加速查询。
  - **组合索引 (Composite)**: 多个字段联合建立索引。
- **最佳实践**:
  - 经常作为查询条件 (`WHERE`)、排序 (`ORDER BY`)、分组 (`GROUP BY`) 的字段建立索引。
  - 索引不是越多越好，维护索引需要成本。

### 3. SQL 调优 (Optimization)

- **EXPLAIN 命令**: 分析 SQL 执行计划 (重点关注 `type`, `key`, `rows`, `Extra`)。
  - `type` 优劣: `system > const > eq_ref > ref > range > index > ALL` (ALL 代表全表扫描，尽量避免)。
- **最左前缀法则**: 组合索引 `(a,b,c)`，查询必须包含 `a` 才能用到索引。
- **索引失效场景**:
  - 在索引列上做运算或函数操作。
  - 字符串不加单引号 (类型隐式转换)。
  - `LIKE '%abc'` (模糊查询以 % 开头)。
  - `OR` 连接的条件，如果有一侧没有索引，涉及的索引都会失效。

### 4. 锁机制 (Locking)

- **乐观锁**: 通过版本号 (`version`) 控制。
- **悲观锁**:
  - **共享锁 (S 锁)**: 读锁，`SELECT ... LOCK IN SHARE MODE`。
  - **排他锁 (X 锁)**: 写锁，`SELECT ... FOR UPDATE`。
- **死锁 (Deadlock)**: 两个事务互相等待对方释放锁。

### 5. MVCC (多版本并发控制)

InnoDB 实现 `REPEATABLE READ` 隔离级别且不加锁的核心机制，通过 Read View 和 Undo Log 实现。

---

## 🆚 阶段四：MySQL 5.7 vs 8.0 关键特性对比

随着 MySQL 8.0 的普及，了解版本差异对于新项目开发和旧项目迁移至关重要。

### 1. 默认字符集 (Default Charset)

- **5.7**: 默认为 `latin1` (这也导致了很多中文乱码问题)，通常需要手动修改为 `utf8mb4`。
- **8.0**: 默认为 `utf8mb4`。`utf8mb4` 是 `utf8` 的超集，支持 Emoji 表情，是现代 Web 的标准。

### 2. 身份认证与安全性

- **5.7**: 默认使用 `mysql_native_password` 插件。
- **8.0**: 默认使用 `caching_sha2_password`。这提供了更高的安全性，但可能导致旧版客户端连接失败（报错 `Authentication plugin 'caching_sha2_password' cannot be loaded`），解决方法是升级驱动或将用户插件改回 native。

### 3. 窗口函数 (Window Functions) - 🔥 核心升级

MySQL 8.0 终于支持了标准的窗口函数，这让做排名、同比环比统计变得极其简单。

- **语法**: `函数() OVER (PARTITION BY ... ORDER BY ...)`
- **常用函数**:
  - `ROW_NUMBER()`: 1, 2, 3, 4 (不并列，连续)
  - `RANK()`: 1, 2, 2, 4 (并列，跳号)
  - `DENSE_RANK()`: 1, 2, 2, 3 (并列，不跳号)
  - `NTILE(n)`: 数据切片

### 4. 公用表表达式 (CTE) - 🔥 核心升级

- **功能**: 使用 `WITH` 子句定义临时的结果集，可以在 SELECT 中多次引用。支持递归 (Recursive CTE)，非常适合处理树形结构数据（如部门层级、菜单栏）。
- **示例**:
  ```sql
  WITH cte_user AS (
      SELECT id, name FROM users WHERE status = 1
  )
  SELECT * FROM cte_user;
  ```

### 5. DDL 原子性 (Atomic DDL)

- **5.7**: DDL 操作（如 `DROP TABLE`）不是原子的。如果删除多个表时中途失败，可能导致部分表被删除，部分未删除。
- **8.0**: 支持原子 DDL。Data Dictionary 重构，所有元数据存在 InnoDB 引擎表中，保证 DDL 操作要么全成功，要么全失败（回滚）。

### 6. JSON 支持增强

- **5.7**: 开始支持 JSON 类型。
- **8.0**: 极大增强。增加了 `JSON_TABLE()` (将 JSON 转为关系表)、`->>` 操作符 (等同于 `JSON_UNQUOTE(JSON_EXTRACT(...))`)，以及对 JSON 列的局部更新优化。

### 7. 索引增强

- **隐藏索引 (Invisible Indexes)**: 可以将索引设为不可见，用来测试删除索引对性能的影响，而无需真正删除它。
- **降序索引 (Descending Indexes)**: 8.0 真正支持了降序索引 (`DESC`)，在多列混合排序场景下（如 `ORDER BY a ASC, b DESC`）性能提升明显。
