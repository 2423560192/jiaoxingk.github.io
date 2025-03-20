以下是将你提供的内容转换为Markdown格式的版本，同时尽量保留了你原本的风格和语气：

# Shell脚本学习笔记：从入门到变量（一）

最近在看 Shell 脚本相关的内容，以下是我从入门到变量部分的整理笔记，内容有点多，但都是干货。

先从基础开始，再逐步深入。

## 一、Shell 脚本入门

### 1\. Linux 如何控制硬件？

Linux 靠**内核**操作硬件（CPU、内存、磁盘、显示器），而我们用户通过 Shell 命令跟内核打交道。简单说，Shell 就是个传话筒。

![null](https://cdn.nlark.com/yuque/0/2025/png/29344884/1742122148492-cf9a9ea3-d32e-454a-a9d0-1b90f3a9de9a.png)

### 2\. Shell 是什么

- **定义**：Shell 是 Linux 的命令解释器，像 Windows 的 DOS，能敲命令干活。
- **特点**：不光是命令，还是一门编程语言，有变量、函数、逻辑控制，写成脚本就能批量跑。
- **Shell 脚本**：把命令攒成文本文件（一般.sh结尾），这就是脚本，也叫 Shell 程序。

### 3\. 为啥学 Shell？

手动敲命令太费劲，脚本一跑，批量处理，效率蹭蹭往上涨，管理 Linux 必备。

### 4\. Shell 咋工作的？

1. 输入命令。
2. Shell 解析后丢给内核。
3. 内核执行完，Shell 把结果显示出来。

### 5\. Shell 解析器

- **查支持的解析器**：`cat /etc/shells`
- **常见类型**：
    - `/bin/sh`：老大哥，UNIX 最早的 Shell。
    - `/bin/bash`：Linux 默认，功能多，界面友好。
    - `/sbin/nologin`：禁止登录，服务用得多。
    - `/bin/dash`：轻量级，交互差点。
    - `/bin/csh` & `/bin/tcsh`：C 风格的 Shell。
- **当前解析器**：`echo $SHELL`，一般是`/bin/bash`。

## 二、Shell 脚本规范

- **文件名**：建议.sh结尾，一看就懂。
- **首行**：得写`#!/bin/bash`，指定用 bash 跑。
- **注释**：
    - **单行**：`#`随便写
    - **多行**：
    ```bash
    :<<!
    # 这儿是注释1
    # 这儿是注释2
    !
    ```

## 三、入门案例：Hello World

### 目标

写个`helloworld.sh`，输出 hello world。

### 步骤

1. 新建：`touch helloworld.sh`

2. 编辑：`vim helloworld.sh`，写：
    ```bash
    #!/bin/bash
    echo "hello world"
    ```
    
3. 保存退出（`:wq`）。

4. 跑：`sh helloworld.sh`，屏幕上就刷出 hello world。

### 三种执行方式

- `sh helloworld.sh`：用 sh 解析器跑。
- `bash helloworld.sh`：用 bash 跑，差不多。
- `./helloworld.sh`：直接跑，得先加权限：`chmod a+x helloworld.sh`。

**区别**：前两种不用权限，第三种得自己加 x。

## 四、多命令案例

### 需求

在`/root/itheima`下建`one.txt`，写上 Hello Shell。

### 步骤

1. 建目录：`mkdir /root/itheima`

2. 新建脚本：`touch batch.sh`

3. 编辑：
    ```bash
    #!/bin/bash
    cd /root/itheima
    touch one.txt
    echo "Hello Shell" >> one.txt
    ```
    
4. 跑：`sh batch.sh`

5. 看结果：`cat /root/itheima/one.txt`，输出 Hello Shell。

![null](https://cdn.nlark.com/yuque/0/2025/png/29344884/1742122149371-50320d44-a020-492e-98f0-1e53927d8127.png)

## 五、Shell 变量详解

### 1\. 变量是啥？

变量就是存数据的临时容器，存在内存里，分三类：

- 系统环境变量
- 自定义变量
- 特殊变量

### 2\. 系统环境变量

- **啥是系统变量**：系统定义好的，共享给所有 Shell 用。
- **配置文件**：
    - **全局**：`/etc/profile`、`/etc/profile.d/*.sh`、`/etc/bashrc`
    - **个人**：`~/.bash_profile`、`~/.bashrc`
- **咋看**：
    - `env`：只看环境变量。
    - `set`：全看，变量、函数都有。
- **常用变量**：
    - `PATH`：命令搜索路径。
    - `HOME`：用户家目录（比如`/root`）。
    - `SHELL`：当前解析器。
    - `PWD`：当前路径。
    - `HISTFILE`：命令历史文件。
- **自定义系统变量**
    - **场景**：想让所有 Shell 都能用某个变量，就写到`/etc/profile`里。
    - **步骤**：
        1. 编辑：`vim /etc/profile`
        2. 加一行：`export VAR1="test"`
        3. 生效：`source /etc/profile`
        4. 验证：`echo $VAR1`，输出 test。

### 3\. 自定义变量

#### （1）局部变量

- **定义**：`name=value`
- **规则**：
    - 字母、数字、下划线，别数字开头。
    - 等号两边没空格。
    - 默认是字符串，想算数得转。
    - 值有空格加双引号。
- **取值**：`$name` 或 `${name}`（拼接用后者，比如`echo "Hi ${name}"`）。
- **删掉**：`unset name`

#### （2）常量

- **定义**：`readonly name`
- **特点**：改不了。

#### （3）全局变量

- **定义**：`export name`
- **范围**：当前脚本和子脚本都能用。
- **案例**：
    - `demo2.sh`：
    ```bash
    #!/bin/bash
    export VAR4="test"
    sh demo3.sh
    ```
    - `demo3.sh`：
    ```bash
    #!/bin/bash
    echo $VAR4
    ```
    - 跑`sh demo2.sh`，输出 test。

### 4\. 特殊变量

- `$n`：脚本参数，`$0`是脚本名，`$1~$9`是第 1 到 9 个参数，`${10}`及以上。
- `$#`：参数个数。
- `$*` & `$@`：所有参数。
    - **不加引号**：一样，`$1 $2` ...。
    - **加引号**：`"$*"`是一整串，`"$@"`是列表，用 for 循环能看出区别：
    ```bash
    for i in "$@"; do echo $i; done
    ```
- `$?`：上个命令返回值，0 是成功，非 0 是失败。
- `$$`：当前 Shell 进程 ID，`ps -aux | grep bash`能对上。
- **案例：玩特殊变量**
    - `demo4.sh`：
    ```bash
    #!/bin/bash
    echo "脚本名: $0"
    echo "参数1: $1"
    echo "参数2: $2"
    echo "参数10: ${10}"
    echo "参数个数: $#"
    echo "所有参数(\$*): $*"
    echo "循环\$@:"
    for i in "$@"; do echo $i; done
    ```
    - 跑：`sh demo4.sh a b c d e f g h i j`，看看效果。

### 5\. Shell 环境深入

#### 交互式 vs 非交互式

- **交互式**：敲命令，马上跑，跑完反馈。
- **非交互式**：脚本跑命令，不跟你聊。

#### 登录 vs 非登录

- **登录 Shell**：要用户名密码，加载`/etc/profile`和`~/.bashrc`。
- **非登录 Shell**：直接跑，只加载`~/.bashrc`。

#### 测试

- 加变量：`/etc/profile`里`export VAR1=1`，`~/.bashrc`里`export VAR2=2`。
- 脚本`demo1.sh`：`echo $VAR1 $VAR2`。
    - `bash demo1.sh`：只出 2。
    - `bash -l demo1.sh`：出 1 2。

#### 切换环境

- 直接登录：用户名密码进系统。
- `su user -l`：切用户，加载登录环境。
- `bash`：非登录环境。

## 六、总结

Shell 脚本入门到变量，内容不少：

- **基础**：命令、脚本、解析器。
- **变量**：系统变量管全局，自定义变量分局部和全局，特殊变量超实用。
- **环境**：登录和非登录区别得搞清楚，影响变量加载。
