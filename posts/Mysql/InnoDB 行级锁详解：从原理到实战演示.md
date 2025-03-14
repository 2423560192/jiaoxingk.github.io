# InnoDB 行级锁详解：从原理到实战演示

## 1. 行级锁是个啥？

### 1.1 用生活化的比喻理解

想象你在图书馆借一本菜谱（表），里面有很多菜（行）。行级锁就像只锁住你想看的几页（某几行数据），其他人还能翻别的页。这样比锁整本书（表级锁）省事，冲突少，大家能一起用，效率高。

在 InnoDB 里，行级锁靠索引实现，锁的是索引项，不是直接锁数据。如果没索引，它可能“翻车”，锁住整张表（后面有例子）。

### 1.2 三种行级锁

InnoDB 的行级锁有三种“玩法”：

- **行锁（Record Lock）**：锁住单行，比如锁住“炒鸡蛋”那页，别人不能改。
- **间隙锁（Gap Lock）**：锁住两页之间的“空隙”，防止别人插新页（新数据），防幻读。
- **临键锁（Next-Key Lock）**：行锁 + 间隙锁一起上，既锁住页，又锁住前面的空隙，默认在**可重复读（RR）**隔离级别用。

### 1.3 怎么查锁？

想知道锁的情况，用这句 SQL：

```sql
SELECT object_schema, object_name, index_name, lock_type, lock_mode, lock_data 
FROM performance_schema.data_locks;
```

或者看看 InnoDB 状态：

```sql
SHOW ENGINE INNODB STATUS;
```

## 2. 行锁：基础中的基础

### 2.1 两种锁类型

行锁有两种“门票”：

- **共享锁（S Lock）**：让你看数据，但不让别人改。就像你看菜谱，别人不能撕页。
- **排他锁（X Lock）**：让你改数据，别人啥也干不了。就像你改菜谱，别人连看都不行。

#### 兼容性表

| 锁类型 | S 锁 | X 锁 |
| :----- | :--- | :--- |
| S 锁   | OK   | 挡住 |
| X 锁   | 挡住 | 挡住 |

#### SQL 加锁一览

| SQL 语句                      | 锁类型 | 说明             |
| :---------------------------- | :----- | :--------------- |
| INSERT ...                    | X 锁   | 自动加           |
| UPDATE ...                    | X 锁   | 自动加           |
| DELETE ...                    | X 锁   | 自动加           |
| SELECT ...                    | 无锁   | 普通查询靠快照读 |
| SELECT ... LOCK IN SHARE MODE | S 锁   | 手动加共享锁     |
| SELECT ... FOR UPDATE         | X 锁   | 手动加排他锁     |

### 2.2 行锁的“脾气”

- **默认模式**：InnoDB 用**可重复读（RR）**隔离级别，靠 Next-Key Lock 防幻读。
- **聪明优化**：查唯一索引（像书的页码），如果找到记录，就只用行锁。
- **小心陷阱**：没索引时，行锁会“膨胀”成表锁，把整本书锁住！

## 3. 实战演示：玩转行锁

### 3.1 先搭个舞台

咱们建个学生表 `stu`，装点数据：

```sql
CREATE TABLE `stu` (
  `id` INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
  `name` VARCHAR(255) DEFAULT NULL,
  `age` INT NOT NULL
) ENGINE = InnoDB CHARACTER SET = utf8mb4;

INSERT INTO `stu` VALUES 
(1, 'tom', 1), 
(3, 'cat', 3), 
(8, 'rose', 8), 
(11, 'jetty', 11), 
(19, 'lily', 19), 
(25, 'luci', 25);
```

### 3.2 动手试试

#### A. 普通查询：不锁门

```sql
SELECT * FROM stu WHERE id = 1;
```

查锁：

```sql
SELECT * FROM performance_schema.data_locks;
```

结果：啥也没锁！普通 `SELECT` 用 MVCC 快照读，不加锁。

#### B. 共享锁：大家都能看

客户端一：

```sql
BEGIN;
SELECT * FROM stu WHERE id = 1 LOCK IN SHARE MODE;
```

给 `id=1` 加 S 锁。

客户端二：

```sql
BEGIN;
SELECT * FROM stu WHERE id = 3 FOR UPDATE;  -- OK，没冲突
SELECT * FROM stu WHERE id = 1 FOR UPDATE;  -- 卡住，S 锁挡 X 锁
```

小结：S 锁让大家都能看，但不让改。提交客户端一后，客户端二才动。

#### C. 排他锁：我改你别动

客户端一：

```sql
BEGIN;
UPDATE stu SET age = 2 WHERE id = 1;
```

给 `id=1` 加 X 锁。

客户端二：

```sql
BEGIN;
UPDATE stu SET age = 3 WHERE id = 1;  -- 卡住，X 锁不让进

```

小结：X 锁独占，提交客户端一后，客户端二才继续。

#### D. 没索引的坑：锁全表

客户端一：

```sql
BEGIN;
UPDATE stu SET age = 20 WHERE name = 'lily';  -- name 没索引

```

客户端二：

```sql
UPDATE stu SET age = 4 WHERE id = 3;  -- 卡住

```

为啥？：没索引，InnoDB 扫全表，行锁变表锁。

加个索引试试：

```sql
CREATE INDEX idx_name ON stu(name);
BEGIN;
UPDATE stu SET age = 20 WHERE name = 'lily';

```

客户端二再跑 `UPDATE id=3`，成功！

小结：索引救命，锁住一行不锁全表。

## 4. 间隙锁和临键锁：防“偷插”高手

### 4.1 啥是这俩？

- **间隙锁（Gap Lock）**：锁住两页之间的“空隙”，不让别人插新页。比如锁住 `(3, 8)`，不能加 `id=5`。
- **临键锁（Next-Key Lock）**：行锁 + 间隙锁，锁住页和前面的空隙，像 `(3, 8]`，默认防幻读。

### 4.2 怎么加锁？

- 唯一索引等值查：查不到记录时，变间隙锁。
- 非唯一索引等值查：扫到不满足条件的值，退化成间隙锁。
- 范围查：分段锁住，防插入。

### 4.3 实战演练

#### A. 查不到记录：间隙锁出马

```sql
BEGIN;
SELECT * FROM stu WHERE id = 10 FOR UPDATE;  -- id=10 没数据

```

锁住 `(8, 11)` 的间隙。

测试：

```sql
INSERT INTO stu (id, name, age) VALUES (10, 'test', 10);  -- 卡住

```

#### B. 非唯一索引：从临键到间隙

加个索引：

```sql
CREATE INDEX idx_age ON stu(age);

```

客户端一：

```sql
BEGIN;
SELECT * FROM stu WHERE age = 1 FOR UPDATE;

```

加 `(1, 3)` 间隙锁（扫到 `age=3` 停下）。

测试：

```sql
INSERT INTO stu (name, age) VALUES ('new', 2);  -- 卡住

```

#### C. 范围查询：分段锁

```sql
BEGIN;
SELECT * FROM stu WHERE id >= 19 LOCK IN SHARE MODE;

```

- `[19]`：行锁  
- `(19, 25]`：临键锁  
- `(25, +∞]`：临键锁  

测试：

```sql
INSERT INTO stu (id, name, age) VALUES (20, 'test', 20);  -- 卡住

```

## 5. 总结：锁的“套路”

通过上面的讲解和实战演示，相信你对 InnoDB 的行级锁有了更清晰的理解。总结一下，行级锁的核心要点如下：

- **行锁（Record Lock）**：S 锁看、X 锁改，靠索引跑得快。
- **间隙锁（Gap Lock）**：防插入，锁空隙的小卫兵。
- **临键锁（Next-Key Lock）**：行锁 + 间隙锁，RR 隔离级别的大招，防止幻读。