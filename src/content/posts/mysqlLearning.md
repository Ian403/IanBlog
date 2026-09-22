---
title: MySQL 学习记录
author: Ian
pubDatetime: 2026-09-23T00:00:00+08:00
featured: true
draft: false
tags:
  - MySQL
  - Technique
ogImage: ../../assets/images/MyFirstBlog.jpg # src/assets/images/example.png
# ogImage: "https://example.org/remote-image.png" # remote URL
description: MySQL 知识点及速查表
---

# MySQL 学习记录

## 前言

2026/9/22 跟随 Harvard CS50 学习了 SQL 的基础知识以及用法，作此贴来巩固，以及方便后续需要时速查。

SQL 是用于操作关系型数据库的语言， 是一种声明式的语言。

数据库（database）用于有组织地存储、管理和查询数据。

## 如何在 Linux 上安装

依次在命令行中运行以下指令

安装指令:

```bash
sudo apt update
sudo apt install mysql-server
```

查看 MySQL 服务状态:

```bash
sudo systemctl status mysql
```

设置开机自动启动（可选）:

```bash
sudo systemctl enable mysql
```

进入 MySQL:

```bash
sudo mysql
```

MySQL 与 SQLite 不同， 不能通过 `sqlite3 databaseName.db` 在当前路径创建数据库，它的数据库是统一储存在电脑中的 `var/lib/mysql` (通常)中。如果要创建新的数据库应当进入 MySQL 后使用 `CREATE` 语句来实现。

## 常用关键字速查

注意字符串中的 `'` 应该替换成 `''`,类似 **python** 中的转义语句。

### 数据库与表操作

| 关键字／语法 | 具体用法 | 示例 |
| --- | --- | --- |
| `CREATE DATABASE` | 创建数据库 | `CREATE DATABASE blog;` |
| `SHOW DATABASES` | 查看数据库列表 | `SHOW DATABASES;` |
| `USE` | 切换当前数据库 | `USE blog;` |
| `DROP DATABASE` | 删除数据库及其中的数据 | `DROP DATABASE blog;` |
| `CREATE TABLE` | 创建表并定义字段 | `CREATE TABLE users (id BIGINT PRIMARY KEY AUTO_INCREMENT, name VARCHAR(50) NOT NULL, age INT);` |
| `SHOW TABLES` | 查看当前数据库中的表 | `SHOW TABLES;` |
| `DESCRIBE` / `DESC` | 查看表结构 | `DESC users;` |
| `SHOW CREATE TABLE` | 查看建表语句 | `SHOW CREATE TABLE users;` |
| `ALTER TABLE ... ADD COLUMN` | 添加字段 | `ALTER TABLE users ADD COLUMN email VARCHAR(255);` |
| `ALTER TABLE ... MODIFY COLUMN` | 修改字段类型及属性 | `ALTER TABLE users MODIFY COLUMN name VARCHAR(100) NOT NULL;` |
| `ALTER TABLE ... DROP COLUMN` | 删除字段及其数据 | `ALTER TABLE users DROP COLUMN email;` |
| `TRUNCATE TABLE` | 清空表，保留表结构，通常重置自增计数器；不支持 `WHERE`，不能通过事务回滚 | `TRUNCATE TABLE users;` |
| `DROP TABLE` | 删除表及其中的数据 | `DROP TABLE users;` |

### 字段约束与属性

| 关键字／语法 | 具体用法 | 示例 |
| --- | --- | --- |
| `PRIMARY KEY` | 定义主键，值唯一且不能为 `NULL` | `CREATE TABLE users (id BIGINT PRIMARY KEY);` |
| `AUTO_INCREMENT` | 为整数列自动生成递增值 | `CREATE TABLE users (id BIGINT PRIMARY KEY AUTO_INCREMENT);` |
| `NOT NULL` | 禁止字段值为 `NULL` | `CREATE TABLE users (name VARCHAR(50) NOT NULL);` |
| `UNIQUE` | 限制字段值唯一；可空的唯一列允许多个 `NULL` | `CREATE TABLE users (email VARCHAR(255) UNIQUE);` |
| `DEFAULT` | 插入时未指定字段值则使用默认值 | `CREATE TABLE users (status VARCHAR(20) DEFAULT 'active');` |
| `FOREIGN KEY ... REFERENCES` | 约束字段引用另一张表的键；示例要求 `users.id` 为类型匹配的主键 | `CREATE TABLE orders (id BIGINT PRIMARY KEY, user_id BIGINT, FOREIGN KEY (user_id) REFERENCES users(id));` |
| `CHECK` | 检查字段值是否满足条件，MySQL 8.0.16 起实际执行 | `CREATE TABLE users (age INT CHECK (age >= 0));` |

### 基础增删改查

| 关键字／语法 | 具体用法 | 示例 |
| --- | --- | --- |
| `SELECT ... FROM` | 从表中查询指定字段，`*` 表示所有字段 | `SELECT id, name, age FROM users;` |
| `INSERT INTO ... VALUES` | 插入一条记录 | `INSERT INTO users (name, age) VALUES ('小明', 20);` |
| `INSERT INTO ... VALUES` | 使用逗号分隔多组值，一次插入多条记录 | `INSERT INTO users (name, age) VALUES ('小红', 22), ('小李', 25);` |
| `UPDATE ... SET` | 更新符合条件的记录；省略 `WHERE` 会更新所有记录 | `UPDATE users SET age = 21 WHERE id = 1;` |
| `DELETE FROM` | 删除符合条件的记录，保留表结构；省略 `WHERE` 会删除所有记录 | `DELETE FROM users WHERE id = 1;` |
| `AS` | 为字段、表达式或表设置别名 | `SELECT u.name AS user_name FROM users AS u;` |

### 条件筛选与运算符

| 关键字／运算符 | 具体用法 | 示例 |
| --- | --- | --- |
| `WHERE` | 筛选满足条件的记录 | `SELECT * FROM users WHERE age >= 18;` |
| `AND` | 要求多个条件同时成立 | `SELECT * FROM users WHERE age >= 18 AND status = 'active';` |
| `OR` | 要求至少一个条件成立；与 `AND` 混用时可用括号明确逻辑 | `SELECT * FROM users WHERE status = 'active' AND (age < 18 OR age >= 60);` |
| `NOT` | 对条件取反 | `SELECT * FROM users WHERE NOT (status = 'active');` |
| `=` | 判断是否相等，不能用来判断 `NULL` | `SELECT * FROM users WHERE age = 18;` |
| `<>` / `!=` | 判断是否不相等 | `SELECT * FROM users WHERE age <> 18;` |
| `>` / `<` | 判断是否大于或小于 | `SELECT * FROM users WHERE age > 18 AND age < 30;` |
| `>=` / `<=` | 判断是否大于等于或小于等于 | `SELECT * FROM users WHERE age >= 18 AND age <= 30;` |
| `IN` | 匹配集合中的任意值 | `SELECT * FROM users WHERE age IN (18, 20, 25);` |
| `NOT IN` | 排除集合中的值；集合包含 `NULL` 时可能导致预期记录无法选中 | `SELECT * FROM users WHERE age NOT IN (18, 20, 25);` |
| `BETWEEN ... AND ...` | 匹配范围，包含两端 | `SELECT * FROM users WHERE age BETWEEN 18 AND 30;` |
| `LIKE` | 模式匹配，`%` 匹配零个或多个字符，`_` 匹配恰好一个字符 | `SELECT * FROM users WHERE name LIKE '小%';` |
| `IS NULL` | 判断字段值是否为 `NULL` | `SELECT * FROM users WHERE email IS NULL;` |
| `IS NOT NULL` | 判断字段值是否不为 `NULL` | `SELECT * FROM users WHERE email IS NOT NULL;` |

注意不能使用 `= NULL` 来替代 `IS NULL`；`NULL` 不用加引号，它代表的是这个地方是空缺的。

### 去重、排序与分页

| 关键字／语法 | 具体用法 | 示例 |
| --- | --- | --- |
| `DISTINCT` | 去除重复结果；多个字段时按字段组合去重 | `SELECT DISTINCT status, age FROM users;` |
| `ORDER BY` | 按指定字段排序，可依次指定多个排序字段 | `SELECT * FROM users ORDER BY age DESC, id ASC;` |
| `ASC` | 升序排列，为默认排序方向 | `SELECT * FROM users ORDER BY age ASC;` |
| `DESC` | 降序排列 | `SELECT * FROM users ORDER BY age DESC;` |
| `LIMIT` | 限制返回条数 | `SELECT * FROM users ORDER BY id LIMIT 10;` |
| `LIMIT ... OFFSET ...` | 按“条数、偏移量”分页；使用唯一字段排序可保证排序稳定 | `SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 20;` |
| `LIMIT 偏移量, 条数` | MySQL 分页简写，先跳过指定行数，再返回指定条数 | `SELECT * FROM users ORDER BY id LIMIT 20, 10;` |

### 分组与聚合函数

| 关键字／函数 | 具体用法 | 示例 |
| --- | --- | --- |
| `COUNT(*)` | 统计行数 | `SELECT COUNT(*) AS user_count FROM users;` |
| `COUNT(column)` | 统计指定字段非 `NULL` 的行数 | `SELECT COUNT(email) AS email_count FROM users;` |
| `SUM()` | 对非 `NULL` 值求和 | `SELECT SUM(amount) AS total_amount FROM orders;` |
| `AVG()` | 对非 `NULL` 值求平均值 | `SELECT AVG(age) AS average_age FROM users;` |
| `MAX()` | 获取非 `NULL` 值中的最大值 | `SELECT MAX(age) AS max_age FROM users;` |
| `MIN()` | 获取非 `NULL` 值中的最小值 | `SELECT MIN(age) AS min_age FROM users;` |
| `GROUP BY` | 按字段分组，常与聚合函数一起使用 | `SELECT status, COUNT(*) AS user_count FROM users GROUP BY status;` |
| `HAVING` | 分组后筛选结果，可使用聚合条件；`WHERE` 在分组前筛选记录 | `SELECT status, COUNT(*) AS user_count FROM users WHERE age >= 18 GROUP BY status HAVING COUNT(*) > 10;` |

### 多表关联

| 关键字／语法 | 具体用法 | 示例 |
| --- | --- | --- |
| `INNER JOIN` | 只返回两张表中满足关联条件的记录 | `SELECT u.name, d.name AS department_name FROM users AS u INNER JOIN departments AS d ON u.department_id = d.id;` |
| `LEFT JOIN` | 保留左表所有记录，右表没有匹配时其字段为 `NULL` | `SELECT u.name, o.id AS order_id FROM users AS u LEFT JOIN orders AS o ON u.id = o.user_id;` |
| `RIGHT JOIN` | 保留右表所有记录，左表没有匹配时其字段为 `NULL` | `SELECT u.name, d.name AS department_name FROM users AS u RIGHT JOIN departments AS d ON u.department_id = d.id;` |
| `CROSS JOIN` | 返回两张表所有行的组合，即笛卡尔积 | `SELECT u.name, d.name AS department_name FROM users AS u CROSS JOIN departments AS d;` |
| `ON` | 指定表之间的关联条件 | `SELECT u.name, o.id FROM users AS u JOIN orders AS o ON u.id = o.user_id;` |

### 子查询与结果合并

| 关键字／语法 | 具体用法 | 示例 |
| --- | --- | --- |
| `IN (子查询)` | 匹配子查询返回的值 | `SELECT id, name FROM users WHERE id IN (SELECT user_id FROM orders);` |
| `EXISTS` | 判断子查询是否至少返回一条记录 | `SELECT u.id, u.name FROM users AS u WHERE EXISTS (SELECT 1 FROM orders AS o WHERE o.user_id = u.id);` |
| `NOT EXISTS` | 判断子查询是否没有返回记录 | `SELECT u.id, u.name FROM users AS u WHERE NOT EXISTS (SELECT 1 FROM orders AS o WHERE o.user_id = u.id);` |
| `UNION` | 合并结果并去重；各查询列数必须相同，对应类型需兼容 | `SELECT name FROM users UNION SELECT name FROM archived_users;` |
| `UNION ALL` | 合并结果并保留重复行；各查询列数必须相同，对应类型需兼容 | `SELECT name FROM users UNION ALL SELECT name FROM archived_users;` |

### 条件表达式

| 关键字／语法 | 具体用法 | 示例 |
| --- | --- | --- |
| `CASE WHEN ... THEN ... ELSE ... END` | 按顺序判断条件，返回首个成立条件对应的值；都不成立时返回 `ELSE` 的值 | `SELECT name, CASE WHEN age IS NULL THEN '未知' WHEN age < 18 THEN '未成年' WHEN age < 60 THEN '成年人' ELSE '老年人' END AS age_group FROM users;` |

### 索引与执行计划

使用索引可以优化数据库的查询效率（具体原理为 **B-Tree**，将查询的时间复杂度从 O(n) 降低到 O(log n ) )。

| 关键字／语法 | 具体用法 | 示例 |
| --- | --- | --- |
| `CREATE INDEX` | 创建单列索引 | `CREATE INDEX idx_users_name ON users (name);` |
| `CREATE INDEX` | 创建由多个字段组成的联合索引 | `CREATE INDEX idx_users_status_age ON users (status, age);` |
| `SHOW INDEX` | 查看表的索引 | `SHOW INDEX FROM users;` |
| `DROP INDEX` | 删除指定索引 | `DROP INDEX idx_users_name ON users;` |
| `EXPLAIN` | 查看查询执行计划，分析索引使用情况和表的访问方式 | `EXPLAIN SELECT id, name FROM users WHERE name = '小明';` |

### 事务

当两个指令同时要求进行数据库操作时可能会导致数据丢失。使用该语句可以确保修改操作正常执行且没有数据丢失。

| 关键字／语法 | 具体用法 | 示例 |
| --- | --- | --- |
| `START TRANSACTION` | 开始事务，数据回滚要求使用 InnoDB 等支持事务的存储引擎 | `START TRANSACTION;` |
| `COMMIT` | 提交当前事务的修改 | `START TRANSACTION; UPDATE users SET age = 21 WHERE id = 1; COMMIT;` |
| `ROLLBACK` | 撤销当前事务中尚未提交的数据修改；不能撤销通常会隐式提交的 `CREATE TABLE`、`ALTER TABLE`、`DROP TABLE`、`TRUNCATE TABLE` 等操作 | `START TRANSACTION; DELETE FROM users WHERE id = 1; ROLLBACK;` |
| `SAVEPOINT` | 在事务中创建保存点 | `START TRANSACTION; SAVEPOINT before_update;` |
| `ROLLBACK TO SAVEPOINT` | 撤销保存点之后的修改，不结束事务 | `START TRANSACTION; SAVEPOINT before_update; UPDATE users SET age = 21 WHERE id = 1; ROLLBACK TO SAVEPOINT before_update; COMMIT;` |

### MySQL 8.0 进阶语法

| 关键字／函数 | 具体用法 | 示例 |
| --- | --- | --- |
| `WITH ... AS` | 定义仅在当前语句中有效的公用表表达式（CTE） | `WITH adult_users AS (SELECT id, name, age FROM users WHERE age >= 18) SELECT * FROM adult_users ORDER BY age DESC;` |
| `OVER` | 定义窗口计算范围及排序，保留每条明细记录 | `SELECT name, age, ROW_NUMBER() OVER (ORDER BY age DESC, id ASC) AS row_num FROM users;` |
| `PARTITION BY` | 将窗口计算按字段分区，每个分区独立计算 | `SELECT name, department_id, ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY age DESC, id ASC) AS row_num FROM users;` |
| `ROW_NUMBER()` | 按窗口排序为每条记录生成从 1 开始的连续编号 | `SELECT id, name, ROW_NUMBER() OVER (ORDER BY id) AS row_num FROM users;` |

## **Python 连接 MySQL**

注意不要直接使用 `f"SELECT id FROM tableName WHERE name = {name}"` 之类的语句，可能会遭到用户的恶意输入的攻击，尽量使用 `"SELECT id FROM tableName WHERE name = ?", name` 这样的语句能有效避免攻击。

## 补充

### 标量子查询

```SQL
SELECT (
  SELECT name FROM students WHERE id = 100
) AS name;
```

可以做到强制返回 **NULL**，即使没有查询到对应的内容。

## 最后的最后

![CS50x 2026 SQL 课结尾的漫画：Little Bobby Tables，用名字中的 SQL 注入语句删除学生数据表](../../assets/images/xkcd-327-exploits-of-a-mom.png)

图片来源：Randall Munroe 的 [xkcd #327 — Exploits of a Mom](https://xkcd.com/327/)，采用 [CC BY-NC 2.5](https://creativecommons.org/licenses/by-nc/2.5/) 许可。
