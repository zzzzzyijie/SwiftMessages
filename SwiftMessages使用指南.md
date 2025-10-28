# SwiftMessages 使用指南

## 项目概述

SwiftMessages 是一个功能强大且极其灵活的 iOS 消息提示和视图控制器展示库。它可以让您轻松地在应用中展示各种风格的消息通知和弹窗。

### 主要特性

- ✅ **灵活的位置选择**：支持在屏幕顶部、底部或中心显示消息
- ✅ **多种展示方式**：可以显示在导航栏下方、标签栏上方或独立窗口中
- ✅ **交互式手势**：支持滑动关闭，包括基于物理引擎的有趣手势效果
- ✅ **背景遮罩**：多种背景变暗模式（灰色、自定义颜色、模糊效果）
- ✅ **内置主题**：提供 info、success、warning、error 四种主题
- ✅ **多种布局**：messageView、cardView、tabView、statusLine、centeredView
- ✅ **高度可定制**：支持完全自定义视图和样式
- ✅ **消息队列**：自动管理消息队列，一次显示一条消息
- ✅ **键盘避让**：自动躲避键盘
- ✅ **无障碍支持**：完善的 VoiceOver 支持
- ✅ **视图控制器展示**：通过 SwiftMessagesSegue 展示视图控制器
- ✅ **暗黑模式支持**：iOS 13+ 自动适配暗黑模式

---

## 安装方式

### Swift Package Manager（推荐）

在 Xcode 中：
1. 选择 `File` → `Swift Packages` → `Add Package Dependency...`
2. 搜索 "SwiftMessages"
3. 选择 SwiftKick Mobile 提供的版本

### CocoaPods

在 `Podfile` 中添加：

```ruby
pod 'SwiftMessages'
```

### Carthage

在 `Cartfile` 中添加：

```ruby
github "SwiftKickMobile/SwiftMessages"
```

### 手动安装

1. 将 SwiftMessages 仓库放到项目目录中
2. 在 Xcode 中添加 `SwiftMessages.xcodeproj` 到项目
3. 在 app target 中添加 SwiftMessages framework 作为嵌入式二进制文件

---

## 基础用法

### 最简单的使用方式

```swift
import SwiftMessages

// 显示任意 UIView
SwiftMessages.show(view: myView)
```

### 使用内置的 MessageView

```swift
import SwiftMessages

// 从内置的 card 视图布局创建消息视图
let view = MessageView.viewFromNib(layout: .cardView)

// 应用警告主题
view.configureTheme(.warning)

// 添加阴影
view.configureDropShadow()

// 设置标题、正文和图标（这里用 emoji 替代默认图标）
let iconText = ["🤔", "😳", "🙄", "😶"].randomElement()!
view.configureContent(title: "警告", body: "请注意这个警告信息。", iconText: iconText)

// 增加外边距
view.layoutMarginAdditions = UIEdgeInsets(top: 20, left: 20, bottom: 20, right: 20)

// 减小圆角半径
(view.backgroundView as? CornerRoundingView)?.cornerRadius = 10

// 显示消息
SwiftMessages.show(view: view)
```

### 使用 ViewProvider（推荐用于后台线程）

当消息可能从后台线程添加时，使用 `viewProvider` 确保 UIKit 代码在主队列执行：

```swift
SwiftMessages.show {
    let view = MessageView.viewFromNib(layout: .cardView)
    view.configureTheme(.success)
    view.configureContent(title: "成功", body: "操作已完成！")
    return view
}
```

---

## 配置选项

### Config 结构体

`SwiftMessages.Config` 提供了丰富的配置选项：

```swift
var config = SwiftMessages.Config()

// 从底部滑入
config.presentationStyle = .bottom

// 在指定窗口层级显示
config.presentationContext = .window(windowLevel: .statusBar)

// 注意：从 iOS 13 开始，无法覆盖状态栏
// 解决方法是隐藏状态栏
config.prefersStatusBarHidden = true

// 禁用自动隐藏
config.duration = .forever

// 背景变暗，点击背景关闭
config.dimMode = .gray(interactive: true)

// 禁用交互式滑动关闭手势
config.interactiveHide = false

// 指定状态栏样式
config.preferredStatusBarStyle = .lightContent

// 添加事件监听器
config.eventListeners.append { event in
    if case .didHide = event {
        print("消息已隐藏，id=\(String(describing: event.id))")
    }
}

// 显示消息
SwiftMessages.show(config: config, view: view)
```

### 展示位置（PresentationStyle）

```swift
config.presentationStyle = .top      // 从顶部滑入
config.presentationStyle = .bottom   // 从底部滑入
config.presentationStyle = .center   // 在中心淡入
config.presentationStyle = .custom(animator: myAnimator) // 自定义动画
```

### 展示上下文（PresentationContext）

```swift
// 自动选择（默认）
config.presentationContext = .automatic

// 在新窗口中显示
config.presentationContext = .window(windowLevel: .normal)
config.presentationContext = .window(windowLevel: .statusBar)

// 在指定 window scene 中显示（iOS 13+）
config.presentationContext = .windowScene(scene, windowLevel: .normal)

// 在指定视图控制器中显示
config.presentationContext = .viewController(viewController)

// 在指定视图中显示
config.presentationContext = .view(containerView)
```

### 显示时长（Duration）

```swift
config.duration = .automatic           // 自动时长（默认）
config.duration = .forever             // 永不自动隐藏
config.duration = .seconds(seconds: 3) // 3秒后自动隐藏

// 高级：延迟显示和最小显示时间
config.duration = .indefinite(delay: 2, minimum: 1)
// 延迟2秒显示，如果显示则至少显示1秒
```

### 背景遮罩模式（DimMode）

```swift
config.dimMode = .none  // 无遮罩（默认）

// 灰色遮罩，可交互（点击关闭）
config.dimMode = .gray(interactive: true)

// 自定义颜色遮罩
config.dimMode = .color(color: UIColor.black.withAlphaComponent(0.7), interactive: true)

// 模糊效果遮罩
config.dimMode = .blur(style: .dark, alpha: 1.0, interactive: true)
```

### 设置默认配置

```swift
// 设置全局默认配置
SwiftMessages.defaultConfig.presentationStyle = .bottom
SwiftMessages.defaultConfig.duration = .seconds(seconds: 3)

// 使用默认配置显示
SwiftMessages.show(view: view)

// 基于默认配置自定义
var config = SwiftMessages.defaultConfig
config.duration = .forever
SwiftMessages.show(config: config, view: view)
```

---

## 主题和布局

### 内置主题

SwiftMessages 提供四种内置主题，支持暗黑模式自动适配：

```swift
view.configureTheme(.info)     // 信息主题（灰色）
view.configureTheme(.success)  // 成功主题（绿色）
view.configureTheme(.warning)  // 警告主题（黄色）
view.configureTheme(.error)    // 错误主题（红色）
```

### 图标样式

```swift
view.configureTheme(.warning, iconStyle: .default) // 默认图标
view.configureTheme(.warning, iconStyle: .light)   // 浅色图标
view.configureTheme(.warning, iconStyle: .subtle)  // 柔和图标
```

### 自定义主题

```swift
// 使用自定义颜色
let iconText = "🎉"
view.configureTheme(
    backgroundColor: UIColor.purple,
    foregroundColor: UIColor.white,
    iconImage: nil,
    iconText: iconText
)
```

### 内置布局

```swift
// MessageView：全宽标准消息视图
let view = MessageView.viewFromNib(layout: .messageView)

// CardView：圆角卡片样式
let view = MessageView.viewFromNib(layout: .cardView)

// TabView：一端附着的卡片样式
let view = MessageView.viewFromNib(layout: .tabView)

// StatusLine：20pt 高的状态栏覆盖层
let view = MessageView.viewFromNib(layout: .statusLine)

// CenteredView：垂直居中的卡片样式
let view = MessageView.viewFromNib(layout: .centeredView)
```

---

## 消息队列管理

### 自动队列

SwiftMessages 自动维护消息队列，一次只显示一条消息。实现了 `Identifiable` 协议的视图（如 `MessageView`）会自动去重。

```swift
// 设置消息间的暂停时间（默认 0.5 秒）
SwiftMessages.pauseBetweenMessages = 1.0

// 可以多次调用 show()，消息会按顺序显示
SwiftMessages.show(view: message1)
SwiftMessages.show(view: message2)
SwiftMessages.show(view: message3)
```

### 隐藏消息

```swift
// 隐藏当前消息
SwiftMessages.hide()

// 隐藏当前消息并清空队列
SwiftMessages.hideAll()

// 隐藏指定 ID 的消息
SwiftMessages.hide(id: "someId")

// 计数隐藏（当 show 和 hideCounted 调用次数相等时才隐藏）
SwiftMessages.hideCounted(id: "someId")
```

### 获取消息

```swift
// 获取当前正在显示或隐藏的消息
if let view = SwiftMessages.current(id: "someId") as MessageView? {
    // 更新消息内容
}

// 获取队列中等待显示的消息
if let view = SwiftMessages.queued(id: "someId") as MessageView? {
    // 更新消息内容
}

// 获取当前或队列中的消息
if let view = SwiftMessages.currentOrQueued(id: "someId") as MessageView? {
    // 更新消息内容
}
```

### 多个实例

可以创建多个 SwiftMessages 实例同时显示多条消息：

```swift
class MyViewController: UIViewController {
    // 保留实例
    let topMessages = SwiftMessages()
    let bottomMessages = SwiftMessages()
    
    func showMessages() {
        // 在顶部显示一条消息
        SwiftMessages.show(view: topView)
        
        // 在底部同时显示另一条消息
        var config = SwiftMessages.Config()
        config.presentationStyle = .bottom
        bottomMessages.show(config: config, view: bottomView)
    }
}
```

---

## 内容配置

### MessageView 的内容元素

MessageView 提供以下可选的 IBOutlet 属性：

- `titleLabel: UILabel?` - 标题
- `bodyLabel: UILabel?` - 正文
- `iconImageView: UIImageView?` - 图标图片
- `iconLabel: UILabel?` - 图标文字（如 emoji）
- `button: UIButton?` - 操作按钮

### 配置内容的便捷方法

```swift
// 只设置正文
view.configureContent(body: "这是消息正文")

// 设置标题和正文
view.configureContent(title: "标题", body: "正文内容")

// 设置标题、正文和图片图标
view.configureContent(title: "标题", body: "正文", iconImage: myImage)

// 设置标题、正文和文字图标（emoji）
view.configureContent(title: "标题", body: "正文", iconText: "🎉")

// 完整配置
view.configureContent(
    title: "标题",
    body: "正文内容",
    iconImage: nil,
    iconText: "🎉",
    buttonImage: nil,
    buttonTitle: "关闭",
    buttonTapHandler: { _ in SwiftMessages.hide() }
)
```

### 隐藏元素

```swift
// 隐藏不需要的元素
view.titleLabel?.isHidden = true
view.button?.isHidden = true
view.iconImageView?.isHidden = true
```

### 按钮点击处理

```swift
// 按钮点击时隐藏消息
view.buttonTapHandler = { _ in 
    SwiftMessages.hide() 
}

// 消息视图点击时隐藏
view.tapHandler = { _ in 
    SwiftMessages.hide() 
}
```

---

## 键盘避让

使用 `KeyboardTrackingView` 让消息视图自动避开键盘：

```swift
var config = SwiftMessages.Config()
config.keyboardTrackingView = KeyboardTrackingView()
SwiftMessages.show(config: config, view: view)
```

`KeyboardTrackingView` 也可以独立使用：

1. 将 `KeyboardTrackingView` 固定到屏幕底部、左右边缘
2. 将需要避开键盘的内容底部固定到 `KeyboardTrackingView` 顶部
3. 使用相等约束严格跟随键盘，或使用不等约束仅在键盘靠近时移动

---

## 无障碍支持

SwiftMessages 提供出色的 VoiceOver 支持：

### 基本支持

- 消息显示时，标题和正文会自动组合为单个播报
- 如果使用 `dimMode`，背景遮罩下的元素不可聚焦
- 遮罩本身可聚焦并读出"关闭"（可自定义）

### 自定义无障碍文本

```swift
// 为消息添加前缀以提供更多上下文
view.accessibilityPrefix = "警告"
// VoiceOver 会播报："警告, 标题文字, 正文文字"

// 自定义遮罩的无障碍标签
config.dimModeAccessibilityLabel = "点击关闭"
```

### 实现 AccessibleMessage 协议

对于自定义视图，实现 `AccessibleMessage` 协议以提供完整的无障碍支持。

---

## 自定义视图

### 方式一：修改 NIB 文件

1. 将内置的 nib 文件拖拽到项目中
2. 修改布局和样式
3. SwiftMessages 会优先加载项目中的 nib 文件

```swift
// 仍使用相同的代码加载，但会使用你修改后的版本
let view = MessageView.viewFromNib(layout: .cardView)
```

### 方式二：子类化 MessageView

```swift
class MyCustomMessageView: MessageView {
    @IBOutlet weak var customButton: UIButton!
    
    // 添加自定义功能
    func customConfiguration() {
        // 自定义逻辑
    }
}

// 加载自定义视图
let view: MyCustomMessageView = try! SwiftMessages.viewFromNib(named: "MyCustomMessageView")
```

### 方式三：使用任意 UIView

```swift
// SwiftMessages 可以显示任何 UIView
let customView = MyCustomView()
SwiftMessages.show(view: customView)
```

---

## 视图控制器展示

`SwiftMessagesSegue` 可以将视图控制器作为消息展示。

### Interface Builder 方式

1. Control-拖拽创建 segue
2. 在 segue 类型中选择 "swift messages"

### 代码方式

```swift
// 创建目标视图控制器
let destinationVC = MyViewController()

// 创建并配置 segue
let segue = SwiftMessagesSegue(identifier: nil, source: self, destination: destinationVC)
segue.configure(layout: .bottomCard)
segue.dimMode = .blur(style: .dark, alpha: 0.9, interactive: true)
segue.messageView.configureDropShadow()

// 执行 segue
segue.perform()

// 关闭
dismiss(animated: true, completion: nil)
```

### 子类化 SwiftMessagesSegue（推荐）

```swift
class VeryNiceSegue: SwiftMessagesSegue {
    override init(identifier: String?, source: UIViewController, destination: UIViewController) {
        super.init(identifier: identifier, source: source, destination: destination)
        configure(layout: .bottomCard)
        dimMode = .blur(style: .dark, alpha: 0.9, interactive: true)
        messageView.configureNoDropShadow()
    }
}
```

子类会自动出现在 Interface Builder 的 segue 类型列表中。

### Segue 配置选项

```swift
// 布局快捷配置
segue.configure(layout: .bottomCard)

// 禁用交互式关闭
segue.interactiveHide = false

// 背景遮罩
segue.dimMode = .gray(interactive: true)

// 展示样式
segue.presentationStyle = .bottom

// 调整边距
segue.messageView.layoutMarginAdditions = UIEdgeInsets(top: 20, left: 20, bottom: 20, right: 20)

// 圆角
segue.containerView.cornerRadius = 20

// 键盘避让
segue.keyboardTrackingView = KeyboardTrackingView()
```

### 控制器尺寸

推荐使用以下方式之一指定视图控制器尺寸：

1. 在视图控制器中添加充分的宽高约束
2. 设置 `preferredContentSize` 属性
3. 为 `segue.messageView.backgroundView` 添加显式宽高约束

---

## 事件监听

使用事件监听器响应消息的显示和隐藏：

```swift
var config = SwiftMessages.Config()
config.eventListeners.append { event in
    switch event {
    case .willShow(let view):
        print("即将显示")
    case .didShow(let view):
        print("已经显示")
    case .willHide(let view):
        print("即将隐藏")
    case .didHide(let view):
        print("已经隐藏")
    }
}
```

---

## 高级自定义

### 自定义动画器

实现 `Animator` 协议创建自定义动画：

```swift
class MyCustomAnimator: Animator {
    // 实现协议方法
}

var config = SwiftMessages.Config()
config.presentationStyle = .custom(animator: MyCustomAnimator())
```

### BackgroundViewable 协议

采用此协议的视图可以指定一个 `backgroundView`，SwiftMessages 会忽略此视图之外的触摸。

```swift
class MyView: UIView, BackgroundViewable {
    var backgroundView: UIView { 
        return myInnerView 
    }
}
```

### MarginAdjustable 协议

SwiftMessages 会自动管理采用此协议的视图的布局边距，确保在各种展示上下文中都有理想的间距。

### Identifiable 协议

实现此协议为消息提供唯一标识符，用于消息去重和查找：

```swift
class MyView: UIView, Identifiable {
    var id: String {
        return "my-unique-id"
    }
}
```

---

## 实用技巧

### 1. 快速显示不同类型的消息

创建便捷方法：

```swift
extension SwiftMessages {
    static func showSuccess(_ message: String) {
        let view = MessageView.viewFromNib(layout: .cardView)
        view.configureTheme(.success)
        view.configureContent(title: "成功", body: message)
        view.button?.isHidden = true
        SwiftMessages.show(view: view)
    }
    
    static func showError(_ message: String) {
        let view = MessageView.viewFromNib(layout: .cardView)
        view.configureTheme(.error)
        view.configureContent(title: "错误", body: message)
        view.button?.isHidden = true
        SwiftMessages.show(view: view)
    }
}

// 使用
SwiftMessages.showSuccess("操作成功！")
SwiftMessages.showError("操作失败，请重试。")
```

### 2. 带加载指示器的消息

```swift
var config = SwiftMessages.Config()
config.duration = .indefinite(delay: 1, minimum: 1)
// 1秒后显示，如果显示则至少显示1秒

// 开始操作
SwiftMessages.show(config: config, view: loadingView)

// 操作完成后隐藏
SwiftMessages.hide(id: loadingView.id)
```

### 3. 防抖动显示

```swift
// 使用计数隐藏避免闪烁
SwiftMessages.show(view: progressView)  // 第1次调用
SwiftMessages.show(view: progressView)  // 第2次调用（被去重）

SwiftMessages.hideCounted(id: progressView.id)  // count 减1
SwiftMessages.hideCounted(id: progressView.id)  // count 减至0，真正隐藏
```

### 4. 响应式圆角

使用 `CornerRoundingView` 实现动态圆角：

```swift
if let cornerView = view.backgroundView as? CornerRoundingView {
    cornerView.cornerRadius = 15
    cornerView.roundsLeadingCorners = true  // 只圆化前缘角
}
```

---

## 注意事项

1. **iOS 13+ 状态栏**：从 iOS 13 开始，窗口无法覆盖状态栏。如需覆盖效果，使用 `config.prefersStatusBarHidden = true` 隐藏状态栏。

2. **暗黑模式**：内置主题自动支持暗黑模式。如果应用不支持暗黑模式，可以设置：
   ```swift
   if #available(iOS 13, *) {
       config.overrideUserInterfaceStyle = .light
   }
   ```

3. **App Extension**：如果在 App Extension 中使用，请使用 CocoaPods 的 `AppExtension` subspec：
   ```ruby
   pod 'SwiftMessages/AppExtension'
   ```

4. **线程安全**：虽然 SwiftMessages 内部做了线程处理，但推荐从主线程调用或使用 `viewProvider` 变体。

5. **内存管理**：SwiftMessages 实例需要被保留。使用静态 API 时会使用共享实例，创建自定义实例时需要作为属性保留。

---

## 常见问题

### Q: 如何让消息永久显示直到用户手动关闭？

```swift
var config = SwiftMessages.Config()
config.duration = .forever
SwiftMessages.show(config: config, view: view)
```

### Q: 如何禁用滑动关闭手势？

```swift
var config = SwiftMessages.Config()
config.interactiveHide = false
SwiftMessages.show(config: config, view: view)
```

### Q: 如何同时显示多条消息？

创建多个 SwiftMessages 实例：

```swift
let instance1 = SwiftMessages()
let instance2 = SwiftMessages()

instance1.show(view: view1)
instance2.show(view: view2)
```

### Q: 如何在显示消息时禁止用户交互其他界面元素？

使用模态遮罩：

```swift
var config = SwiftMessages.Config()
config.dimMode = .gray(interactive: false)  // 不可点击关闭
config.interactiveHide = false  // 禁用滑动关闭
config.duration = .forever  // 不自动关闭
```

### Q: 如何加载自定义 nib 文件？

```swift
// 按名称加载
let view: MessageView = try! SwiftMessages.viewFromNib(named: "MyCustomNib")

// 自动根据类名加载（需要 nib 文件与类同名）
let view: MyCustomView = try! SwiftMessages.viewFromNib()
```

---

## 完整示例

### 示例 1：简单的成功提示

```swift
import SwiftMessages

let view = MessageView.viewFromNib(layout: .cardView)
view.configureTheme(.success)
view.configureContent(title: "成功", body: "您的更改已保存！")
view.button?.isHidden = true

var config = SwiftMessages.Config()
config.presentationStyle = .top
config.duration = .seconds(seconds: 2)
config.dimMode = .none

SwiftMessages.show(config: config, view: view)
```

### 示例 2：底部卡片样式的警告

```swift
let view = MessageView.viewFromNib(layout: .cardView)
view.configureTheme(.warning)
view.configureContent(
    title: "网络连接不稳定",
    body: "请检查您的网络设置",
    iconText: "⚠️"
)
view.layoutMarginAdditions = UIEdgeInsets(top: 0, left: 20, bottom: 20, right: 20)

var config = SwiftMessages.Config()
config.presentationStyle = .bottom
config.duration = .forever
config.dimMode = .color(color: UIColor.black.withAlphaComponent(0.5), interactive: true)
config.interactiveHide = true

view.buttonTapHandler = { _ in
    SwiftMessages.hide()
}

SwiftMessages.show(config: config, view: view)
```

### 示例 3：中心对话框

```swift
let view = MessageView.viewFromNib(layout: .centeredView)
view.configureTheme(.info)
view.configureContent(
    title: "更新可用",
    body: "发现新版本，是否立即更新？",
    iconText: "📱"
)
view.configureDropShadow()

var config = SwiftMessages.Config()
config.presentationStyle = .center
config.duration = .forever
config.dimMode = .blur(style: .dark, alpha: 0.9, interactive: true)

view.buttonTapHandler = { _ in
    SwiftMessages.hide()
    // 执行更新操作
}

SwiftMessages.show(config: config, view: view)
```

### 示例 4：状态栏样式

```swift
let view = MessageView.viewFromNib(layout: .statusLine)
view.configureTheme(.warning)
view.configureContent(body: "无网络连接")

var config = SwiftMessages.Config()
config.presentationContext = .window(windowLevel: .statusBar)
config.duration = .forever
config.prefersStatusBarHidden = true

SwiftMessages.show(config: config, view: view)
```

---

## 总结

SwiftMessages 是一个功能强大、高度灵活的消息提示库，提供了：

- 🎨 丰富的内置主题和布局
- ⚙️ 灵活的配置选项
- 🎭 完整的自定义能力
- ♿ 优秀的无障碍支持
- 📱 现代化的 iOS 特性支持

无论是简单的 toast 提示还是复杂的模态对话框，SwiftMessages 都能轻松胜任。通过合理使用内置功能和适度自定义，您可以创建出既美观又实用的用户界面。

---

## 相关资源

- **GitHub 仓库**: [SwiftKickMobile/SwiftMessages](https://github.com/SwiftKickMobile/SwiftMessages)
- **视图控制器文档**: 查看 `ViewControllers.md` 了解 SwiftMessagesSegue 详细用法
- **Demo 应用**: 项目中的 Demo 展示了各种配置选项的效果
- **在线演示**: [Appetize.io](http://goo.gl/KXw4nD)

---

## 许可证

SwiftMessages 使用 MIT 许可证。详见 LICENSE.md 文件。

## 关于

SwiftMessages 由 SwiftKick Mobile 开发和维护。

