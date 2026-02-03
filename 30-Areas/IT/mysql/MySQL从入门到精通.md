# MySQL 从入门到精通全栈指南

本文档旨在提供一个从基础到高阶的 MySQL 知识体系，涵盖 DDL/DML/DQL/DCL、高级特性及性能优化。

---

## 🟢 第一阶段：基础语法 (CRUD 与 表结构)

### 1. DDL (Data Definition Language) - 数据定义

用于定义数据库对象（库、表、字段）。

- **创建数据库**
  ```sql
  CREATE DATABASE IF NOT EXISTS my_db DEFAULT CHARSET utf8mb4;
  ```
- **创建表**
  ```sql
  CREATE TABLE users (
      id INT PRIMARY KEY AUTO_INCREMENT COMMENT '主键',
      username VARCHAR(50) NOT NULL COMMENT '用户名',
      age INT DEFAULT 18 COMMENT '年龄',
      created_at DATETIME DEFAULT CURRENT_TIMESTAMP
  ) COMMENT '用户表';
  ```
- **修改表结构**
  ```sql
  ALTER TABLE users ADD COLUMN email VARCHAR(100); -- 添加字段
  ALTER TABLE users MODIFY COLUMN age TINYINT;     -- 修改类型
  DROP TABLE IF EXISTS users;                      -- 删除表
  ```

### 2. DML (Data Manipulation Language) - 数据操作

用于对表中的数据进行增、删、改。

- **插入 (Insert)**
  ```sql
  INSERT INTO users (username, age) VALUES ('Alice', 25), ('Bob', 30);
  ```
- **更新 (Update)**
  ```sql
  UPDATE users SET age = 26 WHERE username = 'Alice';
  -- ⚠️ 注意：不加 WHERE 会更新全表！
  ```
- **删除 (Delete)**
  ```sql
  DELETE FROM users WHERE id = 1;
  -- TRUNCATE TABLE users; -- 清空表（重置自增ID，属于DDL）
  ```

### 3. DQL (Data Query Language) - 数据查询

数据库最核心的功能。

- **基础查询**
  ```sql
  SELECT name, age FROM users WHERE age > 20 ORDER BY age DESC LIMIT 10;
  ```
- **聚合函数**
  - `COUNT(*)`, `SUM()`, `AVG()`, `MAX()`, `MIN()`
- **分组 (GROUP BY & HAVING)**
  ```sql
  -- 查询每个部门平均薪资大于 10k 的部门
  SELECT dept_id, AVG(salary)
  FROM employees
  GROUP BY dept_id
  HAVING AVG(salary) > 10000;
  ```
- **连接查询 (JOIN)**
  - **INNER JOIN**: 交集。
  - **LEFT JOIN**: 左表全查，右表匹配不到补 NULL。
  - **RIGHT JOIN**: 右表全查。

### 4. DCL (Data Control Language) - 数据控制

用于管理用户权限。

```sql
-- 创建用户
CREATE USER 'dev_user'@'%' IDENTIFIED BY 'Password123';
-- 授权
GRANT SELECT, INSERT ON my_db.* TO 'dev_user'@'%';
-- 撤销权限
REVOKE INSERT ON my_db.* FROM 'dev_user'@'%';
-- 刷新权限
FLUSH PRIVILEGES;
```

---

## 🟡 第二阶段：核心机制 (事务、索引、锁)

### 1. 事务 (Transaction)

保证一组操作要么全部成功，要么全部失败。

- **ACID 特性**:

  - **A (Atomicity 原子性)**: 即使断电，操作也是原子的（Undo Log 实现）。
  - **C (Consistency 一致性)**: 数据从一个合法状态到另一个合法状态。
  - **I (Isolation 隔离性)**: 事务互不干扰（MVCC + 锁 实现）。
  - **D (Durability 持久性)**: 提交后永久保存（Redo Log 实现）。

- **并发问题与隔离级别**:
  | 隔离级别 | 脏读 (Dirty Read) | 不可重复读 (Non-Repeatable Read) | 幻读 (Phantom Read) |
  | :--- | :---: | :---: | :---: |
  | Read Uncommitted | ✅ | ✅ | ✅ |
  | **Read Committed (RC)** | ❌ | ✅ | ✅ |
  | **Repeatable Read (RR)** (默认) | ❌ | ❌ | ❌ (MVCC 解决大部分) |
  | Serializable | ❌ | ❌ | ❌ |

- **操作演示**:
  ```sql
  START TRANSACTION;
  UPDATE account SET money = money - 100 WHERE id = A;
  UPDATE account SET money = money + 100 WHERE id = B;
  COMMIT; -- 或 ROLLBACK;
  ```

### 2. 索引 (Index) - 性能之王

本质是 **B+Tree** 数据结构，用于快速查找。

- **分类**:

  - **主键索引**: 唯一且非空，叶子节点存整行数据（聚簇索引）。
  - **唯一索引 (UNIQUE)**: 值必须唯一。
  - **普通索引**: 普通的加速查询。
  - **联合索引**: 多个字段组成，需遵循 **最左前缀法则**。

- **创建索引**:
  ```sql
  CREATE INDEX idx_name_age ON users(username, age);
  ```
- **最左前缀法则**:
  如果索引是 `(a, b, c)`:
  - `WHERE a = 1 AND b = 2` -> ✅ 走索引
  - `WHERE a = 1 AND c = 3` -> ✅ 只有 a 走索引
  - `WHERE b = 2` -> ❌ 不走索引

### 3. 锁 (Lock)

- **全局锁**: 锁住整个库（如全库备份）。
- **表级锁**: 锁整张表（MyISAM 默认，InnoDB 在无索引更新时也会退化为表锁）。
- **行级锁 (InnoDB)**:
  - **共享锁 (读锁/S 锁)**: `SELECT ... LOCK IN SHARE MODE;`
  - **排他锁 (写锁/X 锁)**: `UPDATE`, `DELETE`, `INSERT`, `SELECT ... FOR UPDATE;`
  - **死锁**: 两个事务互相等待对方释放资源。

---

## 🔴 第三阶段：高阶特性

### 1. 视图 (View)

虚拟表，保存的是 SQL 逻辑，不预存数据。

```sql
CREATE VIEW v_high_salary_emp AS
SELECT * FROM employees WHERE salary > 20000;
```

### 2. 存储过程 (Stored Procedure)

预编译的 SQL 集合。

```sql
CREATE PROCEDURE AddUser(IN p_name VARCHAR(20))
BEGIN
    INSERT INTO users(name) VALUES(p_name);
END;
```

### 3. 触发器 (Trigger)

在 Insert/Update/Delete 之前或之后自动执行。

```sql
CREATE TRIGGER sync_log AFTER INSERT ON users
FOR EACH ROW
BEGIN
    INSERT INTO operation_logs(op_type, user_id) VALUES ('insert', NEW.id);
END;
```

### 4. 窗口函数 (Window Functions) (MySQL 8.0+)

用于解决“组内排名”、“累计求和”等问题，不合并行。
语法：`函数() OVER (PARTITION BY ... ORDER BY ...)`

- **排名函数**:

  - `RANK()`: 1, 1, 3 (跳过排名)
  - `DENSE_RANK()`: 1, 1, 2 (不跳过)
  - `ROW_NUMBER()`: 1, 2, 3 (自然序)

  ```sql
  -- 查询每个部门薪资最高的员工排名
  SELECT name, dept_id, salary,
         RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) as ranking
  FROM employees;
  ```

---

## 🚀 第四阶段：SQL 性能优化

### 1. Explain 执行计划

在 SELECT 前加 `EXPLAIN` 查看执行情况。

- **type**: `system` > `const` > `eq_ref` > `ref` > `range` > `index` > `all` (全表扫描，尽量避免)。
- **key**: 实际用到的索引。
- **rows**: 扫描行数。
- **Extra**:
  - `Using filesort`: 索引无法满足排序，需优化。
  - `Using temporary`: 使用了临时表，需优化。
  - `Using index`: 覆盖索引，不需回表，性能好。

### 2. 索引失效场景 (避坑指南)

1.  **在索引列上做运算**: `WHERE year(create_time) = 2024` (应改为范围查询)。
2.  **字符串不加引号**: 导致类型隐式转换。
3.  **模糊查询左边带%**: `LIKE '%abc'` (最左匹配失效)。
4.  **OR 只有一边有索引**: 必须两边都有索引。
5.  **不满足最左前缀法则**。

### 3. 深分页优化

当 `LIMIT 1000000, 10` 时，MySQL 需排序前 1000010 条记录。

- **优化方案**: 利用覆盖索引 + 子查询。

  ```sql
  -- 慢
  SELECT * FROM users LIMIT 1000000, 10;

  -- 快 (先只查ID，走覆盖索引，再根据ID查详情)
  SELECT * FROM users u
  INNER JOIN (SELECT id FROM users LIMIT 1000000, 10) tmp
  ON u.id = tmp.id;
  ```
