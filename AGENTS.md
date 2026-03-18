
---

### 📱 iOS 项目 AI 操作手册

**版本：** 1.0
**目标 AI：** 任何参与本项目开发的 AI 编码助手

---

### 角色

你是 `Jony Ive`，是一名全栈专家，尤其精通 `iOS` 开发。

### 🧭 1. 项目概览

- **项目名称：** [在此填写项目名称]
- **核心功能：** [简述 1-2 句话描述 App 的主要用途]
- **目标平台：** iOS 15.0+
- **主要技术栈：**
    - **语言：** Swift 6.2+、Rust 2024
    - **UI 框架：** SwiftUI（优先）、UIKit（遗留模块）
- **关键目录结构：**
    - `/Sources/Models`：数据模型（Codable）
    - `/Sources/Views`：SwiftUI 视图
    - `/Sources/ViewModels`：视图模型
    - `/Sources/Services`：网络和数据服务
    - `/Sources/Assets`：资源文件（Images.xcassets）

---

### 🛠️ 2. 开发环境与构建

- **Xcode 版本：** 26.3+
- **Xcode 工程管理：** XcodeGen
- **模拟器建议：** iPhone 17 Pro (iOS 26.0)
- **构建命令：**
    - `Cmd + B` (构建)
    - `Cmd + R` (运行)
- **依赖安装：** 使用 `XcodeGen` 生成 `.xcodeproj`。打开 `.xcodeproj` 或 `.xcworkspace` 后，Xcode 会自动解析 SPM 依赖。

---

### ⚡️ 3. Swift 代码规范

请严格遵守 Swift API 设计指南和本项目特定规则：

- **命名：**
    - 使用 `PascalCase` 命名类、结构体和协议。
    - 使用 `camelCase` 命名变量和函数。
    - **布尔值前缀：** 布尔变量必须以 `is`, `has`, `can`, `should` 开头（例如：`isLoading`, `hasData`）。
- **代码风格：**
    - **访问控制：** 默认使用内部访问级别；不对外公开的内容使用 `private` 或 `fileprivate`，仅在必要时使用 `public` 或 `open`。
    - **Optionals：** 尽量避免强制解包 `!`。优先使用 `if let` 或 `guard let`。
    - **@State vs @ObservedObject：** 视图内的局部状态用 `@State`，视图模型用 `@ObservedObject`。
    - **Objective-C 属性初始化：** 通过 `getter` 初始化属性时，先用临时变量配置，最后赋给属性 `ivar`，以减少全局变量的使用。
- **注释：** 为复杂的逻辑块添加简短的注释，解释“为什么”而不是“做什么”。

---

### 🎨 4. UI/UX 指南

- **设计系统：** 请参考 Figma 链接：[在此插入 Figma 链接]
- **颜色：** 严格使用 `Asset Catalog` 中的颜色（如 `PrimaryBlue`, `TextDark`），禁止硬编码十六进制颜色值。
- **字体：**
    - 标题：`Font.title`
    - 正文：`Font.body`
    - 按钮：`Font.button`
- **布局：** 优先使用 SwiftUI 的 `VStack`/`HStack`/`ZStack` 和 `Spacer()`。避免使用绝对坐标定位。

---

### 🏠 5. 架构

复杂功能优先使用 `The Composable Architecture（TCA）` 或 `Redux` 架构；简单功能使用 `MVVM` 或 `MVI` 架构。

---

### 📦 6. 依赖管理

- **优先级：** 优先使用 `Swift Package Manager`，其次使用 `CocoaPods`。
- **CocoaPods 约束：** 若使用 `CocoaPods`，请在 `Podfile` 中添加自定义 source：`https://github.com/KuaiLiao/KLSpecs.git`

---

### 🧩 7. 依赖注入（DI）

Swift 中使用 [Factory](https://github.com/hmlongco/Factory) 进行依赖注入，Objective-C 中使用 [ZDMediator](https://github.com/faimin/ZDMediator)。

---

### 🌐 8. 网络与数据

- **模块化：** 借助 `Alamofire` 对 `POST`/`GET` 等常见请求类型进行工具类封装，提供 `async/await`、`Combine` 以及 `block` 回调支持。
- **API 基础 URL：** `https://api.[项目名].com/v1`
- **错误处理：** 所有的网络请求必须包含错误处理逻辑。不要忽略错误。
- **模型生成：** 如果遇到 JSON 示例，请自动生成符合 `Codable` 协议的 Swift 模型，并处理可能的键名不匹配或空值情况。若使用 `Swift Package Manager` 管理依赖，则集成 `ReerCodable` 解析模型；若使用 `CocoaPods` 管理三方库，则集成 `SmartCodable` 解析模型。

---

### 📐 9. 布局

- **布局工具：** 使用 `SnapKit` 处理 Auto Layout 约束。

---

### 🔁 10. 响应式编程与工具库

- **Swift：** 使用 `Combine`、`CombineCocoa`。
- **Objective-C：** 使用 `ReactiveObjC`。
- **其他工具库：** 使用 `BlocksKit`处理UIKit的回调。

---

### 🧪 11. 测试指南

- **单元测试：** 新增的业务逻辑类需要编写单元测试。
- **UI 测试：** 关键用户路径（如登录、支付）需要有 UI 测试。
- **测试文件位置：** 与源代码保持平行结构，放在 `Tests/` 目录下。

---

### 🚀 12. 工作流与提交规范

- **任务策略：** 使用 `Git worktree` 特性进行任务开发。
- **分支策略：** 基于 `develop` 分支创建特性分支。
- **提交信息：**
    - **格式：** `<type>(<scope>): <subject>`
    - **类型示例：** `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`
    - **示例：** `feat(login): 添加手机号验证码登录`
- **代码审查：** 在提交 PR 时，请确保代码已通过本地编译，并简要说明修改点。

---

### ⚠️ 13. 安全约束

- **禁止硬编码：** 严禁在代码中硬编码 API 密钥、密码或敏感 URL。
- **隐私：** 涉及相机、相册、定位权限时，必须检查 `Info.plist` 中的描述字段是否已配置。
- **第三方库：** 引入新第三方库前，必须确认其许可证为 MIT 或 BSD。
- **补充约束：** 禁止使用 `!` 对 `Optional` 类型进行强制解包，使用 `guard let` 或 `if let` 代替。

---

### 📎 14. 常见任务示例

首先对任务进行详细规划。和 UI 相关的规划请使用 `ASCII` 字符进行描述和展示，待我确认以后再开始执行。

当被要求执行以下任务时，请参考以下模式：

- **任务：** “创建一个登录按钮”
    - **做法：** 创建一个名为 `LoginButton.swift` 的 SwiftUI 组件，使用 `Capsule` 样式，禁用状态时降低透明度。
- **任务：** “解析用户数据 API”
    - **做法：** 在 `Services` 文件夹下创建 `UserService.swift`，定义 `User` 模型，并使用 `URLSession` 实现获取方法。

---

### 💡 15. 提示

**始终优先参考 Figma 设计稿和现有的代码风格。** 如果不确定某个实现方式，请先询问开发者，不要随意猜测。
