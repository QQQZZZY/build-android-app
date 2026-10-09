# 第三方来源与许可

## Google Android Skills

下列文件从 Google LLC 的官方仓库 [`android/skills`](https://github.com/android/skills) 原样打包，来源提交为 [`42dc2270e96032bd860bb94511e440aa00a43125`](https://github.com/android/skills/tree/42dc2270e96032bd860bb94511e440aa00a43125)：

| 插件目录 | 上游目录 |
|---|---|
| `skills/android-cli/` | `devtools/android-cli/` |
| `skills/edge-to-edge/` | `system/edge-to-edge/` |
| `skills/adaptive/` | `jetpack-compose/adaptive/` |
| `skills/testing-setup/` | `testing/testing-setup/` |
| `skills/navigation-3/` | `navigation/navigation-3/` |

这些 Skills 的支持文件也按原相对路径保留；`android-cli` 增加了一段仅针对本插件的说明，避免普通构建请求自动安装主机工具。其余上游内容未改。仓库采用 Apache License 2.0；完整文本见本目录的 `LICENSE.txt`。

## OpenAI Test Android Apps

从 OpenAI 官方 [`openai/plugins`](https://github.com/openai/plugins) 的
[`0722921d5542fc593105c27bd52630babd8b8c2a`](https://github.com/openai/plugins/tree/0722921d5542fc593105c27bd52630babd8b8c2a/plugins/test-android-apps)
提交集成 `Test Android Apps 0.1.2` 的两个完整 Skill 目录：

| 插件目录 | 上游目录 | 许可依据 |
|---|---|---|
| `skills/android-emulator-qa/` | `plugins/test-android-apps/skills/android-emulator-qa/` | 该目录自带 Apache-2.0 `LICENSE.txt`，原文保留 |
| `skills/android-performance/` | `plugins/test-android-apps/skills/android-performance/` | 上游插件清单声明 MIT；清单保存在 `licenses/test-android-apps-plugin.json`，标准 MIT 文本附在该 Skill 的 `LICENSE.txt` |

脚本原样保留；针对本包调整 Skill 指引：复用已选设备，优先用 serve-avd
交互；不清空 Logcat；性能录制用本次任务的文件路径并只停止自己启动的
录制进程；QA 声明 Python 3.10+ 要求，不要求全局 Gradle，并补充多屏截图的显示屏 ID 后备流程。两个 Skill 及其
agent 元数据包含本包适配，发布者仍是 Build Android App contributors。
上游 `plugin.lock.json` 只记录旧版单个 Skill，未用它充当 0.1.2 的完整版本证明。

## serve-avd

插件不复制 `serve-avd` 源码；MCP 配置从 npm 获取并固定使用 `serve-avd@0.1.3`。上游为 [`hsandhu/serve-avd`](https://github.com/hsandhu/serve-avd)，许可证为 Apache-2.0，Node.js 要求 `>=18.17`。该 MCP 服务在本机通过 ADB 控制已连接的模拟器/设备。

## 未打包的官方专项 Skill

没有打包官方 `r8-analyzer`：在上述官方提交中，它的定量分析流程引用 `scripts/convert_pb_to_json.py` 与 `scripts/analyze.py`，但这两个脚本不在该 Skill 目录或仓库树中。为了避免发布后指向缺失文件的 Skill，本版本先不包含它。Google `android-profiler` 未另行打包，本版本复用 OpenAI `android-performance` 提供聚焦的 CPU、帧、启动和内存采集工作流。
