# AppleDevelopSKILLS

iOS development AI agent skills for UIKit-first, Swift/Objective-C mixed projects.

## 在 iOS 项目中让所有 AI 自动使用这些 Skills

`AGENTS.md` 是 Claude、Codex、Gemini 等主流 AI 编程工具共同读取的项目配置文件。在 iOS 项目根目录放置一份即可，所有 Agent 都会自动加载 iOS skill 路由规则。

### 一键初始化（推荐）

```bash
# 在任意 iOS 项目根目录执行
~/ZDAppleDevelopSkills/scripts/init-ios-project.sh
# 或指定路径
~/ZDAppleDevelopSkills/scripts/init-ios-project.sh ~/MyApp
```

脚本会检测是否为 iOS 项目（`.xcodeproj` / `.xcworkspace` / `Podfile` / `Package.swift`），然后将 `templates/ios-project/AGENTS.md` 复制到目标目录。

### 手动复制

```bash
cp ~/ZDAppleDevelopSkills/templates/ios-project/AGENTS.md /path/to/your/ios/project/
```

### 只针对 Claude Code（单 Agent）

在 `~/.claude/CLAUDE.md` 末尾追加：

```markdown
## iOS Project Skills
When the working directory is an iOS project (contains .xcodeproj, .xcworkspace, Podfile, or .swift/.m files),
proactively invoke the relevant skills: uikit-modern, objective-c-patterns, swift-modern, project-structure-architecture.
```

---

## 安装

### 方式一：AgentSkills CLI（推荐）

```bash
agentskills install faimin/ZDAppleDevelopSkills
```

### 方式二：手动安装（Claude Code）

```bash
# 克隆仓库
git clone https://github.com/faimin/ZDAppleDevelopSkills

# 复制到 Claude Code 全局 skills 目录
cp -r ZDAppleDevelopSkills/skills/* ~/.claude/skills/
```

### 方式三：软链接（跟随仓库更新，git pull 即生效）

```bash
git clone https://github.com/faimin/ZDAppleDevelopSkills ~/ZDAppleDevelopSkills

for skill in uikit-modern objective-c-patterns swift-modern project-structure-architecture; do
    ln -sf ~/ZDAppleDevelopSkills/skills/$skill ~/.claude/skills/$skill
done
```

## 使用方式

Skills 在 Claude Code 处理匹配任务时自动加载。也可在对话中通过 `Skill` 工具显式触发：

| 任务方向 | Skill |
| --- | --- |
| UIKit 页面 / 列表 / 键盘 / 导航 | `uikit-modern` |
| Objective-C / block / runtime / 混编 | `objective-c-patterns` |
| Swift 语言 / 并发 / Combine / SwiftUI | `swift-modern` |
| 架构 / 目录结构 / 包管理 / 三方库 | `project-structure-architecture` |

路由规则详见 `AGENTS.md` Section 5。

---

## Skills 说明

当前仓库内置 4 个 iOS Skills，位于 `skills/`：

### 1. `uikit-modern`

聚焦 UIKit，尤其是 iOS 13 之后的现代 UIKit 写法。

适用场景：

- `UICollectionView` 优先的列表/网格页面
- `DiffableDataSource`
- `CompositionalLayout`
- `UIKeyboardLayoutGuide`
- scene / navigation / presentation
- 旧 UIKit 页面现代化重构

典型提问方式：

- "这个列表页该用 `UITableView` 还是 `UICollectionView`？"
- "如何把旧的 `reloadData` 改成 diffable data source？"
- "键盘遮挡输入框怎么用新 API 处理？"

### 2. `objective-c-patterns`

处理 Objective-C 语言与传统 iOS 模块中的核心模式问题。

适用场景：

- block、ARC、循环引用、`EXC_BAD_ACCESS`
- runtime、swizzle、forwarding、associated object
- category 设计边界
- KVC / KVO / ReactiveObjC (RAC)
- Objective-C 与 Swift 混编边界
- ObjC 编码规范（getter 写法、`({})` 初始化、delegate 防护）

典型提问方式：

- "这个 block 为什么会循环引用？"
- "这里要不要 swizzle？"
- "category 里放这段逻辑合适吗？"

### 3. `swift-modern`

处理 Swift 4.2–6.x 范围内的现代 Swift 开发问题。

适用场景：

- Swift 语法升级、迁移、版本差异
- `async/await`、actor、`Sendable`、并发隔离
- Combine、Codable、`@Observable`
- Swift Testing
- Swift 与 Objective-C 互操作
- SwiftUI-first 的状态与 UI ownership 问题

典型提问方式：

- "这段 Swift 代码应该用 Concurrency 还是 Combine？"
- "Swift 6 下这个 `Sendable` 报错怎么处理？"
- "Codable 同时兼容 `String` 和 `Int` 怎么建模？"

### 4. `project-structure-architecture`

负责工程层面的决策。

适用场景：

- 工程目录结构设计
- Feature-first 分层
- MVVM / MVI（页面级）/ TCA（应用级）选型
- SPM / CocoaPods 选择
- 三方库选型
- 模块边界、资源 bundle

典型提问方式：

- "这个 iOS 工程应该怎么分目录？"
- "这个模块适合 MVVM 还是 MVI？"
- "SPM 和 CocoaPods 这里该怎么选？"

### 与 `AGENTS.md` 的分工

- `AGENTS.md`：仓库级规则、工作流、安全约束、Skill 路由规则、平台能力参考
- `skills/`：可复用的 Apple / iOS 领域知识与决策规则

通用 iOS 规范沉淀到对应 Skill，不要堆回 `AGENTS.md`。

------
------

## TOOLS

- [cc-switch](https://github.com/farion1231/cc-switch): 快速切换AI
- [cc-connect](https://github.com/chenhg5/cc-connect): 在任何聊天工具里，远程操控你的本地 AI Agent
- [ccx](https://github.com/BenedictKing/ccx): CCX是Claude、OpenAI Chat Codex Responses和Gemini的高性能AI API代理和协议翻译网关。它提供了一个统一的入口点、内置的网络管理、故障转移、多密钥管理、渠道编排和模型路由。
- [AgentSkills](https://github.com/chrlsio/agent-skills) - agent skills管理器
- [rtk](https://github.com/rtk-ai/rtk) - 降低LLM token消耗
- [caveman](https://github.com/JuliusBrussee/caveman) - 降低token
- [graphify](https://github.com/safishamsi/graphify) - 项目结构图

## MCP

- [cupertino](https://github.com/mihaelamj/cupertino) ： 本地苹果官方文档MCP
- [apple-docs-mcp](https://github.com/kimsungwhee/apple-docs-mcp): 苹果官方文档MCP
- [string-catalog-mcp](https://github.com/onmyway133/string-catalog-mcp): Xcode国际化
- [lanhu-mcp](https://github.com/MrDgbot/lanhu-mcp)

## SKILLS

### 工程Skill

- [superpowers](https://github.com/obra/superpowers): 一套行之有效的软件开发方法论
- [Trellis](https://github.com/mindfold-ai/Trellis): superpowers轻量替代品
- [gstack](https://github.com/garrytan/gstack)
- [Get Shit Done](https://github.com/gsd-build/get-shit-done): 项目管理系统，快速创建一个项目
- [gsd-sp-enhancement](https://github.com/yiixiahulun/gsd-sp-enhancement)
- [andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills)

### UI Skill

- [ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill): 让AI写出来的UI更美观

### Tools Skill

- [app-store-connect-cli-skills](https://github.com/rudrankriyam/app-store-connect-cli-skills)
- [xcode-cli-skill](https://github.com/dazuiba/xcode-cli-skill): Xcode CLI Agent技能

### 开发Skill

- [swift-ios-skills](https://github.com/dpearson2699/swift-ios-skills): iOS开发skills
- [Swift-UIKit-Skill](https://github.com/Lakr233/Swift-UIKit-Skill)
- [ios-dev-skills](https://github.com/12207480/ios-dev-skills)
- [Super Skills](https://github.com/onmyway133/skills)
- [Swift-Concurrency-Agent-Skill](https://github.com/AvdLee/Swift-Concurrency-Agent-Skill): Swift并发编程Agent技能
- [SwiftUI-Agent-Skill](https://github.com/AvdLee/SwiftUI-Agent-Skill): SwiftUI Agent技能
- [SwiftUI-Agent-Skill](https://github.com/twostraws/SwiftUI-Agent-Skill): SwiftUI Agent技能
- [UIKit-Agent-Skill](https://github.com/tornikegomareli/UIKit-Agent-Skill)
- [dotclaude](https://github.com/FradSer/dotclaude)
- [MixMax Skills](https://github.com/MiniMax-AI/skills)
- [zipic skill](https://github.com/okooo5km/Skills4U/tree/main/skills/zipic)
