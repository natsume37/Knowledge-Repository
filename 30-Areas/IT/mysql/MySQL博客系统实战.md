# MySQL 博客系统实战：从范式到递归

上一章的“读书笔记”系统展示了 JSON 的灵活性。本章我们将回归 **经典关系型数据库设计** (RDBMS)，打造一个 **完全范式化 (Normalized)** 的博客系统。

我们将重点练习：**多表关联 (Joins)**、**多对多关系 (Many-to-Many)**、**递归查询 (Recursive CTE)** 以及 **传统的索引优化**。

---

## 🏗️ 1. 场景与架构 (Scenario)

我们需要设计一个支持多级分类、文章标签、树形评论的博客系统。

**关键约束 (Constraints)**：

1.  **严谨的结构**：不使用 JSON，所有属性必须独立建列。
2.  **多级分类**：分类下可以有子分类（如 `技术` -> `后端` -> `Java`）。
3.  **多对多标签**：一篇文章可以有多个标签，一个标签可以对应多篇文章。
4.  **数据完整性**：使用外键约束 (Foreign Keys)。

---

## 🛠️ 2. 环境初始化 (Schema Setup)

```sql
DROP DATABASE IF EXISTS blog_system;
CREATE DATABASE blog_system DEFAULT CHARSET utf8mb4;
USE blog_system;

-- 1. 用户表
CREATE TABLE users (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE
) ENGINE=InnoDB;

-- 2. 分类表 (无限级分类设计 - 邻接表模型)
CREATE TABLE categories (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    parent_id INT UNSIGNED DEFAULT NULL COMMENT '父分类ID, NULL表示顶级',
    FOREIGN KEY (parent_id) REFERENCES categories(id)
) ENGINE=InnoDB;

-- 3. 文章表
CREATE TABLE posts (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id INT UNSIGNED NOT NULL,
    category_id INT UNSIGNED NOT NULL,
    title VARCHAR(200) NOT NULL,
    content TEXT NOT NULL,
    status ENUM('draft', 'published', 'archived') DEFAULT 'draft',
    view_count INT UNSIGNED DEFAULT 0,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (category_id) REFERENCES categories(id)
) ENGINE=InnoDB;

-- 4. 标签表
CREATE TABLE tags (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL UNIQUE
) ENGINE=InnoDB;

-- 5. 文章-标签关联表 (多对多核心)
CREATE TABLE post_tags (
    post_id INT UNSIGNED NOT NULL,
    tag_id INT UNSIGNED NOT NULL,
    PRIMARY KEY (post_id, tag_id),
    FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
    FOREIGN KEY (tag_id) REFERENCES tags(id) ON DELETE CASCADE
) ENGINE=InnoDB;

-- 6. 评论表 (树形结构)
CREATE TABLE comments (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    post_id INT UNSIGNED NOT NULL,
    user_id INT UNSIGNED NOT NULL,
    parent_id INT UNSIGNED DEFAULT NULL COMMENT '回复哪条评论',
    content VARCHAR(500) NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (parent_id) REFERENCES comments(id)
) ENGINE=InnoDB;

-- ==================== 数据模拟 ====================

INSERT INTO users (username, email) VALUES ('Admin', 'admin@blog.com'), ('Reader1', 'r1@test.com');

INSERT INTO categories (id, name, parent_id) VALUES
(1, '技术', NULL),
(2, '生活', NULL),
(3, '后端', 1),
(4, 'MySQL', 3),
(5, '随笔', 2);
-- 结构: 技术 > 后端 > MySQL

INSERT INTO tags (name) VALUES ('干货'), ('教程'), ('心情'), ('数据库');

INSERT INTO posts (user_id, category_id, title, content, status, view_count, created_at) VALUES
(1, 4, 'MySQL 索引原理', 'B+树...', 'published', 1024, '2023-01-01 10:00:00'),
(1, 4, 'Join 的七种写法', 'Inner, Left...', 'published', 500, '2023-01-02 10:00:00'),
(1, 5, '今天天气不错', '出去玩...', 'draft', 0, '2023-01-03 10:00:00');

-- 关联标签: 'MySQL索引原理' 属于 '干货', '数据库', '教程'
INSERT INTO post_tags (post_id, tag_id) VALUES (1, 1), (1, 4), (1, 2), (2, 2), (2, 4);

-- 评论
INSERT INTO comments (post_id, user_id, parent_id, content) VALUES
(1, 2, NULL, '写得真好!'), -- ID: 1
(1, 1, 1, '谢谢支持');    -- ID: 2 (回复 ID 1)
```

---

## 🟢 3. 初级挑战 (Level 1: Basic Join)

**目标**: 熟悉标准的多表查询。

1.  **查询文章列表**: 显示 `文章标题`, `作者名`, `分类名`, `发布时间`。
    - 要求: 仅显示 `published` 状态的文章。
2.  **标签查询**: 找出拥有 "数据库" 标签的所有文章标题。

<details>
<summary>点击查看参考答案</summary>

```sql
-- 1. 文章列表
SELECT p.title, u.username, c.name as category, p.created_at
FROM posts p
JOIN users u ON p.user_id = u.id
JOIN categories c ON p.category_id = c.id
WHERE p.status = 'published';

-- 2. 标签反查 (三表 JOIN)
SELECT p.title
FROM posts p
JOIN post_tags pt ON p.id = pt.post_id
JOIN tags t ON pt.tag_id = t.id  -- 注意: tag表主键是id
WHERE t.name = '数据库';
```

</details>

---

## 🟡 4. 中级挑战 (Level 2: Aggregation & Modeling)

**目标**: 处理聚合统计和日期处理。

1.  **标签云 (Tag Cloud)**: 统计每个标签下有多少篇文章。显示 `标签名` 和 `文章数`，按数量降序排列。
2.  **每月归档 (Monthly Archive)**: 很多博客都有“2023 年 1 月 (5 篇)”这样的侧边栏。
    - 请按 `年份-月份` 分组，统计已发布文章的数量。
    - 提示: 使用 `DATE_FORMAT(created_at, '%Y-%m')`。

<details>
<summary>点击查看参考答案</summary>

```sql
-- 1. 标签云
SELECT t.name, COUNT(pt.post_id) as count
FROM tags t
LEFT JOIN post_tags pt ON t.id = pt.tag_id
GROUP BY t.id, t.name
ORDER BY count DESC;

-- 2. 每月归档
SELECT DATE_FORMAT(created_at, '%Y-%m') as `month`, COUNT(*) as total
FROM posts
WHERE status = 'published'
GROUP BY `month`
ORDER BY `month` DESC;
```

</details>

---

## 🔴 5. 高级挑战 (Level 3: Recursion & CTE)

这是非 JSON 数据处理中的难点：处理**树形结构**。

### 任务 1: 生成分类面包屑 (Breadcrumbs)

假设我们在浏览 分类 ID 为 4 的 "MySQL" 版块。
我们需要显示路径：`技术 > 后端 > MySQL`。
**目标**: 使用 `WITH RECURSIVE` 从子分类 (ID=4) 向上递归查找到顶级分类。

### 任务 2: 查找热门分类 (包含子分类)

如果是层级结构，仅仅统计 `category_id = 1` ("技术") 的文章是不够的，因为它下面还有 "后端" 和 "MySQL"。
**目标**: 统计 "技术" 分类 (ID=1) 及其 **所有子分类** 下的文章总数。
(注：这通常需要先递归找出 ID=1 的所有下级 ID，再 `IN (...)` 查询)

<details>
<summary>点击查看参考答案</summary>

```sql
-- 任务 1: 向上递归找父级
WITH RECURSIVE category_path AS (
    SELECT id, name, parent_id, 0 as level
    FROM categories
    WHERE id = 4 -- 起点

    UNION ALL

    SELECT c.id, c.name, c.parent_id, cp.level - 1
    FROM categories c
    JOIN category_path cp ON c.id = cp.parent_id
)
SELECT name FROM category_path ORDER BY level;
-- 结果集顺序拼接即为路径

-- 任务 2: 向下递归找子级并统计
WITH RECURSIVE sub_categories AS (
    SELECT id FROM categories WHERE id = 1 -- 起点: 技术
    UNION ALL
    SELECT c.id FROM categories c
    JOIN sub_categories sc ON c.parent_id = sc.id
)
SELECT COUNT(*)
FROM posts
WHERE category_id IN (SELECT id FROM sub_categories);
```

</details>

---

## 🚀 6. 专家级思考 (Optimization)

在千万级数据量下，深分页 (Deep Pagination) 是个大问题。
比如 `LIMIT 1000000, 10` 会非常慢。

**思考题**:
如果不使用 `LIMIT offset, limit`，而是利用主键 ID 是连续自增（或有索引的时间戳）的特性，该如何优化下一页的查询？

**答案**:
使用 **Keyset Pagination (游标分页)**。
记录这一页最后一条数据的 ID (例如 9999)。
下一页查询时：
`SELECT * FROM posts WHERE id < 9999 ORDER BY id DESC LIMIT 10;`
这利用了索引扫描（Range Scan），比扫描并丢弃前 1000000 行要快几个数量级。
