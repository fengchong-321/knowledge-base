# SQL 命令测试工程师精通指南

> 标签: #sql #database #testing #面试
> 创建时间: 2026-02-26
> 来源: [SQL Tutorial](https://www.w3schools.com/sql/) | [DataCamp SQL](https://www.datacamp.com/)

## 概述

SQL 是测试工程师验证数据正确性的核心技能，涉及数据查询、数据比对、测试数据准备等工作。本文整理测试面试高频 SQL 知识点，按重要程度分类。

---

## 一、知识体系总览

### 掌握程度分类

| 级别 | 说明 | 面试权重 |
|------|------|----------|
| 🔴 必须掌握 | 面试必问，日常必用 | 40% |
| 🟠 重要 | 常见考点，需要熟练 | 30% |
| 🟡 常用 | 工作中频繁使用 | 20% |
| 🟢 了解 | 高级场景，知道即可 | 10% |

---

## 二、核心知识点

### 🔴 必须掌握

#### 1. 基础查询

```sql
-- ========== SELECT 基础 ==========
SELECT * FROM users;                          -- 查询所有列
SELECT name, email FROM users;                -- 指定列
SELECT DISTINCT status FROM orders;           -- 去重

-- ========== WHERE 条件 ==========
SELECT * FROM users WHERE age > 18;
SELECT * FROM users WHERE status = 'active';
SELECT * FROM users WHERE age >= 18 AND city = 'Beijing';
SELECT * FROM users WHERE age > 18 OR vip = 1;

-- ========== 比较运算符 ==========
-- =, !=, <>, <, >, <=, >=
-- IS NULL, IS NOT NULL
-- BETWEEN AND
-- IN (value1, value2, ...)

SELECT * FROM users WHERE age BETWEEN 18 AND 30;
SELECT * FROM users WHERE city IN ('Beijing', 'Shanghai', 'Guangzhou');
SELECT * FROM users WHERE phone IS NULL;

-- ========== LIKE 模糊匹配 ==========
SELECT * FROM users WHERE name LIKE '张%';      -- 以张开头
SELECT * FROM users WHERE name LIKE '%伟';      -- 以伟结尾
SELECT * FROM users WHERE name LIKE '%小%';     -- 包含小
SELECT * FROM users WHERE email LIKE '%@gmail.com';
-- % 任意多个字符，_ 单个字符

-- ========== ORDER BY 排序 ==========
SELECT * FROM users ORDER BY age;              -- 默认升序
SELECT * FROM users ORDER BY age DESC;         -- 降序
SELECT * FROM users ORDER BY city, age DESC;   -- 多字段排序

-- ========== LIMIT 分页 ==========
SELECT * FROM users LIMIT 10;                  -- 前 10 条
SELECT * FROM users LIMIT 10 OFFSET 20;        -- 从第 21 条开始，取 10 条
SELECT * FROM users LIMIT 20, 10;              -- MySQL 语法（跳过 20 条，取 10 条）
```

#### 2. 聚合函数与分组

```sql
-- ========== 聚合函数 ==========
SELECT COUNT(*) FROM users;                    -- 总数
SELECT COUNT(email) FROM users;                -- 非空数量
SELECT COUNT(DISTINCT city) FROM users;        -- 去重计数

SELECT SUM(amount) FROM orders;                -- 求和
SELECT AVG(age) FROM users;                    -- 平均值
SELECT MAX(price) FROM products;               -- 最大值
SELECT MIN(price) FROM products;               -- 最小值

-- ========== GROUP BY 分组 ==========
SELECT city, COUNT(*) FROM users GROUP BY city;
SELECT status, SUM(amount) FROM orders GROUP BY status;
SELECT department, AVG(salary) FROM employees GROUP BY department;

-- ========== HAVING 分组过滤 ==========
-- HAVING 用于过滤分组后的结果（WHERE 过滤原数据）
SELECT city, COUNT(*) as cnt
FROM users
GROUP BY city
HAVING COUNT(*) > 100;

SELECT department, AVG(salary) as avg_sal
FROM employees
GROUP BY department
HAVING AVG(salary) > 10000;

-- ========== 完整语法顺序 ==========
SELECT column
FROM table
WHERE condition
GROUP BY column
HAVING condition
ORDER BY column
LIMIT n;
```

#### 3. 多表连接 (JOIN)

```sql
-- ========== INNER JOIN 内连接 ==========
-- 只返回两表都有匹配的记录
SELECT u.name, o.order_no
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- ========== LEFT JOIN 左连接 ==========
-- 返回左表所有记录，右表无匹配则为 NULL
SELECT u.name, o.order_no
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;

-- ========== RIGHT JOIN 右连接 ==========
-- 返回右表所有记录，左表无匹配则为 NULL
SELECT u.name, o.order_no
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id;

-- ========== FULL OUTER JOIN 全连接 ==========
-- 返回两表所有记录（MySQL 不支持，用 UNION 替代）
SELECT u.name, o.order_no
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
UNION
SELECT u.name, o.order_no
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id;

-- ========== 多表连接 ==========
SELECT u.name, o.order_no, p.product_name
FROM users u
INNER JOIN orders o ON u.id = o.user_id
INNER JOIN products p ON o.product_id = p.id;

-- ========== 自连接 ==========
-- 查询员工和其经理
SELECT e.name as employee, m.name as manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

#### 4. 子查询

```sql
-- ========== WHERE 子句中的子查询 ==========
-- 查询工资高于平均值的员工
SELECT * FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- 查询有订单的用户
SELECT * FROM users
WHERE id IN (SELECT DISTINCT user_id FROM orders);

-- 查询没有订单的用户
SELECT * FROM users
WHERE id NOT IN (SELECT DISTINCT user_id FROM orders WHERE user_id IS NOT NULL);

-- ========== FROM 子句中的子查询（临时表）==========
SELECT dept, avg_sal
FROM (
    SELECT department as dept, AVG(salary) as avg_sal
    FROM employees
    GROUP BY department
) t
WHERE avg_sal > 10000;

-- ========== EXISTS 子查询 ==========
SELECT * FROM users u
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);

-- ========== 关联子查询 ==========
-- 查询每个部门工资最高的员工
SELECT * FROM employees e1
WHERE salary = (
    SELECT MAX(salary)
    FROM employees e2
    WHERE e2.department = e1.department
);
```

#### 5. 数据操作 (DML)

```sql
-- ========== INSERT 插入 ==========
INSERT INTO users (name, email, age) VALUES ('张三', 'zhangsan@email.com', 25);
INSERT INTO users (name, email) VALUES ('李四', 'lisi@email.com'), ('王五', 'wangwu@email.com');

-- ========== UPDATE 更新 ==========
UPDATE users SET status = 'inactive' WHERE id = 1;
UPDATE users SET age = age + 1, updated_at = NOW() WHERE city = 'Beijing';

-- ========== DELETE 删除 ==========
DELETE FROM users WHERE id = 1;
DELETE FROM users WHERE status = 'inactive';

-- ========== 面试常问：三者的区别 ==========
-- DELETE：删除数据，可回滚，记录日志
-- TRUNCATE：删除全部数据，不可回滚，不记录日志，更快
-- DROP：删除表结构和数据，不可恢复
```

---

### 🟠 重要

#### 6. 数据定义 (DDL)

```sql
-- ========== CREATE 创建表 ==========
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    age INT DEFAULT 0,
    status VARCHAR(20) DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_name (name),
    INDEX idx_email (email)
);

-- ========== ALTER 修改表 ==========
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
ALTER TABLE users DROP COLUMN phone;
ALTER TABLE users MODIFY COLUMN name VARCHAR(100);
ALTER TABLE users ADD INDEX idx_city (city);

-- ========== DROP 删除表 ==========
DROP TABLE IF EXISTS users;
TRUNCATE TABLE users;  -- 清空数据保留结构

-- ========== 约束 ==========
-- PRIMARY KEY：主键
-- FOREIGN KEY：外键
-- UNIQUE：唯一
-- NOT NULL：非空
-- DEFAULT：默认值
-- CHECK：检查约束
```

#### 7. 字符串与日期函数

```sql
-- ========== 字符串函数 ==========
SELECT CONCAT(first_name, ' ', last_name) FROM users;    -- 拼接
SELECT SUBSTRING('Hello World', 1, 5);                   -- 截取（结果：Hello）
SELECT LENGTH('Hello');                                   -- 长度
SELECT UPPER('hello'), LOWER('HELLO');                   -- 大小写
SELECT TRIM('  hello  ');                                 -- 去空格
SELECT REPLACE('Hello World', 'World', 'SQL');           -- 替换
SELECT LEFT('Hello', 2), RIGHT('Hello', 2);              -- 左右截取

-- ========== 日期函数 ==========
SELECT NOW(), CURRENT_TIMESTAMP;                          -- 当前时间
SELECT CURDATE(), CURRENT_DATE;                           -- 当前日期
SELECT CURTIME(), CURRENT_TIME;                           -- 当前时间

SELECT YEAR('2024-01-15'), MONTH('2024-01-15'), DAY('2024-01-15');
SELECT DATE_FORMAT(NOW(), '%Y-%m-%d %H:%i:%s');
SELECT DATE_ADD(NOW(), INTERVAL 7 DAY);
SELECT DATE_SUB(NOW(), INTERVAL 1 MONTH);
SELECT DATEDIFF('2024-12-31', '2024-01-01');              -- 日期差

-- ========== 常用日期查询 ==========
SELECT * FROM orders WHERE DATE(created_at) = '2024-01-15';
SELECT * FROM orders WHERE created_at >= '2024-01-01' AND created_at < '2024-02-01';
SELECT * FROM orders WHERE YEAR(created_at) = 2024 AND MONTH(created_at) = 1;
```

#### 8. 条件表达式

```sql
-- ========== CASE WHEN ==========
SELECT
    name,
    CASE
        WHEN score >= 90 THEN 'A'
        WHEN score >= 80 THEN 'B'
        WHEN score >= 60 THEN 'C'
        ELSE 'D'
    END as grade
FROM students;

-- ========== IF 函数（MySQL）==========
SELECT name, IF(score >= 60, 'Pass', 'Fail') as result FROM students;

-- ========== COALESCE 返回第一个非空值 ==========
SELECT COALESCE(phone, email, 'N/A') as contact FROM users;

-- ========== NULLIF 如果相等返回 NULL ==========
SELECT NULLIF(a, b) FROM table;
```

#### 9. UNION 与 UNION ALL

```sql
-- ========== UNION 去重合并 ==========
SELECT name FROM users
UNION
SELECT name FROM customers;

-- ========== UNION ALL 保留重复 ==========
SELECT name FROM users
UNION ALL
SELECT name FROM customers;

-- 面试题：区别？
-- UNION 会去重，UNION ALL 不会，UNION ALL 更快
```

---

### 🟡 常用

#### 10. 窗口函数

```sql
-- ========== ROW_NUMBER 行号 ==========
SELECT
    name,
    department,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) as rank
FROM employees;

-- ========== RANK 排名（有并列）==========
SELECT
    name,
    salary,
    RANK() OVER (ORDER BY salary DESC) as rank
FROM employees;

-- ========== DENSE_RANK 排名（无间隙）==========
SELECT
    name,
    salary,
    DENSE_RANK() OVER (ORDER BY salary DESC) as rank
FROM employees;

-- ========== 分组内排名 ==========
SELECT
    name,
    department,
    salary,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) as dept_rank
FROM employees;

-- ========== 聚合窗口函数 ==========
SELECT
    date,
    amount,
    SUM(amount) OVER (ORDER BY date) as running_total,
    AVG(amount) OVER (ORDER BY date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) as moving_avg
FROM sales;
```

#### 11. 索引与性能

```sql
-- ========== 创建索引 ==========
CREATE INDEX idx_name ON users(name);
CREATE INDEX idx_city_age ON users(city, age);  -- 复合索引
CREATE UNIQUE INDEX idx_email ON users(email);

-- ========== 查看索引 ==========
SHOW INDEX FROM users;

-- ========== 删除索引 ==========
DROP INDEX idx_name ON users;

-- ========== 执行计划分析 ==========
EXPLAIN SELECT * FROM users WHERE name = '张三';

-- ========== 索引使用原则 ==========
-- 1. 频繁作为 WHERE 条件的列
-- 2. 经常用于 JOIN 的列
-- 3. 经常用于 ORDER BY 的列
-- 4. 避免在低基数列建索引（如性别）
-- 5. 复合索引遵循最左前缀原则
```

#### 12. 事务

```sql
-- ========== 事务控制 ==========
BEGIN;                    -- 开始事务
START TRANSACTION;

COMMIT;                   -- 提交
ROLLBACK;                 -- 回滚

-- ========== 示例 ==========
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;

-- ========== ACID 属性 ==========
-- A (Atomicity) 原子性：要么全做，要么全不做
-- C (Consistency) 一致性：事务前后数据一致
-- I (Isolation) 隔离性：事务之间互不干扰
-- D (Durability) 持久性：事务提交后永久保存
```

---

### 🟢 了解

#### 13. 高级特性

```sql
-- ========== 视图 ==========
CREATE VIEW user_orders AS
SELECT u.name, o.order_no, o.amount
FROM users u
JOIN orders o ON u.id = o.user_id;

SELECT * FROM user_orders;
DROP VIEW user_orders;

-- ========== 存储过程 ==========
DELIMITER //
CREATE PROCEDURE GetUsersByCity(IN city_name VARCHAR(50))
BEGIN
    SELECT * FROM users WHERE city = city_name;
END //
DELIMITER ;

CALL GetUsersByCity('Beijing');

-- ========== 触发器 ==========
CREATE TRIGGER before_user_insert
BEFORE INSERT ON users
FOR EACH ROW
SET NEW.created_at = NOW();

-- ========== 临时表 ==========
CREATE TEMPORARY TABLE temp_users AS
SELECT * FROM users WHERE status = 'active';
```

#### 14. 数据库设计

```sql
-- ========== 三范式 ==========
-- 1NF：字段不可分割
-- 2NF：非主键字段完全依赖主键
-- 3NF：非主键字段不传递依赖主键

-- ========== 反范式化 ==========
-- 适当冗余，减少 JOIN，提高查询性能

-- ========== 主键设计原则 ==========
-- 1. 唯一性
-- 2. 非空
-- 3. 不变
-- 4. 单列优先（自增 ID 或 UUID）
```

---

## 三、面试高频问题

### 基础篇

| 问题 | 答案 |
|------|------|
| WHERE 和 HAVING 的区别？ | WHERE 过滤原始数据，HAVING 过滤分组后数据 |
| COUNT(*) 和 COUNT(column) 区别？ | COUNT(*) 包含 NULL，COUNT(column) 不包含 |
| INNER JOIN 和 LEFT JOIN 区别？ | INNER 只返回匹配，LEFT 返回左表全部 |
| LIKE 中 % 和 _ 的区别？ | % 任意多个字符，_ 单个字符 |
| UNION 和 UNION ALL 区别？ | UNION 去重，UNION ALL 不去重 |
| 主键和唯一键区别？ | 主键非空且唯一，唯一键可为空 |

### 进阶篇

| 问题 | 答案 |
|------|------|
| DELETE、TRUNCATE、DROP 区别？ | DELETE 可回滚，TRUNCATE 不可回滚更快，DROP 删表 |
| 什么是 SQL 注入？如何防止？ | 恶意 SQL 拼接，用参数化查询防止 |
| 索引的优缺点？ | 加快查询，但占用空间、降低写性能 |
| 什么情况下索引会失效？ | 使用函数、类型转换、LIKE '%xx'、OR 条件 |
| 如何优化 SQL 查询？ | 索引、避免 SELECT *、优化 WHERE、分页 |

### 高级篇

| 问题 | 答案 |
|------|------|
| 如何查找第 N 高的薪水？ | 子查询或 LIMIT OFFSET |
| 如何去重？ | DISTINCT、GROUP BY、ROW_NUMBER() |
| 什么是 MVCC？ | 多版本并发控制，解决读写冲突 |
| 事务隔离级别有哪些？ | 读未提交、读已提交、可重复读、串行化 |

---

## 四、实战场景

### 场景1：测试数据验证

```sql
-- 验证数据完整性
SELECT COUNT(*) as total, COUNT(DISTINCT user_id) as unique_users
FROM orders;

-- 验证数据一致性
SELECT o.*, u.name
FROM orders o
LEFT JOIN users u ON o.user_id = u.id
WHERE u.id IS NULL;  -- 找出孤儿数据

-- 验证业务规则
SELECT * FROM orders
WHERE amount < 0 OR created_at > NOW();
```

### 场景2：查找重复数据

```sql
-- 查找重复记录
SELECT email, COUNT(*) as cnt
FROM users
GROUP BY email
HAVING COUNT(*) > 1;

-- 查找重复数据的详细信息
SELECT * FROM users
WHERE email IN (
    SELECT email FROM users GROUP BY email HAVING COUNT(*) > 1
);
```

### 场景3：分页查询

```sql
-- MySQL 分页
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 0;   -- 第 1 页
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 10;  -- 第 2 页

-- 通用分页公式
-- OFFSET = (page - 1) * pageSize
-- LIMIT = pageSize
```

### 场景4：统计报表

```sql
-- 每日销售统计
SELECT
    DATE(created_at) as date,
    COUNT(*) as order_count,
    SUM(amount) as total_amount,
    AVG(amount) as avg_amount
FROM orders
GROUP BY DATE(created_at)
ORDER BY date DESC;

-- 用户活跃度分析
SELECT
    CASE
        WHEN order_count = 0 THEN 'inactive'
        WHEN order_count < 5 THEN 'low'
        WHEN order_count < 20 THEN 'medium'
        ELSE 'high'
    END as activity_level,
    COUNT(*) as user_count
FROM (
    SELECT user_id, COUNT(*) as order_count
    FROM orders
    GROUP BY user_id
) t
GROUP BY activity_level;
```

---

## 五、SQL 速查表

### 查询

| 语句 | 说明 |
|------|------|
| `SELECT * FROM t` | 查询所有 |
| `WHERE c = v` | 条件过滤 |
| `ORDER BY c DESC` | 降序排序 |
| `LIMIT n OFFSET m` | 分页 |
| `DISTINCT` | 去重 |

### 聚合

| 函数 | 说明 |
|------|------|
| `COUNT(*)` | 计数 |
| `SUM(c)` | 求和 |
| `AVG(c)` | 平均值 |
| `MAX(c)` | 最大值 |
| `MIN(c)` | 最小值 |
| `GROUP BY` | 分组 |
| `HAVING` | 分组过滤 |

### 连接

| 类型 | 说明 |
|------|------|
| `INNER JOIN` | 内连接 |
| `LEFT JOIN` | 左连接 |
| `RIGHT JOIN` | 右连接 |
| `FULL JOIN` | 全连接 |

### 操作

| 语句 | 说明 |
|------|------|
| `INSERT INTO` | 插入 |
| `UPDATE SET` | 更新 |
| `DELETE FROM` | 删除 |
| `TRUNCATE` | 清空 |

---

## 相关知识点

- [[Linux 命令测试工程师精通指南]]
- [[Python Pandas 精通指南]]
- [[Python Requests 精通指南]]

---
*采集自 Claude Code 对话*

**Sources:**
- [DataCamp SQL Interview Questions](https://www.datacamp.com/blog/top-sql-interview-questions-and-answers-for-beginners-and-intermediate-practitioners)
- [W3Schools SQL Tutorial](https://www.w3schools.com/sql/)
- [菜鸟教程 SQL](https://www.runoob.com/sql/)
