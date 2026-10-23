## Bug: history.back() 无法命中 BFCache，列表页被整页重载

### 环境
- 扩展：沉浸式翻译（Immersive Translate）**1.34.2**（Chrome Web Store id: `bpoadfkcbjbfhfodiogcnhhhpibjhbnh`）
- 浏览器：Chrome（复现时关闭 DevTools）
- 系统：macOS / Linux 均可复现

### 现象
从页面 A 整页跳到 B，再 `history.back()` 回到 A 时：

| | 无扩展 / 禁用扩展 | 启用沉浸式翻译 |
| --- | --- | --- |
| `pageshow.persisted` | `true`（命中 BFCache） | `false`（未命中） |
| 列表接口 / `fetch` | 不重复请求 | 再次请求 |
| 页面表现 | 状态保留，无闪烁重载 | 像整页刷新 |

同一浏览器、同一组静态页面，**只开关本扩展**即可切换 HIT / MISS。内容脚本会注入到 `http://127.0.0.1`（可见悬浮按钮）。

### 最小复现
1. 本地起静态服务（不要用 `file://`）：

```bash
# 解压附件或使用下面两个 HTML + data.json
python3 -m http.server 8765
```

2. 打开 `http://127.0.0.1:8765/a.html`
3. 关闭 DevTools
4. 点「Go to B」→ 再点 `history.back()`
5. 对照：无痕/禁用扩展 vs 启用扩展，看 A 页上的 `persisted` 与 API calls

演示页本身**没有**注册 `unload` / `beforeunload`，也没有下发 `Cache-Control: no-store`，对 BFCache 友好。

### 预期
启用扩展后，返回 A 仍应 `persisted === true`，不应重新挂载或重复打列表接口（与禁用扩展时一致）。

### 实际
启用扩展后稳定 `persisted === false`，API 计数递增。

### 可能原因（猜测，供排查）
扩展打包的 `content_main.js` 在运行时会：

1. 注册 `window` 的 **`beforeunload`** 与 **`pagehide`** 监听（并劫持 `history.pushState` / `replaceState`、监听 `popstate`）
2. 使用 **`chrome.runtime.connect` / 长连接**、以及页面侧的 **BroadcastChannel / SharedWorker / IndexedDB** 等

这些都会让页面更容易被排除出 Back-Forward Cache。建议在内容脚本里：

- 避免无条件注册 `beforeunload`（Chrome 对带该监听的页面进 BFCache 不友好；至少不要在未真正需要拦截离开时挂上）
- 在 `pagehide` 且 `event.persisted === true` 时主动断开 port / 释放会阻止缓存的资源
- 用 `chrome://bfcache` 或不带扩展的对照确认恢复路径

`content_guard.js` 会动态 `import('content_main.js')`；只要 main 跑起来，上述监听就会挂上。

### 附件
见本 issue 附带的 `a.html` / `b.html` / `data.json`（或 gist）。
