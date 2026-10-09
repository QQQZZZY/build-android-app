# Build Android App

一个本地优先的 Codex 插件，按项目现有 Gradle 配置构建 Android 应用，并通过 `serve-avd` MCP 在本机模拟器安装、运行、截图和交互。Android 官方 Skills 作为可追溯的上游快照随插件提供。

版本 `0.1.1` 已集成 OpenAI 官方 **Test Android Apps 0.1.2** 的两个 Skills
及全部配套脚本。共 8 个 Skills 和 1 个 MCP；无需再安装 Test Android Apps
才能使用本包中的 QA 和性能功能。

## 能力

- 环境预检：Java、Gradle Wrapper、Android SDK/ADB、模拟器、Node/npx 与可选 Android CLI。
- Gradle 构建；在请求范围内运行已有测试。
- APK 安装与启动、设备截图、UI 层级读取和点击/滑动/输入。
- 使用 ADB Logcat 定位应用启动与崩溃问题，并在修改后复验。
- OpenAI Emulator QA：复现交互流程，读取 UI 树、定位控件、截图与采集日志。
- OpenAI Android Performance：Simpleperf CPU 热点、Perfetto/Compose Trace、gfxinfo 帧统计、启动和 Java/native 内存分析线索。
- Google Android 官方 Skills：Android CLI、Edge-to-edge、Adaptive、Testing Setup、Navigation 3。

插件主流程不要求单独安装 Android CLI；Gradle Wrapper 和 ADB 是构建/设备流程的核心依赖。缺少设备或工具时，Skill 会报告缺项，不会静默安装 SDK 或清除应用数据。

## 安装到 Codex

可直接添加 GitHub Marketplace：

```bash
codex plugin marketplace add QQQZZZY/build-android-app
```

然后在 Codex 的 Plugins 目录选择该源并安装 **Build Android App**。本地源码安装步骤如下：

1. 将本插件目录放在 Marketplace 根目录下；根目录需有 `.agents/plugins/marketplace.json`，本发布源码的外层目录已提供该文件。
2. 进入包含该文件的 Marketplace 根目录，将它加入 Codex：

   ```bash
   codex plugin marketplace add .
   ```

3. 重启 Codex Desktop，在 Plugins 目录选择该本地源并安装 **Build Android App**。

也可使用 [`../build-android-app-0.1.1-public.zip`](../build-android-app-0.1.1-public.zip) 分享单个插件目录，再按 Codex 的本地 Marketplace 方式注册。

## 主机依赖

- Java 版本需满足项目当前 Gradle/AGP 要求。
- Android SDK 与 ADB；运行验收还需要已启动的 Android Emulator 或用户指定设备。
- Node.js `>=18.17` 与 npm/npx，用于启动固定版本 `serve-avd@0.1.3`。
- 可选：Google Android CLI。官方 Android CLI 可通过 `android skills add --agent=codex --project=.` 单独安装 Skills；本插件已经打包了所列快照。
- 可选：Python 3.10+，用于 OpenAI 的 ADB UI 树定位脚本；有 serve-avd 时可直接使用 MCP UI 工具。
- 可选：Bash 与 Simpleperf / Perfetto trace processor，用于性能报告；Java 泄漏分析可使用 Shark CLI。按所选采集类型检查依赖，无需为了普通构建安装这些工具。Windows 运行 `.sh` 报告脚本需要 Bash 环境。

Google 当前文档注明 Android CLI 的 `android emulator` 命令在 Windows 上暂不可用；Windows 可使用 Android Studio/SDK 自带模拟器与 ADB 流程。

首次连接 MCP 时，`npx` 会按需获取 npm 包。默认运行不带 `--serve` 的 MCP，避免连接时自动启动第一个 AVD。实时预览在选定在线设备后用 `npx -y serve-avd@0.1.3 <serial>` 单独启动，再打开实际输出的 URL。日志排查使用 ADB Logcat，`serve-avd` 的 `event_log` 只记录代理操作事件。

## 官方 Skills 的来源与更新

来源、完整相对路径、许可证和固定提交号见 [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)。Android Skills 使用标准 `SKILL.md` 格式；Google 的 Android CLI 文档支持将 Skills 安装给 `codex` agent。打包的文件是固定快照，不会随插件安装自动更新。更新插件时，从新的官方提交同步所列 Skill 目录及支持文件，并同时更新 notices 中的提交号。

OpenAI 的 `android-emulator-qa` 和 `android-performance` 是 Skills + 脚本，
不自带 MCP。`serve-avd` 负责设备交互，性能工具通过 ADB 与命令行采集。
采集内存快照不能单独证明泄漏；模拟器性能数据也不能替代真机性能验收。

如需单独安装/更新 Android Skills（而非随插件使用），可用：

```bash
android skills add --agent=codex --project=. edge-to-edge
android skills update --agent=codex --project=. edge-to-edge
```

## 配置

插件内 `mcp.json` 是 Agent Plugins 1.0.0 格式；`.mcp.json` 与 `.codex-plugin/plugin.json` 是 Codex 兼容配置，均调用：

```text
npx -y serve-avd@0.1.3 mcp
```

不要同时手动再注册同名的全局 `android` MCP，否则会出现重复工具连接。

## 0.1.1 验证情况

- 8 个 Skill 的元数据、内部路径、许可证和包配置已检查；4 个 OpenAI 脚本保持上游原文。UI 定位脚本已用 Python 3.12 验证目标坐标、未找到目标及真实 UI 树处理；性能报告脚本通过 Bash 语法和帮助参数检查。
- MCP 初始化、20 个工具发现及模拟器 UI 树读取通过。测试用的多屏模拟器会导致上游 MCP 截图失败；已实测指定 display-id 的 ADB 截图并查看真实画面。本版默认 MCP 不自动启动预览，预览按需绑定选定设备。
- 本次未构建应用、执行完整交互、采集真实 CPU/内存 Trace 或验收浏览器视频流。测试启动的 MCP 与模拟器已关闭。
