<div align="center">

<img src="https://avatars.githubusercontent.com/u/344744146?v=4" width="112" height="112" style="border-radius:50%" alt="Aokiisei" />

<h1>Aokiisei</h1>

<p>做那种自己每天都会打开的小工具。</p>

<p>
<a href="https://github.com/Aokiisei" target="_blank">
<img src="https://img.shields.io/badge/GitHub-181717?style=flat-square&logo=github" alt="GitHub" />
</a>
<a href="https://aokiisei.vxni.ink" target="_blank">
<img src="https://img.shields.io/badge/个人站点-aokiisei.vxni.ink-4A90D9?style=flat-square" alt="个人站点" />
</a>
<a href="mailto:lin_648@qq.com" target="_blank">
<img src="https://img.shields.io/badge/QQ%E9%82%AE%E7%AE%B1-lin__648%40qq.com-12B7F5?style=flat-square&logo=tencentqq&logoColor=white" alt="QQ 邮箱" />
</a>
<a href="https://github.com/Aokiisei/-Mimi-/releases/latest" target="_blank">
<img src="https://img.shields.io/github/v/release/Aokiisei/-Mimi-?style=flat-square&label=最新版本&color=2F7CF6" alt="最新版本" />
</a>
</p>

</div>

---

## 👋 关于我

武夷学院在读，写 Android 应用。

喜欢先解决自己身边的问题 —— 课表查起来麻烦，于是有了 Mimi。

信奉“能跑起来的东西才有价值”，所以每个项目都会真刀真枪地编译、装上手机、每天用。

---

## 🚀 主力项目

### 📅 [Mimi 课表](https://github.com/Aokiisei/-Mimi-)

武夷学院课程安排与成绩查询工具。把教务系统里散落的课表和分数，收进一个能随手打开的地方。

<p>
<a href="https://github.com/Aokiisei/-Mimi-/releases/latest" target="_blank">
<img src="https://img.shields.io/github/v/release/Aokiisei/-Mimi-?style=flat-square&label=Release&color=2F7CF6" alt="Release" />
</a>
<a href="https://github.com/Aokiisei/-Mimi-/releases/latest" target="_blank">
<img src="https://img.shields.io/badge/APK-下载-3DDC84?style=flat-square&logo=android" alt="下载 APK" />
</a>
<a href="https://github.com/Aokiisei/-Mimi-/blob/main/LICENSE" target="_blank">
<img src="https://img.shields.io/github/license/Aokiisei/-Mimi-?style=flat-square&label=License" alt="License" />
</a>
<a href="https://github.com/Aokiisei/-Mimi-" target="_blank">
<img src="https://img.shields.io/github/stars/Aokiisei/-Mimi-?style=flat-square&label=Stars" alt="Stars" />
</a>
<img src="https://img.shields.io/badge/Android-7.0%2B-3DDC84?style=flat-square" alt="Android 7.0+" />
</p>

**功能**

- **课程表** —— 按周次查看当日课程，支持周次切换与全屏查看
- **成绩查询** —— 分学期拉取成绩，查看已修课程与分数
- **桌面小组件** —— 主屏直接看当天课程，不用打开应用
- **公交查询** —— 查询武夷山公交线路与到站信息
- **日程管理** —— 倒计日、单点提醒，卡片右滑标记完成
- **暖纸主题** —— 暖色调纸质视觉风格，全 App 统一衬线字体

**工程现状**：完整工程源码开源（Apache-2.0），内置 Gradle Wrapper，
`clone` 后按 README 配置即可自行编译出签名 APK —— 不是只有文档的空仓库。

#### 🏗️ 数据是怎么流动的

教务系统没有开放接口，所以整条链路是自己打通的：先模拟登录拿到会话，
再让 WebView 带着这个会话去取页面，最后把 HTML 解析成结构化数据落到本地。

```mermaid
flowchart LR
    subgraph Edu["🏫 武夷学院教务系统"]
        A["jwxt.wuyiu.edu.cn<br/>课表 / 成绩页面"]
    end

    subgraph App["📱 Mimi 课表"]
        B["OkHttp<br/>模拟登录 · 维持会话"]
        C["WebView<br/>带 Cookie 渲染页面"]
        D["Java 解析层<br/>HTML → 结构化数据"]
        E["SharedPreferences<br/>本地缓存"]
        F["原生 UI<br/>暖纸主题 · 衬线字体"]
        G["桌面小组件<br/>AppWidgetProvider"]
    end

    A -->|HTML / JSON| B
    B -->|Cookie 注入| C
    C -->|页面源码| D
    D -->|课程 · 成绩对象| E
    E --> F
    E --> G

    style A fill:#EEF3FA,stroke:#4A90D9,stroke-width:1px
    style B fill:#F5F1E8,stroke:#C9A063,stroke-width:1px
    style C fill:#F5F1E8,stroke:#C9A063,stroke-width:1px
    style D fill:#F5F1E8,stroke:#C9A063,stroke-width:1px
    style E fill:#EDF7EE,stroke:#5C9E6E,stroke-width:1px
    style F fill:#FBEEEA,stroke:#D98A6C,stroke-width:1px
    style G fill:#FBEEEA,stroke:#D98A6C,stroke-width:1px
```

#### 🔐 会话怎么保住

教务系统的会话会过期，这是这类工具最容易翻车的地方 ——
自动跳回登录页就什么都查不了了。所以做了会话超时的主动判断与无感续期：

```mermaid
sequenceDiagram
    autonumber
    participant U as 用户
    participant A as Mimi 课表
    participant N as OkHttp
    participant S as 教务系统

    U->>A: 打开课表
    A->>A: 读取本地 Cookie
    alt Cookie 有效
        A->>N: 携带 Cookie 请求
        N->>S: GET 课表页面
        S-->>N: 200 返回课表 HTML
        N-->>A: 页面源码
        A->>A: 解析 → 缓存 → 刷新 UI / 小组件
    else Cookie 已过期
        A->>N: 携带过期 Cookie 请求
        N->>S: GET 课表页面
        S-->>N: 302 跳回登录页
        N-->>A: 检测到登录页特征
        A->>N: 自动重新登录（已存凭据）
        N->>S: POST 账号密码
        S-->>N: 新 Cookie
        N-->>A: 更新会话
        A->>N: 重放原请求
        N->>S: GET 课表页面
        S-->>N: 200 返回课表 HTML
        A->>A: 解析 → 缓存 → 刷新 UI / 小组件
    end
    A-->>U: 展示本周课程
```

---

## 🧰 技术栈

<p>
<img src="https://img.shields.io/badge/Java-17-ED8B00?style=flat-square&logo=openjdk&logoColor=white" alt="Java" />
<img src="https://img.shields.io/badge/Android%20SDK-34-3DDC84?style=flat-square&logo=android&logoColor=white" alt="Android SDK" />
<img src="https://img.shields.io/badge/Gradle-8.9-02303A?style=flat-square&logo=gradle&logoColor=white" alt="Gradle" />
<img src="https://img.shields.io/badge/WebView-E1E4E8?style=flat-square&logo=webassembly&logoColor=black" alt="WebView" />
<img src="https://img.shields.io/badge/Material%20Design-757575?style=flat-square&logo=materialdesign&logoColor=white" alt="Material Design" />
<img src="https://img.shields.io/badge/OkHttp-4.12-2F2F2F?style=flat-square&logo=square&logoColor=white" alt="OkHttp" />
<img src="https://img.shields.io/badge/Gson-2.10-2F2F2F?style=flat-square&logo=google&logoColor=white" alt="Gson" />
</p>

---

## 📬 找我

- 🌐 个人站点：[aokiisei.vxni.ink](https://aokiisei.vxni.ink)
- 💻 GitHub：[@Aokiisei](https://github.com/Aokiisei)
- 📦 项目发布：[Mimi 课表 Releases](https://github.com/Aokiisei/-Mimi-/releases)
- ✉️ QQ 邮箱：[lin_648@qq.com](mailto:lin_648@qq.com)

交流、建议、Bug 反馈都欢迎直接邮件，我看到会回。

---

<div align="center">

<sub>想法很多，慢慢写。</sub>

</div>
