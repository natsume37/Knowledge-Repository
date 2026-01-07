# MySQL 终极实战：打造商业级“读书笔记”系统

这是一个综合了 **数据库设计思维** 与 **MySQL 8.0 全特性** 的终极练习。

## 🏢 1. 架构设计思维 (Architecture Design)

既然是“面向商业”的架构，我们需要考虑以下关键点：

1.  **扩展性 (Extensibility)**: 书籍的属性千奇百怪（有的有译者，有的有副标题，有的有 ISBN-13），不能为每个属性建列。-> **解决方案：使用 `JSON` 类型存储元数据**。
2.  **软删除 (Soft Delete)**: 商业系统几乎不真删数据，而是标记删除，以便数据分析和误操作恢复。-> **解决方案：`is_deleted` 字段**。
3.  **状态流转**: 书籍状态（想读、在读、读完）。
4.  **性能**: 主键使用 `BIGINT`，关键查询字段（如 UserID, BookID）必须有索引。

---

## 🛠️ 2. 环境初始化

请执行以下 SQL 语句来构建你的“读书笔记”数据库。

```sql
DROP DATABASE IF EXISTS book_notes_app;
CREATE DATABASE book_notes_app DEFAULT CHARSET utf8mb4;
USE book_notes_app;

-- 1. 用户表 (Users)
CREATE TABLE users (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE COMMENT '用户名',
    email VARCHAR(100) NOT NULL UNIQUE COMMENT '邮箱',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_email (email)
) ENGINE=InnoDB COMMENT='用户核心表';

-- 2. 书籍表 (Books) - 使用 JSON 存储扩展信息
CREATE TABLE books (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(100) NOT NULL COMMENT '书名',
    author VARCHAR(50) NOT NULL COMMENT '作者',
    isbn CHAR(13) COMMENT 'ISBN号',

    -- 【扩展性设计】: 存放页数、出版社、译者、封面图URL等不确定字段
    extra_info JSON COMMENT '书籍扩展元数据',

    avg_rating DECIMAL(3,1) DEFAULT 0.0 COMMENT '缓存的平均分(冗余设计,避免频繁聚合)',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_title (title)
) ENGINE=InnoDB COMMENT='书籍库';

-- 3. 阅读记录表 (Readings) - 用户与书的关系
CREATE TABLE readings (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NOT NULL,
    book_id BIGINT UNSIGNED NOT NULL,
    status TINYINT DEFAULT 1 COMMENT '1:想读 2:在读 3:读完',
    my_rating TINYINT UNSIGNED COMMENT '个人评分 1-5',
    start_time DATETIME,
    finish_time DATETIME,

    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (book_id) REFERENCES books(id),
    UNIQUE KEY uk_user_book (user_id, book_id) -- 同一个人对同一本书只有一条记录
) ENGINE=InnoDB;

-- 4. 笔记表 (Notes)
CREATE TABLE notes (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    reading_id BIGINT UNSIGNED NOT NULL COMMENT '关联阅读记录',
    chapter_title VARCHAR(100) COMMENT '章节标题',
    content TEXT NOT NULL COMMENT '笔记内容',

    -- 【商业设计】: 笔记私密性与软删除
    is_private TINYINT DEFAULT 0 COMMENT '0:公开 1:私密',
    is_deleted TINYINT DEFAULT 0 COMMENT '0:有效 1:已删除',

    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (reading_id) REFERENCES readings(id),
    INDEX idx_reading (reading_id)
) ENGINE=InnoDB;

-- ==================== 插入模拟数据 ====================

-- 用户
INSERT INTO users (username, email) VALUES
('Alice', 'alice@example.com'),
('Bob', 'bob@example.com'),
('Charlie', 'charlie@example.com');

-- 书籍 (注意 JSON 格式)
INSERT INTO books (title, author, extra_info) VALUES
('MySQL必知必会', 'Ben Forta', '{"pages": 240, "publisher": "人民邮电出版社", "tags": ["技术", "数据库"]}'),
('三体', '刘慈欣', '{"pages": 302, "publisher": "重庆出版社", "tags": ["科幻", "小说"]}'),
('高性能MySQL', 'Baron', '{"pages": 800, "publisher": "电子工业出版社", "type": "Hardcore"}'),
('活着', '余华', '{"pages": 190, "tags": ["文学", "经典"]}');

-- 阅读关系 (Alice 读了 MySQL 和 三体, Bob 读了 三体)
INSERT INTO readings (user_id, book_id, status, my_rating) VALUES
(1, 1, 3, 5), -- Alice 读完 MySQL, 5分
(1, 2, 2, 4), -- Alice 在读 三体, 4分
(2, 2, 3, 5), -- Bob 读完 三体, 5分
(2, 3, 1, NULL), -- Bob 想读 高性能MySQL
(3, 1, 3, 3); -- Charlie 读完 MySQL, 3分

-- 笔记
INSERT INTO notes (reading_id, chapter_title, content, is_private) VALUES
(1, '第1章', 'SELECT 必须接 FROM', 0),
(1, '第3章', '排序使用 ORDER BY', 0),
(2, '黑暗森林', '不要回答！', 1), -- 私密笔记
(3, '第1章', '物理学不存在了', 0),
(5, '索引优化', '索引不是越多越好', 0);
```

---

## ⚔️ 3. 终极挑战 (The Challenge)

请完成以下 7 个任务，运用你学到的所有知识。

### 📌 任务 1: JSON 查询与元数据提取

有些书的 `extra_info` 里包含 `"publisher"` (出版社) 字段。
**目标**: 查询所有书籍的 `书名` 和 `出版社`。如果 JSON 中没有出版社，显示 "未知"。

> _考点: JSON_UNQUOTE, JSON_EXTRACT (->>)_

### 📌 任务 2: 软删除的日常维护

用户 Alice 删除了她的第一条笔记（假设 ID 为 1）。
**目标**:

1. 编写 SQL 逻辑删除该笔记。
2. 编写查询语句，查询 Alice (`user_id=1`) 所有 **有效的**、**公开的** 笔记内容及对应的书名。
   > _考点: UPDATE, JOIN, multi-table WHERE_

### 📌 任务 3: 复杂分组与聚合

商用系统通常需要展示“书籍热度榜”。
**目标**: 查询每本书的 **书名**、**阅读人数**、**平均评分**。结果按阅读人数降序排列。

> _考点: GROUP BY, COUNT, AVG, ORDER BY_

### 📌 任务 4: 窗口函数 (MySQL 8.0 特性)

我们需要做一个“阅读达人榜”。
**目标**: 对所有用户，按照“读完的书籍数量”进行排名。
要求显示：`用户名`, `读完数量`, `排名` (使用 `DENSE_RANK`, 并列时名次连续)。

> _考点: Window Functions, PARTITION vs GROUP_

### 📌 任务 5: CTE (公用表表达式) 分析

我们需要找出“笔记狂魔”。
**目标**: 使用 `WITH` (CTE) 语句。

1. 先定义一个 CTE `cte_note_counts`，计算每个用户的笔记总数。
2. 主查询从中选出笔记数大于 0 的用户，并显示 `用户名` 和 `笔记数`。
   > _考点: CTE 语法, 思路清晰化_

### 📌 任务 6: 索引优化 (Explain)

假设 `notes` 表有 1000 万条数据。我们经常执行如下查询：查询某个阅读记录 ID 下，有效的笔记。
SQL: `SELECT * FROM notes WHERE reading_id = 99 AND is_deleted = 0;`
**目标**:

1. 写出该查询的 `EXPLAIN` 语句。
2. 如果发现是全表扫描，请写出添加最优索引的语句。
   > _考点: 复合索引, Explain type 分析_

### 📌 任务 7: 事务控制 (ACID)

用户 Bob 给“三体”这本书打分时，系统需要同时做两件事：

1. 在 `readings` 表更新 Bob 的评分。
2. (模拟) 重新计算 `books` 表的 `avg_rating` (虽然真实场景可能是异步的，这里模拟同步事务)。
   **目标**: 写一个 Transaction 代码块，包含这两步操作。如果中间报错，需要回滚。

---

## ✅ 参考答案 (Hidden Answers)

<details>
<summary>点击查看任务 1 答案 (JSON)</summary>

```sql
SELECT
    title,
    IFNULL(extra_info->>'$.publisher', '未知') as publisher
FROM books;

-- 或者使用 MySQL 8.0 语法糖
-- extra_info->>'$.publisher' 等同于 JSON_UNQUOTE(JSON_EXTRACT(extra_info, '$.publisher'))
```

</details>

<details>
<summary>点击查看任务 2 答案 (软删除 & Join)</summary>

```sql
-- 1. 软删除
UPDATE notes SET is_deleted = 1 WHERE id = 1;

-- 2. 查询 Alice 的有效公开笔记
SELECT b.title, n.content
FROM users u
JOIN readings r ON u.id = r.user_id
JOIN notes n ON r.id = n.reading_id
JOIN books b ON r.book_id = b.id
WHERE u.id = 1
  AND n.is_deleted = 0 -- 过滤已删除
  AND n.is_private = 0; -- 过滤私密
```

</details>

<details>
<summary>点击查看任务 3 答案 (聚合)</summary>

```sql
SELECT
    b.title,
    COUNT(r.id) as read_count,
    ROUND(AVG(r.my_rating), 1) as avg_score
FROM books b
LEFT JOIN readings r ON b.id = r.book_id
GROUP BY b.id, b.title
ORDER BY read_count DESC;
```

</details>

<details>
<summary>点击查看任务 4 答案 (窗口函数)</summary>

```sql
SELECT
    u.username,
    COUNT(r.book_id) as books_finished,
    DENSE_RANK() OVER (ORDER BY COUNT(r.book_id) DESC) as `rank`
FROM users u
JOIN readings r ON u.id = r.user_id
WHERE r.status = 3 -- 3代表读完
GROUP BY u.id, u.username;
```

</details>

<details>
<summary>点击查看任务 5 答案 (CTE)</summary>

```sql
WITH cte_note_counts AS (
    SELECT r.user_id, COUNT(n.id) as note_total
    FROM readings r
    JOIN notes n ON r.id = n.reading_id
    WHERE n.is_deleted = 0
    GROUP BY r.user_id
)
SELECT u.username, c.note_total
FROM users u
JOIN cte_note_counts c ON u.id = c.user_id
WHERE c.note_total > 0;
```

</details>

<details>
<summary>点击查看任务 6 答案 (索引)</summary>

```sql
-- 分析
EXPLAIN SELECT * FROM notes WHERE reading_id = 1 AND is_deleted = 0;

-- 如果 type 是 ALL，说明需要索引。
-- 最优索引：覆盖查询条件
CREATE INDEX idx_reading_deleted ON notes(reading_id, is_deleted);
```

</details>

<details>
<summary>点击查看任务 7 答案 (Transaction)</summary>

```sql
START TRANSACTION;

-- 1. 更新个人评分
UPDATE readings SET my_rating = 5 WHERE user_id = 2 AND book_id = 1;

-- 2. (模拟) 更新书籍总分，这里简化为写死，实际需计算
UPDATE books SET avg_rating = 4.8 WHERE id = 1;

-- 如果无误
COMMIT;
-- 如果报错
-- ROLLBACK;
```

</details>
