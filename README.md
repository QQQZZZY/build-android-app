# Build Android App 插件包

插件源码在 [`build-android-app/`](build-android-app/)，公开发布包为 [`build-android-app-0.1.1-public.zip`](build-android-app-0.1.1-public.zip)。

当前版本 `0.1.1` 包含 8 个 Skills：5 个 Google Android 官方 Skills、
OpenAI Test Android Apps 的 Emulator QA 和 Android Performance，以及构建调试主流程。
设备 MCP 仍为 `serve-avd@0.1.3`。

详细能力和运行依赖见 [插件说明](build-android-app/README.md)，许可和固定上游版本见 [第三方来源](build-android-app/THIRD_PARTY_NOTICES.md)。原创内容采用 Apache-2.0，打包的第三方内容遵循各自的 Apache-2.0 或 MIT 许可。

从 GitHub 安装 Marketplace：

```bash
codex plugin marketplace add QQQZZZY/build-android-app
```

然后在 Codex 的 Plugins 目录选择该源，安装 **Build Android App**。

此目录已包含 `.agents/plugins/marketplace.json`。进入该目录，在 Codex CLI 中将它加入本地 Marketplace：

```bash
codex plugin marketplace add .
```

然后重启 Codex Desktop，在 Plugins 目录中从这个本地源安装 **Build Android App**。安装后需要本机具备 Java、Android SDK/ADB、模拟器，以及 Node.js 18.17+ 来启动 serve-avd。

## ZCode 安装

本仓库同时提供 ZCode 插件适配：ZCode 清单在 [`build-android-app/.zcode-plugin/plugin.json`](build-android-app/.zcode-plugin/plugin.json)，本地市场目录为根级 [`marketplace.json`](marketplace.json)（市场名 `build-android-app`）。Skills 与 serve-avd MCP 配置由 Codex / ZCode 两端共享，功能一致。

在 ZCode Desktop 中安装：

1. 打开 **插件市场 → 添加 → 添加插件市场**，粘贴本仓库根目录的绝对路径（即 `marketplace.json` 所在目录）。
2. 在该市场（`build-android-app`）中选择 **Build Android App** 并安装。
3. 新建任务试用：例如「构建当前 Android 应用并在模拟器上运行，检查 UI」。

运行依赖与 Codex 侧相同：Java、Android SDK/ADB、模拟器，以及 Node.js 18.17+。
