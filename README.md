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

## DSH 安装

本仓库同时提供 DeepSeek Harness（DSH）组合包适配：根级 [`package.json`](package.json) 声明 DSH 组合包（`dsh.bundle.patch`），[`cordis.patch.yml`](cordis.patch.yml) 把一个隔离的 skill 提供方和 serve-avd MCP 挂进当前 profile。8 个 Skill 与 MCP 命令由 Codex / ZCode / DSH 三方共享，功能一致。

在 DSH 侧边栏 **插件** 页（或 `plugin_manager` 工具，`action: install_bundle`）安装：

1. 添加组合包，目标填本仓库根目录的绝对路径（例如 `/path/to/build-android-app`）；也可用 Git 地址 `github:QQQZZZY/build-android-app`。
2. 安装会写入当前 profile 的 `dsh.profile.bundles`，配置热重载后即可生效；无需手工编辑 profile 的 `cordis.patch.yml`。
3. 生效后 8 个 Skill 进入 Skill 目录，MCP 工具以 `mcp__android__<tool>` 出现。试用提示词：「构建当前 Android 应用并在模拟器上运行，检查 UI」。

DSH 适配只新增根级 `package.json` 与 `cordis.patch.yml`，未改动 `plugin.json`、`.codex-plugin/`、`.mcp.json`、`mcp.json`、`agents/`、`skills/`、`.agents/plugins/marketplace.json` 与 `marketplace.json`，Codex 与 ZCode 的插件配置保持不变。Skill 通过独立的 filesystem 提供方挂载（`includeDefaultRoots: false`），不会与 DSH 默认的项目/用户 skill 根重复扫描。

运行依赖与 Codex 侧相同：Java、Android SDK/ADB、模拟器，以及 Node.js 18.17+ 来启动 serve-avd。
