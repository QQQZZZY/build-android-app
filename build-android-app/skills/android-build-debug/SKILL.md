---
name: android-build-debug
description: 检查 Android 项目的开发环境，按项目现有 Gradle 配置构建应用，在指定模拟器安装并启动 APK，查看截图、交互和 Logcat，再验证修改。用户请求 Android 构建、运行、模拟器 UI 调试或构建后复验时使用。
---

# Android 构建与调试

按用户请求完成必要步骤；只构建、只查环境或只看 UI 时，不额外扩展成完整流程。

## 约束

- 先阅读项目内的 `AGENTS.md` 和 Gradle 配置，沿用当前模块、变体、依赖和任务命名。
- 优先使用仓库自带的 Gradle Wrapper：macOS/Linux 用 `./gradlew`，Windows 用 `gradlew.bat`。不要为了让构建通过而升级 AGP、Gradle、JDK、compileSdk 或添加插件/依赖，除非用户要求。
- 官方专项 Skill 若要求升级 targetSdk、迁移导航架构或新增测试依赖，而这些不在当前请求范围内，先说明变更范围并等待用户决定，不要顺手扩大项目改动。
- Android CLI 是可选工具。缺少它时，核心构建流程仍可用 Gradle Wrapper 与 ADB；不要为了本次检查静默安装 CLI、SDK 或模拟器。若请求明确要求安装，再按 `android-cli` 官方 Skill 的步骤执行。
- 用户指定设备时使用该设备。否则优先选唯一已启动的模拟器；存在多个模拟器或只有真机时，先确认目标，不要默认操作列表中的第一个设备。
- 不卸载应用、不清除应用数据、不清空 Logcat，也不修改持久设备设置。安装失败或需要清数据时先报告原因。
- `serve-avd` 的 `event_log` 记录的是代理交互事件；Android 系统与应用日志用 ADB Logcat 诊断。

## 流程

### 1. 识别项目与环境

查看 `settings.gradle[.kts]`、根和应用模块的 `build.gradle[.kts]`、`gradle/wrapper/gradle-wrapper.properties`、`gradle/libs.versions.toml` 与应用 Manifest。先确定实际应用模块和可用变体，不假设模块一定叫 `app`。

检查当前环境中是否有 Java、Android SDK、ADB、模拟器、Node.js/npm/npx，以及 Android CLI（若存在）。在 Bash/Zsh 中可用 `command -v`，在 PowerShell 中可用 `Get-Command`。记录 `java -version`、`adb version`、`adb devices -l`；若 `android` 命令存在，可用 `android info` 查看 SDK 与设备信息；若 `emulator` 命令存在，可用 `emulator -list-avds` 查看 AVD。只报告完成请求实际需要的缺项。

### 2. 构建

从现有 Gradle 配置与任务名确认目标，再运行最小相关任务。例如，应用模块确认为 `app` 且用户要 Debug APK 时才用 `./gradlew :app:assembleDebug`。按用户请求运行已有的对应单元测试或设备测试；不要为一次普通构建新建测试框架。

只有 Gradle 命令成功退出且对应 APK 文件存在，才报告构建成功。失败时先用输出定位具体的配置、依赖、JDK、SDK 或编译错误，不要猜版本或做无关升级。

### 3. 选择设备并安装运行

若 `android` MCP 已连接，先调用其 `list_devices` 查看设备；否则使用 `adb devices -l`。确认目标后，优先调用 serve-avd 的 `install_apk` 和 `launch_app` 工具。只在 MCP 不可用时使用同一设备序列号的 `adb -s <serial> install -r <apk>`，并用 ADB 启动应用。`-r` 保留已有应用数据；遇到签名或版本降级错误时停止并说明，不卸载绕过。

若用户要求运行但没有已启动模拟器，先用 `android emulator list` 或 `emulator -list-avds` 查看已配置 AVD。只有一个合适 AVD 时，Android CLI 可用则运行 `android emulator start <AVD>`；否则运行 `emulator -avd <AVD>` 并等待设备启动。若有多个可选 AVD，先询问目标；不要自动创建 AVD 或下载 SDK 镜像。

启动后先获取 `screenshot` 和 `ui_tree`。若没有对应 MCP 工具，使用 Android CLI 的屏幕/布局工具或 ADB 截图；先看当前画面再点击，优先用可访问性标签定位，坐标只在必要时使用。输入内容前确认焦点在正确控件。

多屏/折叠屏若遇到 `screencap produced no image`，按 Emulator QA 的显示屏选择步骤用 ADB 指定 display-id 截图，不循环重试同一个 MCP 调用。

默认 MCP 不带 `--serve`，连接本身不会启动预览或自动选择 AVD。用户需要实时预览时，确认模拟器在线，在可跟踪的长运行终端执行 `npx -y serve-avd@0.1.3 <serial>`，打开它实际输出的本地 URL，并检查真实画面。结束预览时停止这个终端及本次启动的服务；不要终止其他任务的服务或原有模拟器。不要无设备参数启动预览，上游可能自动启动第一个 AVD。

### 4. 诊断日志

先读取应用包名，再定位当前进程并尽量按 PID 查看 Logcat；没有 PID 时按包名、`AndroidRuntime` 和相关错误标签缩小范围。不要先清空日志。区分应用崩溃、系统服务、Gradle 构建错误和模拟器/MCP 连接错误。

### 5. 修改后的复验

如果本次修改了代码，重跑受影响的构建任务；用户要求运行时验收时，重新安装、启动并检查相关页面的截图或 UI 树。只重复能验证本次改动的检查。最终说明构建任务、APK 位置、目标设备、实际检查的页面/交互、日志结论，以及未能验证的部分。

## 官方专项 Skills

- UI 流程复现、树定位、截图与日志验收：使用 [Emulator QA](../android-emulator-qa/SKILL.md)，复用同一个目标设备。优先使用 serve-avd MCP；ADB 和上游树处理脚本作为后备。
- CPU 热点、启动耗时、卡顿、Compose Trace、内存或泄漏线索：使用 [Android Performance](../android-performance/SKILL.md)。普通构建不自动进入性能采集；内存计数不能单独证明泄漏。
- Compose 系统栏遮挡或输入法 Insets：使用 `edge-to-edge`。
- 手机、平板或折叠屏自适应：使用 `adaptive`，先确认项目满足该 Skill 的 Compose/Navigation 3 前置条件。
- 测试架构、UI/截图/E2E 测试搭建：仅在用户请求测试方案或测试基础设施时使用 `testing-setup`；普通构建只运行仓库已有的相关检查。
- Navigation 3：仅在用户请求导航实现或迁移时使用 `navigation-3`。
- Android CLI、SDK/AVD、官方 Android 文档搜索：使用 `android-cli`；不把安装 CLI 当作普通 Gradle 构建的前置条件。
