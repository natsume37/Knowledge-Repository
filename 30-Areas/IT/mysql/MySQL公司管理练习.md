# MySQL 基础实战：公司管理系统练习

## 1. 数据库与表结构初始化

请将以下 SQL 代码直接复制到 MySQL 客户端中执行。它会创建一个名为 `company_db` 的数据库，并建立 4 张关联表：部门、员工、薪资、项目。

```sql
-- 如果存在则删除旧库（慎用！）
DROP DATABASE IF EXISTS company_db;
CREATE DATABASE company_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE company_db;

-- 1. 部门表 (Departments)
CREATE TABLE departments (
    dept_id INT PRIMARY KEY AUTO_INCREMENT COMMENT '部门ID',
    dept_name VARCHAR(50) NOT NULL COMMENT '部门名称',
    location VARCHAR(100) COMMENT '办公地点'
);

-- 2. 员工表 (Employees)
CREATE TABLE employees (
    emp_id INT PRIMARY KEY AUTO_INCREMENT COMMENT '员工ID',
    name VARCHAR(50) NOT NULL COMMENT '姓名',
    gender ENUM('男', '女') DEFAULT '男' COMMENT '性别',
    join_date DATE NOT NULL COMMENT '入职日期',
    job_title VARCHAR(50) COMMENT '职位',
    dept_id INT COMMENT '所属部门ID',
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
);

-- 3. 薪资表 (Salaries) - 记录历史薪资
CREATE TABLE salaries (
    id INT PRIMARY KEY AUTO_INCREMENT,
    emp_id INT NOT NULL COMMENT '员工ID',
    amount DECIMAL(10, 2) NOT NULL COMMENT '薪资数额',
    pay_date DATE NOT NULL COMMENT '发放日期',
    FOREIGN KEY (emp_id) REFERENCES employees(emp_id)
);

-- 4. 项目表 (Projects) - 员工与项目的多对多关系
CREATE TABLE projects (
    proj_id INT PRIMARY KEY AUTO_INCREMENT,
    proj_name VARCHAR(100) NOT NULL COMMENT '项目名称',
    budget DECIMAL(12, 2) DEFAULT 0 COMMENT '项目预算'
);

-- 关联表：记录员工参与的项目
CREATE TABLE employee_projects (
    emp_id INT,
    proj_id INT,
    role VARCHAR(50) COMMENT '项目中担任的角色',
    PRIMARY KEY (emp_id, proj_id),
    FOREIGN KEY (emp_id) REFERENCES employees(emp_id),
    FOREIGN KEY (proj_id) REFERENCES projects(proj_id)
);
```

---

## 2. 模拟数据插入

```sql
-- 插入部门
INSERT INTO departments (dept_name, location) VALUES
('研发部', '北京海淀'),
('市场部', '上海黄浦'),
('人事部', '北京朝阳'),
('财务部', '深圳南山');

-- 插入员工
INSERT INTO employees (name, gender, join_date, job_title, dept_id) VALUES
('张三', '男', '2021-03-15', '后端工程师', 1),
('李四', '女', '2020-06-20', '产品经理', 1),
('王五', '男', '2019-11-01', '市场总监', 2),
('赵六', '女', '2022-01-10', 'HR专员', 3),
('孙七', '男', '2023-05-20', '实习生', 1),
('周八', '女', '2018-09-01', '财务主管', 4);

-- 插入薪资记录 (每人模拟发了两个月工资)
INSERT INTO salaries (emp_id, amount, pay_date) VALUES
(1, 15000.00, '2023-11-10'), (1, 15500.00, '2023-12-10'),
(2, 18000.00, '2023-11-10'), (2, 18000.00, '2023-12-10'),
(3, 25000.00, '2023-11-10'), (3, 26000.00, '2023-12-10'),
(4, 8000.00, '2023-11-10'), (4, 8000.00, '2023-12-10'),
(5, 3000.00, '2023-11-10'), (5, 3000.00, '2023-12-10'),
(6, 20000.00, '2023-11-10'), (6, 20000.00, '2023-12-10');

-- 插入项目
INSERT INTO projects (proj_name, budget) VALUES
('企业官网改版', 50000.00),
('App v2.0开发', 2000000.00),
('年度市场推广', 800000.00);

-- 分配项目人员
INSERT INTO employee_projects (emp_id, proj_id, role) VALUES
(1, 2, '核心开发'),
(2, 2, '项目负责人'),
(5, 2, '测试助理'),
(1, 1, '技术支持'),
(3, 3, '总策划');
```

---

## 3. 基础练习题 (尝试自己写 SQL！)

### 3.1 基础查询查询 (SELECT)

1.  查询所有员工的姓名和职位。
2.  查询入职时间在 2021 年之后（含 2021）的员工信息。
3.  查询薪资大于 20000 的薪资发放记录。

### 3.2 聚合与分组 (GROUP BY)

1.  统计每个部门有多少名员工？
2.  计算全公司的平均薪资水平（基于最近一个月 2023-12-10 的数据）。
3.  哪个项目的预算最高？

### 3.3 多表连接 (JOIN) - 重点！

1.  **内连接**: 查询“张三”所在的部门名称和办公地点。
2.  **左连接**: 列出所有部门的名称，以及该部门下的员工人数（注意：即使部门没有员工也要显示 0）。
3.  **多对多**: 查询“App v2.0 开发”项目中，参与的所有员工姓名及其在项目中的角色。

---

## 4. 参考答案 (写完再看)

<details>
<summary>点击查看 SQL 答案</summary>

```sql
-- 3.1 基础查询
SELECT name, job_title FROM employees;
SELECT * FROM employees WHERE join_date >= '2021-01-01';
SELECT * FROM salaries WHERE amount > 20000;

-- 3.2 聚合与分组
-- 统计部门人数
SELECT dept_id, COUNT(*) as emp_count FROM employees GROUP BY dept_id;
-- 关联部门名显示更好：
SELECT d.dept_name, COUNT(e.emp_id)
FROM departments d
LEFT JOIN employees e ON d.dept_id = e.dept_id
GROUP BY d.dept_name;

-- 12月平均薪资
SELECT AVG(amount) FROM salaries WHERE pay_date = '2023-12-10';

-- 最高预算项目
SELECT proj_name, budget FROM projects ORDER BY budget DESC LIMIT 1;

-- 3.3 多表连接
-- 张三的部门
SELECT e.name, d.dept_name, d.location
FROM employees e
JOIN departments d ON e.dept_id = d.dept_id
WHERE e.name = '张三';

-- App v2.0开发的项目成员
SELECT p.proj_name, e.name, ep.role
FROM projects p
JOIN employee_projects ep ON p.proj_id = ep.proj_id
JOIN employees e ON ep.emp_id = e.emp_id
WHERE p.proj_name = 'App v2.0开发';
```

</details>
