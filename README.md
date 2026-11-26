# Lock Screen Splash Texts

Minecraft 风格的 Windows 11 锁屏随机标语 Windhawk Mod。

## 使用

1. 打开 Windhawk 的 Mod 编辑器。
2. 将 `lock-screen-splashes.wh.cpp` 的全部内容粘贴到编辑器，编译并启用。
3. 在设置中编辑短句、字体、备用字体、字号、颜色、旋转角度和动画。
4. 按 `Win+L` 查看效果。

标语显示在 `StackPanel#TimeAndDatePanel` 内，默认位于日期下方。默认 Segoe UI、24 号、黄色 `#FFFF00`、旋转 −20°，缩放范围为 90%～110%。右下方的黑色阴影随文字一起旋转和缩放。偏移量可用于调整位置。动画关闭时缩放为 100%。

备用字体可留空，或填入按优先级排列的字体名称，用逗号分隔，例如 `Microsoft YaHei UI, Segoe UI Emoji`。XAML 会在主字体无法渲染时尝试备用字体。

首次挂载和锁屏重新显示时会随机选句。有多个不同短句时避免连续重复。锁屏显示时直接读取并应用已保存的字号、字体等设置。外部短句文件由后台线程读取，读完后更新标语。

## 外部短句文件

将 UTF-8 编码的 `splashes.txt` 的本地绝对路径填入设置，一行一条，支持 BOM、中文及 CRLF。空白行会被忽略。文件内容优先于设置列表；读取失败或无有效内容时退回列表，两者均为空时使用 `Removed Herobrine!`。

文件须允许 LockApp 读取，最大 1 MiB，最多读取 4096 条，每条最多 512 个 UTF-16 代码单元。Mod 在启动、保存设置及每次锁屏显示时由后台检查文件。文件标识、大小及时间戳没有变化时跳过内容读取。需要读取时，解析结果与缓存列表不同时才会更新短句。Windhawk 日志会记录打开失败的错误码或成功载入的条数。

## 验证状态与限制

源码已用 Windhawk 1.7.3 的 x64 工具链编译并链接为 DLL。锁屏显示、内部 XAML 容器名称、布局以及与其他锁屏样式 Mod 的兼容性仍需实机确认。长句或较大旋转可能被父容器裁剪，可缩短短句或减小字号。

Mod 通过 `CoreWindow.VisibilityChanged` 响应锁屏显示。若窗口已显示而 XAML 容器尚未就绪，Mod 只在挂载阶段短暂重试，最多约 10 秒。动画由 XAML Storyboard 执行。启动和卸载各自用一次临时 Hook 把工作送到 UI 线程。禁用或卸载前，Mod 会注销事件、取消重试、移除标语并停止动画。新版挂载时也会清除旧版遗留的标语。实现目前针对进程中的首个 CoreWindow，窗口或 UI 线程被重建时需重新加载 Mod。LockApp 卡住时，同步 UI 调用可能延迟卸载。

建议实机检查：首次启用后 Win+L；连续解锁和锁屏；中文及外部文件；无效颜色和空列表回退；关闭动画、交换缩放上下限；锁屏显示期间修改设置及禁用 Mod，确认新增文字移除且原时间日期保留。

## 参考与许可

- [Lock Screen Clock Styler](https://github.com/ramensoftware/windhawk-mods/blob/main/mods/lock-screen-clock-styler.wh.cpp)
- [Lock Screen Date and Time Format](https://github.com/ramensoftware/windhawk-mods/blob/main/mods/lockscreen-datetime-format.wh.cpp)

参考这些项目的 CoreWindow、UI 线程 Hook 和视觉树访问方式。源码以 GPL-3.0-or-later 发布。
