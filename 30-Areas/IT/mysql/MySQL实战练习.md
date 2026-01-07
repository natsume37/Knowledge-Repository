# MySQL 实战演练场 (升级版)

这里提供了一套经典的数据模型供你练习。请按照顺序执行下面的 SQL 语句来初始化环境。

## 🛠️ 0. 环境准备 (初始化数据)

我们使用经典的 **“学生-老师-课程-成绩”** 模型。

```sql
-- 创建数据库
CREATE DATABASE IF NOT EXISTS sql_practice DEFAULT CHARSET utf8mb4;
USE sql_practice;

-- 1. 学生表 (Student)
CREATE TABLE Student (
    SId VARCHAR(10) PRIMARY KEY COMMENT '学生ID',
    Sname VARCHAR(10) COMMENT '学生姓名',
    Sage DATETIME COMMENT '出生日期',
    Ssex VARCHAR(10) COMMENT '性别'
);

-- 2. 教师表 (Teacher)
CREATE TABLE Teacher (
    TId VARCHAR(10) PRIMARY KEY COMMENT '教师ID',
    Tname VARCHAR(10) COMMENT '教师姓名'
);

-- 3. 课程表 (Course)
CREATE TABLE Course (
    CId VARCHAR(10) PRIMARY KEY COMMENT '课程ID',
    Cname VARCHAR(10) COMMENT '课程名称',
    TId VARCHAR(10) COMMENT '教师ID'
);

-- 4. 成绩表 (SC)
CREATE TABLE SC (
    SId VARCHAR(10) COMMENT '学生ID',
    CId VARCHAR(10) COMMENT '课程ID',
    score DECIMAL(18,1) COMMENT '分数'
);

-- 插入数据
INSERT INTO Student VALUES('01' , '赵雷' , '1990-01-01' , '男');
INSERT INTO Student VALUES('02' , '钱电' , '1990-12-21' , '男');
INSERT INTO Student VALUES('03' , '孙风' , '1990-05-20' , '男');
INSERT INTO Student VALUES('04' , '李云' , '1990-08-06' , '男');
INSERT INTO Student VALUES('05' , '周梅' , '1991-12-01' , '女');
INSERT INTO Student VALUES('06' , '吴兰' , '1992-03-01' , '女');
INSERT INTO Student VALUES('07' , '郑竹' , '1989-07-01' , '女');
INSERT INTO Student VALUES('08' , '王菊' , '1990-01-20' , '女');

INSERT INTO Teacher VALUES('01' , '张三');
INSERT INTO Teacher VALUES('02' , '李四');
INSERT INTO Teacher VALUES('03' , '王五');

INSERT INTO Course VALUES('01' , '语文' , '02');
INSERT INTO Course VALUES('02' , '数学' , '01');
INSERT INTO Course VALUES('03' , '英语' , '03');

INSERT INTO SC VALUES('01' , '01' , 80);
INSERT INTO SC VALUES('01' , '02' , 90);
INSERT INTO SC VALUES('01' , '03' , 99);
INSERT INTO SC VALUES('02' , '01' , 70);
INSERT INTO SC VALUES('02' , '02' , 60);
INSERT INTO SC VALUES('02' , '03' , 80);
INSERT INTO SC VALUES('03' , '01' , 80);
INSERT INTO SC VALUES('03' , '02' , 80);
INSERT INTO SC VALUES('03' , '03' , 80);
INSERT INTO SC VALUES('04' , '01' , 50);
INSERT INTO SC VALUES('04' , '02' , 30);
INSERT INTO SC VALUES('04' , '03' , 20);
INSERT INTO SC VALUES('05' , '01' , 76);
INSERT INTO SC VALUES('05' , '02' , 87);
INSERT INTO SC VALUES('06' , '01' , 31);
INSERT INTO SC VALUES('06' , '03' , 34);
INSERT INTO SC VALUES('07' , '02' , 89);
INSERT INTO SC VALUES('07' , '03' , 98);
```

---

## 📝 1. 基础篇练习 (单表查询)

### Q1: 查询所有男生的名单

<details>
<summary>点击查看参考答案</summary>

```sql
SELECT * FROM Student WHERE Ssex = '男';
```

</details>

### Q2: 查询名字中带有 "风" 字的学生

<details>
<summary>点击查看参考答案</summary>

```sql
SELECT * FROM Student WHERE Sname LIKE '%风%';
```

</details>

### Q3: 查询 "02" 号课程及格（分数 >= 60）的学生学号和分数，并按分数降序排列

<details>
<summary>点击查看参考答案</summary>

```sql
SELECT SId, score
FROM SC
WHERE CId = '02' AND score >= 60
ORDER BY score DESC;
```

</details>

---

## 🚀 2. 进阶篇练习 (多表 JOIN & 聚合)

### Q4: 查询每门课程的平均成绩，结果显示：课程 ID，平均成绩

<details>
<summary>点击查看参考答案</summary>

```sql
SELECT CId, AVG(score) as avg_score
FROM SC
GROUP BY CId;
```

</details>

### Q5: 查询所有学生的学号、姓名、选课数、总成绩 (注意：没选课的学生也要显示)

> 💡 提示：需要使用 LEFT JOIN

<details>
<summary>点击查看参考答案</summary>

```sql
SELECT
    s.SId,
    s.Sname,
    COUNT(sc.CId) as course_count,
    SUM(sc.score) as total_score
FROM Student s
LEFT JOIN SC sc ON s.SId = sc.SId
GROUP BY s.SId, s.Sname;
```

</details>

### Q6: 查询学过 "张三" 老师授课的同学的信息

> 💡 提示：这里涉及 4 张表关联：Student -> SC -> Course -> Teacher

<details>
<summary>点击查看参考答案</summary>

```sql
SELECT s.*
FROM Student s
JOIN SC sc ON s.SId = sc.SId
JOIN Course c ON sc.CId = c.CId
JOIN Teacher t ON c.TId = t.TId
WHERE t.Tname = '张三';
```

</details>

### Q7: 查询同时学过 "01" 课程和 "02" 课程的情况

> 💡 提示：可以将 SC 表自连接 (Self Join)

<details>
<summary>点击查看参考答案</summary>

```sql
SELECT a.SId
FROM
    (SELECT SId, score FROM SC WHERE CId='01') a
JOIN
    (SELECT SId, score FROM SC WHERE CId='02') b
ON a.SId = b.SId;
```

</details>

### Q8: 查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩

<details>
<summary>点击查看参考答案</summary>

```sql
SELECT s.SId, s.Sname, AVG(sc.score) as avg_score
FROM Student s
JOIN SC sc ON s.SId = sc.SId
WHERE sc.score < 60
GROUP BY s.SId, s.Sname
HAVING COUNT(sc.CId) >= 2;
```

</details>

---

## ⚡ 3. 高级篇练习 (窗口函数 & 复杂查询)

> _这部分特别针对 MySQL 8.0 新特性_

### Q9: 按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩

> 💡 提示：使用窗口函数计算平均分，可以避免复杂的 Group By

<details>
<summary>点击查看参考答案</summary>

```sql
SELECT
    s.SId,
    s.Sname,
    c.Cname,
    sc.score,
    AVG(sc.score) OVER (PARTITION BY s.SId) as avg_score
FROM Student s
JOIN SC sc ON s.SId = sc.SId
JOIN Course c ON sc.CId = c.CId
ORDER BY avg_score DESC, s.SId;
```

</details>

### Q10: 查询各科成绩前三名的记录 (分组取 Top N)

> 💡 提示：这是面试必考题，使用 `RANK()` 或 `DENSE_RANK()`

<details>
<summary>点击查看参考答案</summary>

```sql
SELECT * FROM (
    SELECT
        sc.CId,
        sc.SId,
        sc.score,
        DENSE_RANK() OVER (PARTITION BY sc.CId ORDER BY sc.score DESC) as rk
    FROM SC sc
) t
WHERE t.rk <= 3;
```

</details>

---

## 🛡️ 4. 优化篇练习 (索引与性能)

假设数据量变大（比如 SC 表有 100 万行），我们来练习索引优化。

### Q11: 解释计划分析 (Explain)

假设我们要频繁查询 `SELECT * FROM SC WHERE CId = '01' AND score > 80;`

1. 请写出给 SC 表添加合适索引的语句。
2. 使用 `EXPLAIN` 分析该语句。

<details>
<summary>点击查看参考答案</summary>

```sql
-- 创建联合索引，注意顺序：等值查询在前，范围查询在后
CREATE INDEX idx_cid_score ON SC(CId, score);

-- 分析
EXPLAIN SELECT * FROM SC WHERE CId = '01' AND score > 80;

-- 只有 CId 是索引生效的（因为 score 是范围查询），但在这里它也能用到索引。
-- 最优索引应该是 (CId, score)，这样可以利用索引下推。
```

</details>

### Q12: 最左前缀原则测试

假设已有索引 `idx_sid_cid_score` (SId, CId, score)。
请问以下查询是否用到了索引？

1. `SELECT * FROM SC WHERE SId = '01';`
2. `SELECT * FROM SC WHERE score = 90;`
3. `SELECT * FROM SC WHERE SId = '01' AND CId = '02';`

<details>
<summary>点击查看参考答案</summary>

1. **用到**。符合最左前缀。
2. **没用到**。跳过了前两个字段，直接查最后一个，违反最左前缀，会全表扫描 (type=ALL)。
3. **用到**。符合最左前缀。
</details>
