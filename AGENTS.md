# Rooftop Station 引擎 fork 开发约定

本仓库是 `ZouAgTao/godot` 的 `audreborn-4.7` 分支，供 Rooftop Station 通过 libgodot 嵌入使用。
基线是社区 libgodot 4.7 分支的 `30aca496`，Swift 嵌入层和打包脚本位于兄弟仓库 `../SwiftGodotKit`。

## 范围与协作

- 本仓库负责引擎 C++/Objective-C++ 修改；`../SwiftGodotKit` 负责 Swift 嵌入层、帧循环、消息桥和打包；`../RooftopStation` 是 App 宿主。
- 开发前读取 `../SwiftGodotKit/AGENTS.md` 和 README 的当前构建流程。进入 App 联调前读取其 `AGENTS.md`。
- 中文交流，英文提交，正文解释原因。导入的历史只作参考，不自动执行旧待办。
- 开始任务、提交和推送前检查 Git 状态。只暂存本次任务的具体文件，保留其他会话工作。
- 按当前授权提交和推送；发布引擎 release 需要相应授权。不强推，不使用 `git pull -q`。
- 修改尽量局限于嵌入宿主确有需要的行为，保留兼容的 API 和现有 SwiftGodot 绑定。

## 引擎修改与验证

- 当前 fork 提供 `AudioServer.restart_output_driver()` 和 `stop_output_driver()`，实现位于 `servers/audio/audio_server.{cpp,h}`。
- 重建音频驱动需要重新读取音频会话格式；停止后不要再次 `finish()` 已停止的驱动，部分驱动的二次释放不安全。
- 不随意改变 AudioServer 的对象布局；新增状态优先参照现有补丁，检查 ABI 影响。
- 音频问题先记录驱动、播放实例和音量状态，再定位故障。模拟器结果不能替代 AVAudioSession 和线路变化的真机验证。
- App 的标准桌面 Godot 没有 fork 专用方法；宿主 GDScript 使用 `has_method` / 动态调用。
- 只改这些嵌入行为不需要换桌面编辑器或导出模板，App 只导出 `.pck` 资源包。

## 构建与发布

从 `../SwiftGodotKit` 执行：

```sh
scripts/build-ios-audreborn.sh
scripts/make-libgodot.xcframework . ../godot artifacts
```

脚本构建 arm64 真机以及 arm64/x86_64 模拟器的 iOS release 静态库。
工具在 `../.venv-godot/bin/scons`，日志在 `../build-logs`，产物在 `../SwiftGodotKit/artifacts`，均无需提交。

- 联调需宿主引用本地 SwiftGodotKit 包，并在解析包时设置 `LIBGODOT_LOCAL=1`；否则仍消费已发布引擎二进制。
- 交付前按改动运行必要检查，以及宿主 `tools/verify_ios.sh`；逐个确认构建切片和检查命令的退出码。
- 发布前先提交、推送引擎源码，再运行 `../SwiftGodotKit/scripts/publish-audreborn-release.sh`，使用全新的版本 tag。
- 把发布得到的版本和 checksum 写入 SwiftGodotKit 的 `Package.swift`，提交、推送后由 App 更新固定 revision。
- 不替换已有 release 的 zip。Swift 源码单独变化时无需重发引擎二进制。
