---
name: browser-cdp
description: "Use this skill when you need to control a Chrome browser via CDP (Chrome DevTools Protocol) to reuse existing login sessions. Covers: launching Chrome in debug mode, opening URLs, waiting for page load, evaluating JavaScript, taking snapshots, and extracting auth tokens. Trigger phrases: browser automation, CDP, agent-browser, 浏览器操作, 操作浏览器, Chrome CDP, 复用登录态, extract token from browser."
---

# Browser CDP 操作工具

通过 CDP 协议控制 Chrome，复用已有登录态，执行浏览器自动化操作。

## 前置条件

- 已安装 Google Chrome 或 Chromium
- `agent-browser` 命令行工具已安装
- 需要复用登录态时，优先使用独立调试用户目录，避免影响日常浏览器

---

## 第一步：启动 CDP Chrome 环境

先判断当前系统，再选择启动方式。

### macOS

```bash
bash {SKILL_DIR}/scripts/setup_cdp_chrome.sh 9222
```

### Windows PowerShell

```powershell
$chrome = "${env:ProgramFiles}\Google\Chrome\Application\chrome.exe"
if (-not (Test-Path $chrome)) { $chrome = "${env:ProgramFiles(x86)}\Google\Chrome\Application\chrome.exe" }
Start-Process $chrome -ArgumentList "--remote-debugging-port=9222 --user-data-dir=$env:TEMP\agent-browser-cdp"
```

### Windows Bash / Git Bash

```bash
"/c/Program Files/Google/Chrome/Application/chrome.exe" --remote-debugging-port=9222 --user-data-dir="$TEMP/agent-browser-cdp" >/dev/null 2>&1 &
```

成功后所有 `agent-browser` 命令带 `--cdp 9222`。

---

## 常用操作

### 打开页面并等待加载

```bash
agent-browser --cdp 9222 open "<URL>"
agent-browser --cdp 9222 wait 3000
```

### 提取页面文本内容

```bash
agent-browser --cdp 9222 eval 'document.body.innerText.substring(0, 8000)'
```

### 提取 Auth Token

```bash
# 从 localStorage 或 cookie 提取
agent-browser --cdp 9222 eval 'localStorage.getItem("token") || document.cookie'
```

### 页面截图 / 交互式快照

```bash
# 查找页面元素（用于登录按钮等交互）
agent-browser --cdp 9222 snapshot -i
```

### 点击元素

```bash
agent-browser --cdp 9222 click "<CSS selector>"
```

### 填写表单

```bash
agent-browser --cdp 9222 type "<CSS selector>" "<text>"
```

---

## 常见问题

| 问题 | 解决方案 |
|------|----------|
| CDP 端口未监听 | 重新运行 `setup_cdp_chrome.sh` |
| 页面跳转到登录页 | `snapshot -i` 找登录按钮并操作 |
| eval 返回 null | 检查 localStorage key 名称，或改用 `document.cookie` |
| Chrome 进程残留 | `pkill -9 -f "Google Chrome"` 后重新运行脚本 |
