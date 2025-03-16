## 前言：Docker 数据为啥会丢？

如果你是 Docker 新手，可能遇到过这样的尴尬：辛辛苦苦跑了个 MySQL 容器，建了数据库、加了数据，结果一删容器，全没了！这是因为 Docker 容器默认是“一次性纸杯”，用完就丢，里面的东西（数据）也跟着没了。

那咋办？那就用 **数据卷**！它能让数据“活下来”，即使容器没了也能找回来。

这篇博客，从零开始，搞懂 Docker 的数据卷和数据卷容器。

------

## 一、数据卷是什么

**数据卷（Volume）** 是 Docker 用来存数据的“神器”，简单说，就像容器的“移动硬盘”：

- **没数据卷**：容器删了，数据全丢。
- **有数据卷**：数据存到“硬盘”里，删容器也不怕，下次还能接着用。

专业点讲，数据卷是 Docker 提供的一种持久化存储机制，把数据从容器的文件系统剥离出来，存在宿主机（运行 Docker 的服务器）上，生命周期独立于容器。

### 比喻

想象容器是个手机，数据是照片：

- 没数据卷：手机丢了，照片没了。
- 有数据卷：照片存到 U 盘或云盘，手机丢了再买个新的，照片还能用。

------

## 二、数据卷怎么用？

Docker 数据卷有两种用法，小白也能上手：

### 1. 宿主机路径挂载（`-v 宿主机:容器`）

- **啥意思**：把宿主机上的一个文件夹“挂”到容器里，容器用这个文件夹存数据。

- **咋操作**：

  ```bash
  docker run -d \
      --name=c_mysql \
      -p 3307:3306 \
      -v /mysql/data:/var/lib/mysql \
      -e MYSQL_ROOT_PASSWORD=123456 \
      mysql:latest
  ```

  - `/mysql/data`：宿主机文件夹（您自己建）。
  - `/var/lib/mysql`：容器里 MySQL 存数据的地方。

**效果**：MySQL 数据存到 `/mysql/data`，删容器后，宿主机上还有数据。

**优点**：您能直接在宿主机上看和管理数据，像操作本地文件夹。

#### 小实验

1. 创建文件夹：

   ```bash
   mkdir -p /mysql/data
   ```

2. 跑容器，加点数据：

   ```bash
   docker exec -it c_mysql mysql -uroot -p123456 -e "CREATE DATABASE mydb;"
   ```

3. 删容器：

   ```bash
   docker rm -f c_mysql
   ```

4. 再跑新容器：

   ```bash
   docker run -d \
       --name=c_mysql \
       -p 3307:3306 \
       -v /mysql/data:/var/lib/mysql \
       -e MYSQL_ROOT_PASSWORD=123456 \
       mysql:latest
   ```

5. 检查：

   ```bash
   docker exec -it c_mysql mysql -uroot -p123456 -e "SHOW DATABASES;"
   ```

   输出有 `mydb`，数据没丢！

### 2. Docker 管理的卷（`-v 容器路径`）

- **啥意思**：不指定宿主机路径，让 Docker 自己弄个“云盘”存数据。

- **咋操作**：

  ```bash
  docker run -d \
      --name=c_mysql \
      -p 3307:3306 \
      -v /var/lib/mysql \
      -e MYSQL_ROOT_PASSWORD=123456 \
      mysql:latest
  ```

  - `/var/lib/mysql`：容器里的路径。
  - Docker 自动在 `/var/lib/docker/volumes` 下建个卷。

**效果**：数据存到 Docker 的“云盘”，删容器后卷还在。

**查看卷**：

```bash
docker volume ls
```

输出：

```
DRIVER    VOLUME NAME
local     abcdef123456
```

**优点**：Docker 全程管理，您不用操心路径。

#### 小实验

1. 跑容器，加数据（同上）。

2. 删容器：

   ```bash
   docker rm -f c_mysql
   ```

3. 再跑新容器，用相同路径 `-v /var/lib/mysql`。

4. 检查数据还在，完美！

------

## 三、数据卷容器

**数据卷容器** 是 Docker 早期的一个玩法，不是“数据卷”本身，而是一个“搬运工容器”：

- **作用**：专门带着数据卷，给其他容器用。
- **咋弄**：创建一个不运行的容器，挂上卷，其他容器通过 `--volumes-from` 借用。

#### 例子

1. 创建数据卷容器：

   ```bash
   docker create --name=data_container -v /data busybox
   ```

   - `busybox`：轻量镜像。
   - `/data`：卷的路径。

2. 跑 MySQL 容器借用：

   ```bash
   docker run -d \
       --name=c_mysql \
       --volumes-from data_container \
       -e MYSQL_ROOT_PASSWORD=123456 \
       mysql:latest
   ```

3. 检查共享：

   ```bash
   docker exec -it c_mysql bash
   echo "Hello" > /data/test.txt
   exit
   docker run --rm --volumes-from data_container busybox cat /data/test.txt
   ```

   输出：`Hello`，数据共享成功！

------

## 四、数据卷 vs 数据卷容器：有啥不一样？

| 特点         | 普通数据卷（Volume）                | 数据卷容器                   |
| :----------- | :---------------------------------- | :--------------------------- |
| **创建方式** | `-v /容器路径` 或 `-v 宿主机:容器`  | `docker create -v /路径`     |
| **管理方式** | Docker 直接管（`docker volume ls`） | 通过容器（`--volumes-from`） |
| **使用场景** | 直接挂到容器，简单方便              | 多个容器共享卷（老方法）     |
| **现代用法** | 主流，推荐                          | 不常用，被 Volume 取代       |

**为啥数据卷容器不火了？**

- 普通卷更简单：直接 `-v`，不用多建个容器。
- 管理方便：`docker volume create`、`docker volume rm`，一目了然。
- 数据卷容器麻烦：得维护一个“搬运工”，容易忘。

------

## 五、小白常见问题

### 数据存哪儿了？

- **宿主机挂载**：您指定的路径（比如 `/mysql/data`）。
- **Docker 卷**：`/var/lib/docker/volumes` 下的随机文件夹，用 `docker volume inspect` 看具体位置。

### 不加 `-v` 会咋样？

- 数据只存容器里，删容器就丢，像没插 U 盘就拍照。

### 怎么选方式？

- **想自己管**：用宿主机挂载。
- **懒得管**：用 Docker 卷。

------

## 六、总结

- **数据卷**：容器的“移动硬盘”，让数据不丢。
- **两种用法**：
  - **宿主机挂载（-v /宿主机:/容器）**：自己指定位置。
  - **Docker 卷（-v /容器路径）**：Docker 自动管。
- **数据卷容器**：老派“搬运工”，带着卷给别人用，现在不流行。