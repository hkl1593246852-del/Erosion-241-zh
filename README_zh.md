# Erosion 简体中文本地化

## 改了什么

| 文件 | 改动 |
|---|---|
| `Erosion/zh-Hans.lproj/Localizable.strings` | **新增**，164 条中文 |
| `Erosion/Info.plist` | 加 `CFBundleDevelopmentRegion`、`CFBundleLocalizations [en, zh-Hans]` |
| `Erosion.xcodeproj/project.pbxproj` | `knownRegions` 加 `zh-Hans` |
| `Erosion/UI/Lists/HeaderLabel.swift` | `Text(label)` → `Text(LocalizedStringKey(label))` |
| `Erosion/UI/Interactions/ButtonLabel.swift` | 同上 |
| `Erosion/UI/Interactions/ToolbarLabel.swift` | 同上 |
| `Erosion/UI/Interactions/PlainToggle.swift` | 同上 |
| `Erosion/UI/Alerts/CompactAlert.swift` | `title` / `text` 都包一层 |
| `Erosion/UI/Alerts/PlainAlert.swift` | 同上 |
| `Erosion/App/InfoSheet.swift` | `title` / `context` 都包一层 |
| `Erosion/Helpers/App/Alertinator.swift` | UIKit 弹窗改用 `NSLocalizedString` |

## 为什么不能只加 strings

SwiftUI 里 `Text("字面量")` 会自动查 `Localizable.strings`，但**自定义组件传进来的变量不会**：

```swift
// HeaderLabel 内部
Text(label)          // label 是 String 变量 → 直接显示，不查表
```

所以有两类要分开处理：

1. `Text("Logs")`、`Button("Respring")`、`.navigationTitle("Erosion")` —— 原生调用，建表就生效
2. `HeaderLabel("Logs")`、`PlainAlert(title: "…")` —— 走自定义组件，得在组件内部包一层 `LocalizedStringKey`

不处理第二类，界面上会一半中文一半英文。

`LocalizedStringKey` 对找不到的 key 返回原文，所以对动态内容（文件名、日志、数据值）安全，不会显示成 key 本身。

## 语言怎么生效

Erosion **没有 App 内语言选择器**，跟随系统：

1. iPhone 设置 → 通用 → 语言与地区 → 首选语言 → 简体中文
2. 杀掉 Erosion 进程重开（切后台不算）

`CFBundleLocalizations` 声明后，简体中文才会出现在系统支持列表里。

## 编译

仓库已配好 GitHub Actions，push 或手动 dispatch 都会自动出包：

```
.github/workflows/build.yml
  runs-on: macos-latest
  → xcodebuild (走 ipabuild.sh)
  → ldid 签名
  → 上传 Erosion.ipa
```

本地编译（需要 macOS）：

```bash
chmod +x ipabuild.sh
./ipabuild.sh
```

产出 `Erosion.ipa`，bundle id `com.jbdotparty.Erosion`。

## 验证

```bash
unzip -l Erosion.ipa | grep lproj
# Payload/Erosion.app/zh-Hans.lproj/Localizable.strings

plutil -convert json -o - \
  Payload/Erosion.app/zh-Hans.lproj/Localizable.strings | jq length
# 164
```

编译后的 `.strings` 是二进制 plist（`bplist00`），这是 Xcode 构建时的正常转换，不是损坏。

## 翻译约定

| 保留原文 | 理由 |
|---|---|
| Gestalt / MobileGestalt | 系统框架名，日志里都是这个词 |
| PosterBoard | 进程名 |
| `.tendies` | 文件扩展名 |
| AOD / SRD / Plist / Inode | 技术缩写 |
| iPhone 16 Pro 等型号 | 设备标识，不能翻 |
| Erosion | 应用名 |

`Respring` 统一译「注销重启」。`Tweaks` 译「调整」，`Theming` 译「主题」，`Supervision` 译「监管」——都是越狱圈通行叫法。

## 未翻译的部分

- 设备型号列表（`iPhone 18 Pro Max`、`iPad Pro 11-inch (M4)` 等），属于标识符
- MobileGestalt 的键名（`ArtworkDeviceSubType`、`DeviceSupportsGenerativeModelSystems` 等），改了会找不到
- 日志输出、文件路径、Plist 数据值
