# Pi 版本兼容性与影响分析

> 记录上游 Pi 更新对本项目的直接影响和可执行兼容策略。

## 当前基线

- Pi：`0.80.10`
- pi-autoname：使用 Pi Extension（扩展）和 Pi 宿主提供的 peer dependency（对等依赖），不安装额外运行或开发依赖。

## 直接采用的 Pi 0.80.x 能力

| Pi 能力 | 项目用法 | 收益 |
|---|---|---|
| `agent_settled` | 在所有自动重试、压缩重试和后续消息结束后，后台考虑自动命名 | 避免对未稳定上下文过早命名 |
| `session_info_changed` | 立即记录 `/name` 或 UI 改名 | 不再等下一轮代理事件才能识别手工名称 |
| `session_shutdown` | 取消未完成的命名请求 | 防止切换、重载或退出后旧请求迟到写入 |
| `ModelRegistry` | 从当前运行时模型目录解析配置的 `provider/modelId` | 兼容 Pi 动态 provider（提供商）与模型目录 |

## 模型调用兼容性

Pi `0.80` 将旧的全局 `complete()` / `getModel()` API（接口）从 `@earendil-works/pi-ai` 根入口移至 `@earendil-works/pi-ai/compat`。本项目显式使用 compat（兼容）入口：

- `complete()` 继续执行独立命名请求；
- 模型解析只使用 `ctx.modelRegistry.find()`，不再回退旧的静态 `getModel()`；
- `ctx.modelRegistry.getApiKeyAndHeaders()` 保留为 Extension（扩展）兼容门面。

这是当前 Pi 官方 Extension（扩展）示例仍使用的路径；未来若 Pi 将 `ModelRegistry` 替换为公开的 `ModelRuntime`（模型运行时）上下文接口，再做单独迁移。

## 性能与生命周期边界

- 自动命名在 `agent_settled` 后以 best-effort（尽力而为）后台任务运行，不阻塞主代理收口。
- 每次命名拥有 30 秒共享预算；单个模型最多 12 秒，回退链不会按模型数线性叠加等待。
- `session_shutdown` 和新的命名请求都会 abort（中止）旧请求。
- 当前标题会一并提供给模型；标题仍准确时保持不变，避免会话选择器抖动。

## 保留的兼容契约

- `pi.setSessionName()` / `pi.getSessionName()` 管理显示名称。
- `pi.appendEntry("pi-autoname-state", ...)` 持久化扩展 marker（标记）；marker 不进入 LLM（大语言模型）上下文。
- `respectManualName: false` 时，手工 `/name` 获得一个冷却窗口；`true` 时保持到用户显式运行 `/autoname`。
- AI 调用失败后仍可使用本地、脱敏的降级标题。

## 版本记录

| 日期 | Pi 基线 | 变更 |
|---|---|---|
| 2026-07-18 | `0.80.10` | 对齐新生命周期、模型 compat（兼容）入口和模型运行时目录 |
| 2026-06-09 | 发布版 | 初版：确认 skill prompt（技能提示词）间距修复对本扩展低影响 |
