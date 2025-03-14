# 前言

嗨，我是JiaoXingK，今天带你聊聊 MySQL 数据库里的“锁”。
锁是什么？简单说，它就像图书馆的管理员，帮我们协调多个程序（进程或线程）同时用数据，避免乱套。

> 数据库里，除了抢CPU、内存、硬盘（I/O）这些资源，数据本身也是大家共享的“宝贝”。锁的任务是保证数据一致性和并发访问的有效性，不然数据乱了，或者大家排队太久（锁冲突），性能就崩了。MySQL
> 的锁按“管多大范围”分三种：全局锁、表级锁、行级锁。咱们一个个聊！

## 1. 锁是个啥？
想象你在图书馆借书，很多人同时想借同一本，管理员得定规矩：谁先借，谁等一等。这就是锁机制，在计算机里用来协调并发访问。在 MySQL 中，锁解决的是数据一致性（数据不乱）和并发性能（别卡太久）的问题。锁冲突多了，性能就差，所以锁很重要，也挺复杂。

MySQL 的锁按粒度（锁多大一块）分：
- **全局锁（Global Lock）**：锁住整个数据库实例。
- **表级锁（Table-Level Lock）**：锁住一张表。
- **行级锁（Row-Level Lock）**：锁住表里几行数据。

## 2. 全局锁：锁住整个图书馆
### 啥是全局锁？
全局锁就像锁住整个图书馆大门，用命令 `FLUSH TABLES WITH READ LOCK`（简称 FTWRL）加锁。锁上后，整个数据库变成只读状态，所有的写操作（DML，如 `INSERT`、`UPDATE`）、改结构操作（DDL，如 `ALTER TABLE`）、甚至事务提交（`COMMIT`）都会被阻塞，只能查（`SELECT`）。

### 干啥用？
主要用来做全库逻辑备份。比如用 `mysqldump` 把数据库所有表的数据导出来，得保证备份时数据一致，不能有人偷偷改。这时候全局锁就派上用场了，锁住所有表，给你一个一致性视图。

### 为啥备份要全局锁？
不锁会出问题。假设有三张表：
- `tb_stock`（库存表）
- `tb_order`（订单表）
- `tb_orderlog`（订单日志表）

备份时：
1. 先备份 `tb_stock`，库存是 10。
2. 有人下单，库存减到 9（`UPDATE tb_stock`），订单加一行（`INSERT INTO tb_order`）。
3. 再备份 `tb_order`，有新订单。
4. 最后备份 `tb_orderlog`。

结果呢？备份里库存是 10，但订单里有买东西的记录，数据不一致！加全局锁后，备份前锁住（`FLUSH TABLES WITH READ LOCK`），没人能改，数据就稳了。

### 怎么用？
```sql
FLUSH TABLES WITH READ LOCK;         -- 加全局锁
mysqldump -uroot -p1234 itcast > itcast.sql  -- 备份
UNLOCK TABLES;                        -- 解锁
```

### 有啥问题？
全局锁很“重”：
- 主库备份：锁住后不能写，业务停摆。
- 从库备份：从库停了主库的二进制日志（Binlog）同步，会有主从延迟。

解决办法：InnoDB 引擎可以用 `mysqldump --single-transaction`，通过 **MVCC（多版本并发控制）** 实现不加锁的备份。

## 3. 表级锁：锁住一本书
### 啥是表级锁？
表级锁就像锁住图书馆里的一本书，管整张表。锁住了，其他人用这张表得按规矩来。粒度大，容易撞车（锁冲突概率高），并发低。MySQL 的 MyISAM 和 InnoDB 都支持表级锁，分三种：
- **表锁（Table Lock）**：普通锁。
- **元数据锁（MDL，Metadata Lock）**：管表结构。
- **意向锁（Intention Lock）**：帮行锁和表锁协调。

### 3.1 表锁：普通的书锁
#### 两种锁
- **表共享读锁（Read Lock）**：
  - 大家都能查（`SELECT`），但不能写。
  - 用 `LOCK TABLES tb_user READ;` 加锁。
- **表独占写锁（Write Lock）**：
  - 只有你能查能写，别人啥也不行。
  - 用 `LOCK TABLES tb_user WRITE;` 加锁。

#### 例子
- 读锁：
  ```sql
  LOCK TABLES tb_user READ;
  SELECT * FROM tb_user;  -- 你能查
  ```
  别人也能 `SELECT`，但 `INSERT` 会阻塞。

- 写锁：
  ```sql
  LOCK TABLES tb_user WRITE;
  UPDATE tb_user SET name = 'Bob' WHERE id = 1;  -- 你能改
  ```
  别人连 `SELECT` 都不行，得等。

#### 特点
- 读锁：不挡读，挡写。
- 写锁：全挡，读写都不行。
- 解锁：`UNLOCK TABLES;` 或客户端断开。

### 3.2 元数据锁（MDL）：管书的简介
#### 啥是 MDL？
MDL 就像锁住书的“简介卡片”（元数据，比如表名、字段名）。它由 MySQL 自动加锁，保护表结构不乱。比如你在查表时，别人不能删表或加字段。

#### 两种锁
- **MDL 读锁（Shared Lock）**：
  - 加在查（`SELECT`）或改数据（`INSERT`）时，大家都能用，但不能改结构。
  - 类型：`SHARED_READ` 或 `SHARED_WRITE`。
- **MDL 写锁（Exclusive Lock）**：
  - 加在改结构（`ALTER TABLE`）时，别人啥也干不了。
  - 类型：`EXCLUSIVE`。

#### 例子
- 查表：
  ```sql
  SELECT * FROM tb_user;
  ```
  加 `SHARED_READ`，别人也能查，但 `ALTER TABLE` 阻塞。

- 改结构：
  ```sql
  ALTER TABLE tb_user ADD COLUMN age INT;
  ```
  加 `EXCLUSIVE`，别人查不了改不了。

#### 为啥有 MDL？
- 避免你在查数据时，表结构变了（比如字段没了），数据乱套。
- MySQL 5.5 引入 MDL，自动管。

#### 查锁
```sql
SELECT object_type, object_schema, object_name, lock_type, lock_duration 
FROM performance_schema.metadata_locks;
```

### 3.3 意向锁：提前打招呼
#### 啥是意向锁？
意向锁像在书封面贴便签，说：“我要用里面的页！”它是 InnoDB 的表级锁，帮行锁和表锁不打架。

#### 两种锁
- **意向共享锁（IS，Intention Shared）**：
  - “我要读几行！”（`SELECT ... LOCK IN SHARE MODE`）
- **意向排他锁（IX，Intention Exclusive）**：
  - “我要改几行！”（`INSERT`、`UPDATE`）

#### 例子
- 改一行：
  ```sql
  UPDATE tb_user SET name = 'Bob' WHERE id = 1;
  ```
  加 IX 锁（表级），再加行级排他锁（X）。

- 别人想锁表：
  ```sql
  LOCK TABLES tb_user WRITE;
  ```
  看到 IX 锁，得等。

#### 为啥有意向锁？
- 没它，锁整张表得检查每行，太慢。
- 有它，一看表上有 IS/IX，就知道有人用行，省事。

#### 查锁
```sql
SELECT * FROM information_schema.innodb_locks;
```

## 4. 行级锁：锁住几页
行级锁就像锁住书里的几页，只管你用到的行。InnoDB 用得多，锁得细（粒度小），并发高。比如：
```sql
SELECT * FROM tb_user WHERE id = 1 FOR UPDATE;
```
只锁 `id=1` 的行，其他行随便用。

## 5. 小结
- **全局锁**：锁整个数据库，备份用，命令 `FLUSH TABLES WITH READ LOCK`。
- **表锁**：锁一张表，读锁共享（`READ`），写锁独占（`WRITE`）。
- **元数据锁（MDL）**：锁表结构，自动加，读锁共享，写锁排他。
- **意向锁**：表级便签，IS/IX，帮行锁和表锁协调。
- **行级锁**：锁几行，InnoDB 的并发王牌。

锁就像图书馆的规矩，保护数据不乱，让大家轮流用。