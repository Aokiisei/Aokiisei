<div align="center">

<img src="https://avatars.githubusercontent.com/u/344744146?v=4" width="104" height="104" style="border-radius:50%" alt="Aokiisei" />

<h1>Aokiisei</h1>

<p>武夷学院在读 · 写 Android 应用</p>

<p>
<a href="https://aokiisei.vxni.ink" target="_blank">
<img src="https://img.shields.io/badge/%E4%B8%AA%E4%BA%BA%E7%AB%99%E7%82%B9-aokiisei.vxni.ink-4A90D9?style=flat-square" alt="个人站点" />
</a>
<a href="mailto:lin_648@qq.com" target="_blank">
<img src="https://img.shields.io/badge/QQ%E9%82%AE%E7%AE%B1-lin__648%40qq.com-12B7F5?style=flat-square&logo=tencentqq&logoColor=white" alt="QQ 邮箱" />
</a>
</p>

</div>

---

## 关于

写自己每天都会打开的小工具。

课表查起来麻烦，于是有了 Mimi —— 把教务系统里散落的课表和分数，收进一个能随手打开的地方。

习惯是先让它跑起来，装到自己手机上用一阵，哪里别扭就改哪里。

---

## 项目

### [Mimi 课表](https://github.com/Aokiisei/-Mimi-)

武夷学院课程表与成绩查询工具，需要本人的教务账号登录。Android 5.0 及以上，Apache-2.0 开源。

<p>
<a href="https://github.com/Aokiisei/-Mimi-/releases/latest" target="_blank">
<img src="https://img.shields.io/github/v/release/Aokiisei/-Mimi-?style=flat-square&label=Release&color=2F7CF6" alt="Release" />
</a>
<a href="https://github.com/Aokiisei/-Mimi-/blob/main/LICENSE" target="_blank">
<img src="https://img.shields.io/github/license/Aokiisei/-Mimi-?style=flat-square&label=License" alt="License" />
</a>
<img src="https://img.shields.io/badge/Android-5.0%2B-3DDC84?style=flat-square&logo=android&logoColor=white" alt="Android 5.0+" />
</p>

- **课程表** —— 按周次查看，支持「全部周次」，全屏一屏看完整周
- **成绩查询** —— 分学期拉取，可展开每门课的构成
- **上课提醒** —— 课前 10 分钟提醒课程与地点，每晚汇总明日课表
- **校园公告** —— 汇总学校官网与 13 个二级学院公告
- **校园地图标注** —— 同学共同标注校园地点
- **公交查询 / 日程管理 / 桌面小组件**

安装包、更新日志和完整源码都在[仓库](https://github.com/Aokiisei/-Mimi-)里。

---

## 案例分析

记录两个做这个项目时卡住过的问题。光看现象很难想到原因，画出来会清楚一些。

### 一、晚上 19:00 的课不显示

教务页面里明明有这门课，抓下来却少了几节。排查后发现问题出在**「怎么判断一节课属于第几节」**。

教务的课表表格里，同一门课的节次有好几种写法：「第九、十节」「第9-10节」「晚上」。早先是按行标签的先后顺序去数，一遇到合写就整行错位，严重的会连整列一起丢。

改成三级判定后稳定了——**先看编号，编号没有再看标题里的数字，最后才退回按位置数**：

```mermaid
flowchart TD
    A["抓下教务课表表格"] --> B{"怎么确定是第几节？"}

    B -->|"旧做法：按位置数"| C["遇到「第九、十节」这种合写<br/>整行错位 · 整列丢失"]
    B -->|"新做法：三级判定"| D["① 看页面里记录的起始节次"]

    D -->|"没有"| E["② 找行标题里的数字"]
    E -->|"也没有"| F["③ 才退回按位置数"]
    D -->|"找到"| G["节次确定"]
    E -->|"找到"| G
    F --> G

    G --> H["第 9–12 节都能显示<br/>卡片上标注「晚七 / 晚九」"]

    classDef bad fill:#FCEBEB,stroke:#A32D2D,color:#501313
    classDef good fill:#EAF3DE,stroke:#3B6D11,color:#173404
    classDef plain fill:#F1EFE8,stroke:#5F5E5A,color:#2C2C2A
    class C bad
    class G,H good
    class A,B,D,E,F plain
```

顺带发现另一个问题：教务首页是陆续加载出来的，有时只抓到前几门课就返回了。所以登录后会再等一会儿重新抓一次。

### 二、登录过期后一片空白

教务的登录状态会过期。过期后服务器直接把人送回登录页，页面本身不报错——用户看到的是一张空课表，不知道发生了什么。

难点在于**不能把所有异常都当成过期**：如果输错密码也走自动重登，就会反复提交，看起来像卡死。所以判定分两层：

```mermaid
flowchart TD
    A["请求课表页面"] --> B{"拿到的是课表？"}

    B -->|"是"| C["正常显示"]
    B -->|"否"| D{"地址是否跳到统一登录页？"}

    D -->|"是"| E["判定：登录过期了"]
    E --> F["清掉旧登录状态<br/>用记住的账号密码重登<br/>再请求一次"]
    F --> C

    D -->|"否"| G{"页面里有没有「密码错误」这类提示？"}
    G -->|"有"| H["判定：账号密码错了<br/>如实提示，不自动重试"]
    G -->|"没有"| I["按登录超时处理"]

    classDef flow fill:#E6F1FB,stroke:#185FA5,color:#0C447C
    classDef ok fill:#EAF3DE,stroke:#3B6D11,color:#173404
    classDef warn fill:#FAEEDA,stroke:#854F0B,color:#412402
    class A,B,D,G flow
    class C,F ok
    class E,H,I warn
```

第一层看地址有没有被送到统一登录页 —— 这说明「登录确实没了」；第二层再看页面里有没有「密码错误」这类具体提示 —— 这能区分「账号密码真的输错了」。

有个地方容易踩：判据里**特意排除了「验证码」这个词**。登录页本身就有验证码，如果把它当成错误特征，那么任何停在登录页的情况都会被判成"密码错了"，从而永远触发不了自动重登。

另外还加了一条按时间的判断：App 被系统清掉后台后重进，如果停了很久，就直接认定「登录大概过期了」，省掉一次注定失败的请求。

---

## 技术栈

<p>
<img src="https://img.shields.io/badge/Java-8-ED8B00?style=flat-square&logo=openjdk&logoColor=white" alt="Java" />
<img src="https://img.shields.io/badge/Android%20SDK-34-3DDC84?style=flat-square&logo=android&logoColor=white" alt="Android SDK" />
<img src="https://img.shields.io/badge/AGP-8.5.2-3B6D11?style=flat-square&logo=androidstudio&logoColor=white" alt="AGP" />
<img src="https://img.shields.io/badge/Gradle-8.9-02303A?style=flat-square&logo=gradle&logoColor=white" alt="Gradle" />
<img src="https://img.shields.io/badge/WebView-E1E4E8?style=flat-square&logo=html5&logoColor=white" alt="WebView" />
<img src="https://img.shields.io/badge/Material%20Design-757575?style=flat-square&logo=materialdesign&logoColor=white" alt="Material Design" />
<img src="https://img.shields.io/badge/OkHttp-4.12-2F2F2F?style=flat-square&logo=square&logoColor=white" alt="OkHttp" />
<img src="https://img.shields.io/badge/Gson-2.10-2F2F2F?style=flat-square&logo=google&logoColor=white" alt="Gson" />
</p>

---

## 联系

- 站点：[aokiisei.vxni.ink](https://aokiisei.vxni.ink)
- 邮箱：[lin_648@qq.com](mailto:lin_648@qq.com)
- 项目：[Mimi 课表](https://github.com/Aokiisei/-Mimi-)

有问题或者建议，直接发邮件就行。

---

<div align="center">

<sub>想法很多，慢慢写。</sub>

</div>
