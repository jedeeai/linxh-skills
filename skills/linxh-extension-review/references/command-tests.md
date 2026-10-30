# 命门 10 分钟验证脚本库

每个评估结尾的「Step 0 验证」部分，不能只说一句「先验证 XX」就完。要给一段**用户能直接复制粘贴就跑的代码**，10 分钟出结果。

这些脚本都是「不写完整插件，先用最简办法证明最关键的一步能不能过」。能过再写插件，过不去就换方向。

**给用户贴代码时，必须配一句大白话**：「你不用看懂代码，把这段复制到 [终端 / 浏览器 Console]，跑出来截图给我看就行。能过我们继续做，过不去我们换方向。」

---

## 1 · 验证目标接口要不要 token / signature（签名码）

**适用**：采集型，怀疑详情接口有签名参数。

**方法 A · 浏览器抠现成请求**

大白话步骤：
1. 在目标网站登录正常的情况下，按 F12 打开开发者工具，切到「网络」（Network）面板。
2. 点你要抓的那个操作（比如打开详情）。
3. 在网络面板找到对应那条请求，右键 → Copy → Copy as cURL。
4. 把复制的命令粘到终端跑一遍，能拿到数据：说明用现成的登录态就够，没签名问题。
5. 把里面某个看起来像签名的参数（比如 URL 里的 `token=xxx` 或 header 里的 `x-csrf-token`）删掉再跑。
6. 还能拿到 → 这个不是命门；拒绝 → 这就是命门。

**方法 B · 浏览器 Console 直接试**

```js
// 在目标网站按 F12，切到 Console，粘贴下面，按回车
fetch('/api/某接口?param=test', {credentials: 'include'})
  .then(r => r.json())
  .then(console.log)
  .catch(console.error)
```

返回正常数据：用浏览器登录态就够，开发可以走插件后台直接请求。
返回错误：要找额外参数。

---

## 2 · 验证站内点链接换页时插件还能不能正常跑

**适用**：UI 增强型、监控型，需要在不同子页面都生效。

装个最简插件，里面 `content.js` 就两行：

```js
console.log('[probe] 我在', location.href)
window.addEventListener('popstate', () => console.log('[probe] URL 变了', location.href))
```

配套 `manifest.json`：
```json
{
  "manifest_version": 3,
  "name": "probe",
  "version": "0.0.1",
  "content_scripts": [{"matches": ["https://目标域名/*"], "js": ["content.js"]}]
}
```

大白话步骤：
1. Chrome 地址栏输 `chrome://extensions/`，打开「开发者模式」，点「加载已解压的扩展程序」，选这个文件夹。
2. 打开目标网站，按 F12，看 Console 里有没有出现 `[probe] 我在 …`。
3. 在站内点一个链接换页，再看 Console：
   - 出现新一行 `[probe] 我在 …` 或 `[probe] URL 变了` → 插件能跟上换页
   - 啥都没出现 → 插件在站内换页时不会重跑，主逻辑要专门处理

---

## 3 · 验证目标输入框能不能用程序塞文字

**适用**：自动化操作型，要给富文本编辑器（推特、微博、知乎、Notion 等）塞内容。

大白话步骤：
1. 打开目标网站，鼠标点一下要塞文字的输入框（让它获得焦点）。
2. 按 F12 打开 Console，把下面 3 段一段一段粘进去跑。
3. 看哪段能让「测试 1 / 2 / 3」的字出现在输入框里，能进的那条就是写入方案。

```js
// 试法 1：直接赋值（普通输入框有效，富文本通常无效）
document.activeElement.value = '测试1'

// 试法 2：模拟粘贴（很多富文本编辑器吃这套）
const el = document.activeElement
el.focus()
const dt = new DataTransfer()
dt.setData('text/plain', '测试2')
el.dispatchEvent(new ClipboardEvent('paste', {clipboardData: dt, bubbles: true}))

// 试法 3：浏览器老接口（兼容性兜底）
document.execCommand('insertText', false, '测试3')
```

---

## 4 · 验证目标网站能不能被动监听请求（零风控）

**适用**：监控型、采集型想走「被动捡」路径，不主动发请求。

装最简插件，`content.js` 里：

```js
const code = `
  const orig = window.fetch
  window.fetch = async function(...args) {
    const resp = await orig.apply(this, args)
    if (args[0]?.includes?.('某接口关键词')) {
      console.log('[hook]', args[0])
      resp.clone().json().then(d => console.log('[hook] 数据', d)).catch(()=>{})
    }
    return resp
  }
`
const s = document.createElement('script')
s.textContent = code
document.documentElement.appendChild(s)
s.remove()
```

把 `'某接口关键词'` 换成目标接口路径里有的字符串（比如 `'graphql'`、`'/api/'`）。

大白话步骤：
1. 装完插件，打开目标网站，按 F12 看 Console。
2. 在网站正常浏览滚动，Console 里应该开始冒 `[hook] …` 日志。
3. 有日志 → 能用「被动捡」方案，零封号风险；没日志 → 网站不走这条路，要再调整。

---

## 5 · 验证浏览器登录态能不能给后台 Python 用

**适用**：要做本地后台抓取的项目。

大白话步骤：

```bash
# 先装一次工具
pip3 install browser_cookie3 requests

# 试一下（DOMAIN 换成目标域名）
python3 - <<'EOF'
import browser_cookie3, requests
DOMAIN = '目标域名.com'
cookies = browser_cookie3.chrome(domain_name=DOMAIN)
print('找到 cookies:', [c.name for c in cookies][:10])
# 试一个登录态接口
r = requests.get(f'https://{DOMAIN}/某登录态接口', cookies=cookies)
print('状态码:', r.status_code)
EOF
```

200 → 登录态可用，按「插件 + 本地后台」架构走。
401 / 403 → 要么 cookie 抽错（可能登录态在别的浏览器 Profile，要指定 cookie 文件路径），要么接口要额外 header。

如果登录态在非默认 Profile（比如你专门用 Profile 5 登录某账号），改成：
```python
import os
PROFILE_PATH = os.path.expanduser('~/Library/Application Support/Google/Chrome/Profile 5/Cookies')
cookies = browser_cookie3.chrome(domain_name=DOMAIN, cookie_file=PROFILE_PATH)
```

---

## 6 · 验证本地服务端口能不能跟插件通信

**适用**：要做插件 + 本地后台的所有项目。

**先起一个最简服务**：
```bash
# 用 Python 一行起 18888（避开本机已被用的端口）
python3 -m http.server 18888 --bind 127.0.0.1
```

**插件 `background.js` 里**：
```js
chrome.runtime.onMessage.addListener((msg, sender, sendResponse) => {
  if (msg.type === 'ping') {
    fetch('http://127.0.0.1:18888/')
      .then(r => r.text())
      .then(t => sendResponse({ok: true, body: t.slice(0, 100)}))
      .catch(e => sendResponse({ok: false, err: String(e)}))
    return true
  }
})
```

大白话步骤：
1. 装完插件，打开任意网页，按 F12 切到 Console。
2. 粘贴 `chrome.runtime.sendMessage({type: 'ping'}, console.log)` 跑。
3. 看到 `{ok: true, body: '…'}` → 插件能跟本地服务通信；看到 `{ok: false, err: …}` → 通信失败，要排查端口或本地服务有没起。

---

## 7 · 验证目标网站有没有反自动化检测

**适用**：自动化操作型，担心被识别为机器人。

大白话步骤：在目标网站 F12 Console 里粘贴下面跑：

```js
console.log({
  webdriver: navigator.webdriver,
  plugins: navigator.plugins.length,
  chrome: typeof window.chrome,
  permissions: typeof navigator.permissions,
})
```

正常浏览器 `webdriver` 是 `false` 或 `undefined`。
如果担心网站读这些字段做反爬，要走 Playwright + stealth 方案而不是简单脚本。

但**用插件操作不是 Playwright**，插件在正常浏览器里跑，这条只在「要不要外包给 Playwright」的形态决策时有用。

---

## 用法收尾

评估结尾「Step 0 验证」部分，从上面挑 1-2 条 **跟命门最相关**的，把代码贴给用户，配大白话：

> 你不用看懂代码，把这段复制到 [终端 / 浏览器 Console]，跑出来截图给我看就行。
> 能过我们就继续，过不去我们换方向。

用户不会编程但会复制粘贴 + 截图，这套验证流程他执行得动。
