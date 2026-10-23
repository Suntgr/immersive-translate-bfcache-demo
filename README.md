# 沉浸式翻译 BFCache 演示 | Immersive Translate BFCache Demo

## 中文说明

这是一个最小化的 BFCache（Back-Forward Cache）复现演示，用于报告沉浸式翻译扩展的问题。

### 问题描述

启用沉浸式翻译扩展后，使用 `history.back()` 返回页面时无法命中 BFCache，导致页面重新加载。

### 复现步骤

1. 启动本地静态服务器：

```bash
python3 -m http.server 8765
```

2. 在浏览器中打开：`http://127.0.0.1:8765/a.html`

3. **关闭 DevTools**（DevTools 会影响 BFCache）

4. 点击「Go to B」链接，进入 B 页面

5. 点击 `history.back()` 按钮返回 A 页面

6. 对比观察：
   - **无扩展/禁用扩展**：`persisted: true`（绿色），API calls 不增加
   - **启用沉浸式翻译**：`persisted: false`（红色），API calls 递增

### 预期行为

启用扩展后，返回时应该 `persisted === true`，页面从 BFCache 恢复，不应重新请求 API。

### 实际行为

启用扩展后稳定显示 `persisted === false`，页面完全重新加载，API 请求次数递增。

---

## English Description

A minimal BFCache (Back-Forward Cache) reproduction demo for reporting an issue with the Immersive Translate extension.

### Issue

When the Immersive Translate extension is enabled, navigating back via `history.back()` fails to hit BFCache, causing a full page reload.

### Reproduction Steps

1. Start a local static server:

```bash
python3 -m http.server 8765
```

2. Open in browser: `http://127.0.0.1:8765/a.html`

3. **Close DevTools** (DevTools affects BFCache)

4. Click the "Go to B" link to navigate to page B

5. Click the `history.back()` button to return to page A

6. Compare behavior:
   - **Without extension / Extension disabled**: `persisted: true` (green), API calls count unchanged
   - **With Immersive Translate enabled**: `persisted: false` (red), API calls count increments

### Expected Behavior

With the extension enabled, navigating back should show `persisted === true`, restoring the page from BFCache without re-fetching the API.

### Actual Behavior

With the extension enabled, consistently shows `persisted === false`, causing a full page reload and incrementing the API request count.

---

## Files

- `a.html` - Demo page A (shows BFCache status and API call count)
- `b.html` - Demo page B (with back button)
- `data.json` - Sample API response
- `ISSUE.md` - Detailed bug report (Chinese)

## Links

- Extension: [Immersive Translate](https://chromewebstore.google.com/detail/bpoadfkcbjbfhfodiogcnhhhpibjhbnh)
- BFCache Info: [web.dev/bfcache](https://web.dev/articles/bfcache)
