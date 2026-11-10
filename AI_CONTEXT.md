# 桌宠（Desktop Pet）· AI 上下文交接文件

**用途**：这是整个项目的「记忆锚点」。项目会在多台电脑、多次 AI 会话中推进，**任何一次新会话都必须先完整读本文件**，再读 `docs/技术路线.md` 与 `docs/决策记录.md`。

**最后更新**：2026-11-10 · v0.1（仅完成路线规划，尚未写任何代码）

---

## 0. AI 协作硬规则

1. **默认不写完整代码。** 用户明确说过「不要直接给出代码，我遇到不会的时候会问你」。所以默认输出：思路、关键 API 名称、设计取舍、伪代码级说明。**只有用户明确说「给我代码 / 帮我写」时才提供可运行实现。**
2. **不擅自推翻决策。** 已定决策见 `docs/决策记录.md`。要改就新增一条 ADR，说明背景与代价，等用户确认。
3. **一次只推进一个里程碑**，做完再进下一个；每个里程碑结束时更新本文件第 6、7、9 节。
4. **编码与路径**：所有文档/源码用 UTF-8；路径含中文与空格（`G:\桌宠`），脚本里必须加引号。
5. **项目文件只放在 `G:\桌宠` 内**；运行时数据放 `%AppData%\DesktopPet\`，不要污染项目目录。
6. **依据事实回答，不凭记忆断言。** 版本号、路径、API 行为、模板差异这类能查证的事，必须先查（读实际文件 / `dotnet --info` / 模板元数据 / 官方文档）再回答；查不到就明说"不确定"，不要猜。
7. **98% 门槛。** 有疑点先问，把握到 98% 再执行；宁可多问一句，也不要猜着动手。

---

## 1. 项目目标

做一个 Windows 桌面宠物：透明背景、常驻桌面、会自己走动/待机/睡觉、可拖动、可点击互动、素材可替换。
优先级：**能跑 → 好看 → 不烦人（不抢焦点、不挡操作、全屏时自动隐藏） → 好玩**。

## 2. 用户偏好与协作方式

- 语言：**C#**（用户暂定，已确认方向可行）
- 学习方式：边做边学，遇到不懂的会主动提问 → **解释要讲清「为什么这么做」，而不是甩一段能跑的代码**
- 进度节奏：用户主导，AI 提供路线、拆解、关键点提醒；不要一口气推到底
- 技能现状：有 Visual Studio / WinForms 的使用经验，**WPF 是第一次接触** → 讲解时可借用 WinForms 概念类比，但要主动点出 XAML、依赖属性、布局面板这些差异
- 可能的形态：项目会包含一个 Visual Studio 的 Windows 桌面程序（见 ADR-002 的 UI 栈选择说明）
- 跨机器协作：靠本文件 + `docs/` 目录同步上下文

## 3. 环境记录

### 机器 A（2026-11-10 实测）

| 项 | 值 |
|---|---|
| OS | Windows 11（NT 10.0.26200） |
| .NET SDK | 10.0.401（仅此一个） |
| 桌面运行时 | Microsoft.WindowsDesktop.App 8.0.31 / 10.0.12 |
| Visual Studio | Community 2026，安装于 `D:\VS\main`（需自查是否勾选「.NET 桌面开发」工作负载） |
| Git | 2.56.0（`C:\Program Files\Git`，winget 安装）；已为 `G:/桌宠` 加 `safe.directory` 例外 |
| 项目根 | `G:\桌宠`（已存在，初始为空；**exFAT 分区**，不支持硬链接/符号链接） |
| 当前工作目录 | `F:\工作区`（与项目目录无关，别搞混） |

### 机器 B / C …（在新机器上首次会话时补填）

| 项 | 值 |
|---|---|
| OS | 待填 |
| .NET SDK | 待填 |
| Visual Studio | 待填 |
| Git | 待填 |
| 项目本地路径 | 待填（可能不是 `G:\桌宠`，注意路径差异） |

## 4. 技术选型摘要（详见 `docs/决策记录.md`）

- 语言/运行时：C# + .NET 10（`net10.0-windows`）
- UI 宿主：**WPF（推荐，待用户确认）**，备选 WinForms
- 架构：`DesktopPet.Core`（纯逻辑，无 UI 依赖）+ `DesktopPet`（WPF 宿主）+ `DesktopPet.Core.Tests`
- 动画：精灵图集（sprite sheet）+ 元数据，时间驱动的帧循环
- 配置：`%AppData%\DesktopPet\config.json`
- 发布：`dotnet publish` 自包含单文件
- 版本管理：Git（**待安装**，是跨机器同步的前提）

## 5. 目录结构约定

```
G:\桌宠\
├─ DesktopPet.slnx         ← 解决方案（VS 2026 新格式），双击打开
├─ AI_CONTEXT.md           ← 本文件，最先读
├─ README.md               ← 人与 AI 的入口
├─ docs\                   ← 技术路线 / 决策记录
├─ assets\                 ← 美术 / 音效素材
├─ DesktopPet\             ← WPF 宿主（可执行程序）：窗口、托盘、拖拽、屏幕信息、开机自启
├─ DesktopPet.Core\        ← 状态机、物理、动画时间轴、配置模型（无 UI 依赖）
└─ DesktopPet.Core.Tests\  ← Core 的单元测试
```

> 采用**扁平布局**（与 VS 新建项目默认一致，见 ADR-008）。除 `docs\` 外其余项目逐个在 M0 创建。

## 6. 当前进度

- 阶段：**M0 进行中**（骨架与工程配置已完成，只剩 Git）
- 已完成：
  - WPF 项目 `DesktopPet`（`net10.0-windows`，`UseWPF`），`dotnet build` 通过
  - `DesktopPet.Core`（`net10.0` 类库）+ `DesktopPet.Core.Tests`（xUnit 2.9.3），Tests 已引用 Core
  - `DesktopPet.slnx` 已在仓库根目录，项目路径改为 `DesktopPet\DesktopPet.csproj`
  - `.gitignore` / `global.json`（SDK 10.0.100 起，latestFeature）/ `Directory.Build.props` 已就位
- 未开始：Git 仓库初始化与 GitHub 远程、任何业务代码、美术素材

## 7. 下一步（按顺序，一次一步）

1. 安装 Git for Windows（`winget install --id Git.Git -e --source winget`），核对 `user.name` / `user.email`
2. `git init` → 首次提交 → 推送 GitHub（远程仓库待建）
3. 确认 **美术风格**（像素风 / 手绘柔和风）
4. **M1**：第一个「能看见、能拖、能关」的透明置顶窗口
5. M2 起按 `docs/技术路线.md` 的里程碑走

## 8. 待确认问题

- [ ] UI 栈：WPF（推荐）还是 WinForms？
- [ ] 美术风格与素材来源：自己画 / 约稿 / 免费素材？基准分辨率多少？
- [ ] 目标系统范围：只 Win10/11 x64，还是要兼容 Win7？
- [ ] 是否要开机自启、常驻托盘？
- [ ] 多台机器是否用同一个 Git 远程仓库（GitHub/Gitee/自建）同步？

## 9. 变更日志

| 日期 | 版本 | 内容 |
|---|---|---|
| 2026-11-10 | v0.1 | 建立上下文文件；记录机器 A 环境；产出技术路线与 ADR 草案 |
| 2026-11-10 | v0.2 | 用户确认 UI 栈为 WPF；目录结构改为扁平布局（取消 src/tests 分层）；补充 IDE 建项目要点 |
| 2026-11-10 | v0.3 | 用户追加规则：必须依据事实/文档作答、98% 把握再执行；已用 VS 创建 WPF 项目并验证可编译 |
| 2026-11-10 | v0.4 | slnx 移至仓库根并修正项目路径；App 项目实际名为 `DesktopPet`（非 DesktopPet.App）；构建验证通过 |
| 2026-11-10 | v0.5 | Core/Tests 项目建成并互相引用；加入 .gitignore、global.json、Directory.Build.props；构建与测试通过；下一步接 Git |
| 2026-11-10 | v0.6 | 安装 Git 2.56.0；解决 exFAT 的 dubious ownership（加 safe.directory）；仓库已 init（main，无提交，无远程） |