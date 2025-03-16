# Mermaid + AI：一键生成流程图和架构图，程序员的“偷懒”神器

## 前言：从“画图恐惧症”到“AI救星”

作为一名程序员，你有没有遇到过这样的场景：老板或产品经理兴冲冲地跑过来，说：“小张啊，明天开会前把系统架构图画出来，顺便加个流程图，简单点就行！”你心里一万个“简单点？”，打开 Visio 或 Draw.io，手还没动，心已经累了。

画图这事儿，费时费力不说，还经常被挑剔：“这个箭头歪了”“这个框太小了”“能不能再直观点”……

别慌，今天我要介绍一个“偷懒”神器组合——**Mermaid + AI**。有了它，画流程图和系统架构图就像写代码一样简单，甚至还能让 AI 帮你自动生成，省时省力还省心！这篇文章不仅会带你入门，还会让你笑着学会如何“忽悠”老板：这图是我熬夜画的，其实只花了5分钟。

------

## Mermaid 是什么？程序员的“文本画图魔法”

先来认识一下 Mermaid。它是一个基于 JavaScript 的工具，能让你用类似 Markdown 的文本语法生成各种图表，比如流程图、时序图、类图，甚至系统架构图。它的核心理念是“代码即图表”，写几行文本，就能自动渲染出漂亮的图形。

比如，一个简单的流程图：

```
flowchart TD
    A[开始] --> B{判断}
    B -->|是| C[执行]
    B -->|否| D[结束]
```

复制到 [Mermaid Live Editor](https://mermaid.live)

![1742130883022](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1742130883022.png)

秒变流程图！是不是比拖拽鼠标轻松多了？

Mermaid 的优点：

- **简单易学**：语法直白，程序员一看就懂。
- **版本控制友好**：文本格式，直接丢进 Git，告别“图丢了咋办”的尴尬。
- **跨平台**：支持 GitHub、VS Code、Obsidian 等多种环境。

但手动写 Mermaid 代码还是有点麻烦，尤其是复杂的系统架构图。这时，AI 就派上用场了！

------

## AI 登场：从需求描述到图表，一气呵成

想象一下，你只需要告诉 AI：“帮我画个文件管理系统的流程图，包含登录、创建文件、修改文件和查看文件。”然后 AI 直接吐出一段 Mermaid 代码，复制粘贴，图表就出来了。这不是梦，这是现实！

目前，许多 AI 工具（如 ChatGPT、DeepSeek 或国产大模型）都能生成 Mermaid 语法。以下是我用 DeepSeek测试的一个例子：

### 需求

> 权限控制功能支持读写权限设置和检查。用户创建文件时指定权限（只读或可写），系统记录状态。修改文件时，系统检查权限，若为只读则提示禁止操作。属性查看功能展示文件的类型、大小、时间和权限信息。实现中在文件数据结构中加入权限字段，使用标志位表示状态，确保权限逻辑的正确性。（需要一张流程图）

这个时候我使用以下提示词发给DeepSeek：

> 权限控制功能支持读写权限设置和检查。用户创建文件时指定权限（只读或可写），系统记录状态。修改文件时，系统检查权限，若为只读则提示禁止操作。属性查看功能展示文件的类型、大小、时间和权限信息。实现中在文件数据结构中加入权限字段，使用标志位表示状态，确保权限逻辑的正确性。**画一张流程图（使用Mermaid格式）**

### AI 输出

```
flowchart TD
    A[用户创建文件] --> B[指定文件权限]
    B --> C{权限类型}
    C -->|只读| D[系统记录文件为只读]
    C -->|可写| E[系统记录文件为可写]
    D --> F[文件创建完成]
    E --> F[文件创建完成]

    G[用户修改文件] --> H{文件权限}
    H -->|只读| I[系统提示禁止操作]
    H -->|可写| J[允许修改文件]
    I --> K[操作结束]
    J --> K[操作结束]

    L[用户查看文件属性] --> M[系统获取文件信息]
    M --> N[展示文件类型、大小、时间、权限]
    N --> O[操作结束]
```

粘贴到 Mermaid Live Editor：

![1742131159988](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1742131159988.png)

瞬间生成一个清晰的流程图！AI 不仅帮你理清逻辑，还直接输出代码，简直是程序员的“外挂”。



------

## 实战案例：系统架构图也能 AI 生成

流程图简单，架构图呢？一样不在话下！假设我们要设计一个简单的文件管理系统架构，前端用 Vue，后端用 Spring Boot，数据库用 MySQL。直接丢给 AI：

### 需求

“生成一个文件管理系统的架构图，前端 Vue，后端 Spring Boot，数据库 MySQL，用 Mermaid 表示。”

### AI 输出

```
flowchart LR
    subgraph Frontend[前端 - Vue]
        A[用户界面 UI] --> B[Vue 组件]
        B --> C[API 调用]
        C --> D[Axios HTTP 请求]
    end

    subgraph Backend[后端 - Spring Boot]
        E[API 网关] --> F[控制器 Controller]
        F --> G[服务层 Service]
        G --> H[数据访问层 DAO]
        H --> I[数据库连接]
    end

    subgraph Database[数据库 - MySQL]
        J[文件表]
        J --> K[权限表]
        J --> L[用户表]
    end

    D --> E
    I --> J
```

![1742131310315](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1742131310315.png)

渲染出来就是一个分层清晰的架构图，三层结构一目了然，连模块和表都标注好了。老板看了直点头：“小张这图画得不错啊！”你微微一笑，心想：AI 牛逼。

------

## 有趣的“加戏”：让图表更好看

Mermaid 支持样式调整，比如加颜色、改形状。AI 生成的代码可能比较朴素，我们可以手动加点料。比如，给上面的流程图加点颜色：

```
flowchart TD
    A[用户创建文件] --> B[指定文件权限]
    B --> C{权限类型}
    C -->|只读| D[系统记录文件为只读]
    C -->|可写| E[系统记录文件为可写]
    D --> F[文件创建完成]
    E --> F[文件创建完成]

    G[用户修改文件] --> H{文件权限}
    H -->|只读| I[系统提示禁止操作]
    H -->|可写| J[允许修改文件]
    I --> K[操作结束]
    J --> K[操作结束]

    L[用户查看文件属性] --> M[系统获取文件信息]
    M --> N[展示文件类型、大小、时间、权限]
    N --> O[操作结束]

    style A fill:#f9f,stroke:#333,stroke-width:2px
    style B fill:#bbf,stroke:#333,stroke-width:2px
    style C fill:#f96,stroke:#333,stroke-width:2px
    style D fill:#6f9,stroke:#333,stroke-width:2px
    style E fill:#6f9,stroke:#333,stroke-width:2px
    style F fill:#9cf,stroke:#333,stroke-width:2px
    style G fill:#f9f,stroke:#333,stroke-width:2px
    style H fill:#f96,stroke:#333,stroke-width:2px
    style I fill:#f66,stroke:#333,stroke-width:2px
    style J fill:#6f9,stroke:#333,stroke-width:2px
    style K fill:#9cf,stroke:#333,stroke-width:2px
    style L fill:#f9f,stroke:#333,stroke-width:2px
    style M fill:#bbf,stroke:#333,stroke-width:2px
    style N fill:#6f9,stroke:#333,stroke-width:2px
    style O fill:#9cf,stroke:#333,stroke-width:2px
```

![1742131412678](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1742131412678.png)

绿色、蓝色、橙色，瞬间高大上！如果嫌手动改麻烦，直接告诉 AI：“加点颜色，让开始节点绿色，过程蓝色，决策橙色。”AI 照样能搞定。

------

## 为什么这组合这么香？

1. **效率翻倍**：AI 生成初稿，你微调一下，几分钟搞定。
2. **零基础友好**：不懂画图？没关系，AI 帮你写，Mermaid 帮你画。
3. **文档一体化**：流程图和架构图直接嵌在 Markdown 里，和代码无缝衔接。
4. **装X利器**：老板问：“这图咋画的？”你淡定回答：“手写代码生成的，技术含量高吧！”

------

## 小彩蛋：还能干啥？

除了流程图和架构图，Mermaid + AI 还能生成：

- **时序图**：展示用户和系统的交互。
- **类图**：设计数据结构。
- **Gantt 图**：规划项目进度。

比如：“生成一个登录功能的时序图，用户、认证服务和数据库之间的交互。”

AI 输出：

```
sequenceDiagram
    participant User as 用户
    participant AuthService as 认证服务
    participant Database as 数据库

    User->>AuthService: 提交登录请求（用户名、密码）
    AuthService->>Database: 查询用户信息（用户名）
    Database-->>AuthService: 返回用户信息（包括密码哈希值）
    AuthService->>AuthService: 验证密码（比对哈希值）
    alt 密码正确
        AuthService-->>User: 返回登录成功（生成 Token）
    else 密码错误
        AuthService-->>User: 返回登录失败（错误信息）
    end
```

![1742131514036](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1742131514036.png)

是不是很有趣？动动嘴，图就出来了。

------

## 总结：解放双手，拥抱未来

Mermaid + AI 的组合，就像程序员的“魔法棒”，从繁琐的画图工作中解放出来，让我们专注于更有趣的事情——写代码和摸鱼（划掉）。下次开会前，别再苦哈哈地拖拽鼠标了，试试这个神器组合吧！

**读者互动**：你用过 Mermaid 或 AI 生成过什么图表吗？欢迎留言分享你的“偷懒”经验，或者丢个需求给我，我帮你生成一段 Mermaid 代码！