# tests/

pi-autoname 的测试目录。

## 文件说明

| 文件 | 职责 |
|---|---|
| `pi-autoname.test.ts` | `extensions/lib.ts` 纯函数测试：配置、脱敏、名称质量、尾部对话提取、marker 解析 |
| `extension-lifecycle.test.ts` | `extensions/controller.ts` 生命周期测试：首次/周期命名、手工名称保护、稳定标题、请求替换与关闭取消 |

## 运行

```bash
npm test
```

测试只使用 Node.js（JavaScript 运行时）内置的 `node:test`（测试 API）；Node.js 22.6 及以上可直接执行本目录的 TypeScript（类型脚本）测试。
