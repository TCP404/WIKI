# Content Security Policy (CSP)

> Ref: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Security-Policy


## 什么是 CSP

**内容安全策略 (CSP)** 是一个 HTTP 响应头 (HTTP Response Header)。

它的核心思想是：**“只信任我告诉你的内容来源”**。

作为一个网站的开发者，你可以通过发送一个名为 Content-Security-Policy 的 HTTP 头部，来告诉浏览器一个“白名单”。这个白名单精确地定义了哪些 **资源**（如脚本、样式表、图片、字体等）是可信的，并允许被加载和执行。

如果浏览器在页面上发现一个试图加载的资源（比如一个来自 evil-attacker.com 的脚本）不在这个白名单上，浏览器将 **拒绝加载并执行** 它。

**主要目的**：

- **防御 XSS**： 即使攻击者成功地向你的页面注入了 `<script>` 标签，如果该脚本的来源（或其本身）不在你的 CSP 白名单中，浏览器也不会执行它。
- **防御数据注入**： 限制可以加载的内容类型（例如，禁止 `<img>` 标签加载一个恶意脚本）。
- **防止点击劫持 (Clickjacking)**： 通过特定指令（`frame-ancestors`）来控制你的网站是否可以被嵌入到其他网站的 `<iframe>` 中。

## 工作原理

1. **服务器配置**： 网站管理员在 Web 服务器（如 Nginx, Apache）或后端应用（如 Node.js, Python）中配置，为所有（或特定）的 HTTP 响应添加 `Content-Security-Policy` 头部。
2. **浏览器接收**： 当用户访问该网站时，他们的浏览器接收到这个 HTTP 响应头和其中定义的策略。
3. **浏览器解析与执行**： 浏览器会逐字解析这个策略。
4. **内容加载**： 当浏览器解析 HTML 并需要加载资源（如 `<script src="...">, <img src="...">, <link href="...">` 或执行内联脚本 `<script>alert(1)</script>`）时，它会执行以下检查：
    - 检查来源： 这个资源的 URL 是否在 CSP 策略的相应指令（如 script-src, img-src）的白名单中
    - 检查内联： 这个资源是内联（inline）的吗？CSP 默认禁止所有内联脚本和样式。
    - 检查 eval： 是否有代码在尝试使用 eval()？CSP 默认也禁止它。
5. **阻止或允许**：
    - 如果合规： 资源被正常加载和执行。
    - 如果违规： 浏览器会阻止该资源，并在控制台（Console）中打印一个详细的安全违规错误。
6. **(可选) 报告**： 如果 CSP 策略中定义了 report-uri 或 report-to 指令，浏览器还会将这次违规行为的详细信息（以 JSON 格式）发送到你指定的服务器地址。这对于监控潜在的攻击或调试策略非常有帮助。

例如: 你的网站发送了以下 CSP 头部：

```http
Content-Security-Policy:
    default-src 'self';
    script-src 'self'
        https://trusted.cdn.com;
    img-src 'self' data:;
```

这表示：

- 默认情况下，所有资源只能从同源（'self'）加载。
- 脚本只能从同源和 https://trusted.cdn.com 加载。
- 图片可以从同源加载，或者使用 data URI。
- 如果页面尝试加载一个来自 https://evil-attacker.com 的脚本，浏览器会阻止这个请求，并在控制台中记录一条 CSP 违规的警告。
preview 
## CSP 指令(Directives)

CSP 头由一个或多个 **指令(Directives)** 组成,每个指令之间用分号(`;`)隔开.

每个指令包含一个指令名和一个或多个源值 (Source Values)，源值之间用空格隔开。

基本结构: 

```http
Content-Security-Policy: 
    <directive-name> <source-value1> <source-value2>; 
    <directive-name2> <source-value3>;
```

指令可以分为多种: 

- **Fetch directives(抓取指令)**
- **Document directives(文档指令)**
- **Navigation directives(导航指令)**
- **Reporting directives(报告指令)**
- **Other diretctives(其他指令)**

### 📚 第一部分：抓取指令 (Fetch Directives)
这类指令控制着 **可以加载哪些类型的资源** 以及 **从哪里加载**。它们是 CSP 的核心，也是对抗 XSS 的主要防线。

1. `default-src` **(默认源)**, 这是 CSP 的 **基石和回退 (Fallback)** 指令。
    - **作用**：为所有其他未明确设置的抓取指令（如 `script-src`, `style-src` 等）提供一个默认值。
    - **示例**：`default-src 'self' https://trusted.com;`
    - **解释**：这意味着，如果你的策略里没有 script-src，那么浏览器就会使用 default-src 的值来限制脚本，只允许加载来自 'self' (同源) 和 https://trusted.com 的脚本。

2. `script-src` **(脚本源)**, 这是 CSP 中最关键的指令，是防止 XSS 的核心防线。
    - **作用**： 定义了哪些来源的 JavaScript 脚本可以被执行。
    - **涵盖**：
        - `src` 属性指向的 `<script>` 标签。
        - 内联脚本 (Inline script)，即 `<script>...</script>` (默认禁止)。
        - 内联事件处理器，如 `onclick=""` (默认禁止)。
        - `eval()` 和 `new Function()` 等 (默认禁止)。
    - **示例**： `script-src 'self' https://apis.google.com 'nonce-RANDOM123'`;
    - **解释**： 这允许脚本来自：同源、`apis.google.com`、以及 `nonce` 属性匹配的内联脚本。

3. `script-src-elem` **(脚本-元素)**
    - **作用**： 只控制 `<script>` 元素。
    - **涵盖**： 
      - `<script src="...">` (外部脚本)
      - `<script>...</script>` (内联脚本块)
    - **不涵盖**： onclick 等事件处理器。

4. `script-src-attr` **(脚本-属性)**
    - **作用**： 只控制内联事件处理器属性。
    - **涵盖**：
        - 所有 `on...` 事件处理器, 如`onclick="..."`, `onmouseover="..."`
        - `javascript:` 伪协议（例如 `href="javascript:alert(1)"`）。
    - **不涵盖**： `<script>` 元素。

| HTML 代码示例                                      | 它是“元素”还是“属性”？ | 优先控制它的指令 | 如果优先指令不存在，回退给谁？ |
| -------------------------------------------------- | ---------------------- | ---------------- | ------------------------------ |
| A. `<script src="https:"//a.com/app.js"></script>` | 元素 (Element)         | script-src-elem  | script-src                     |
| B. `<script>alert(1);</script>`                    | 元素 (Element)         | script-src-elem  | script-src                     |
| C. `<button onclick="doWork();">`                  | 属性 (Attribute)       | script-src-attr  | script-src                     |
| D. `<div onmouseover="showTip();"></div>`          | 属性 (Attribute)       | script-src-attr  | script-src                     |
| E. `<a href="javascript:alert(1)">`                | 属性 (Attribute)       | script-src-attr  | script-src                     |
 


> **最重要的概念**： **回退 (Fallback) 机制** 浏览器在寻找一个资源的策略时，会按顺序检查：
> 
> 1. **特定指令**： 比如加载一个脚本，浏览器会先找 `script-src`。
> 2. **回退到 `default-src`**： 如果 `script-src` 不存在，浏览器就会去找 `default-src`。
> 3. **默认允许**： 如果两者都不存在，浏览器会默认允许所有来源（即没有 CSP 保护）。

5. `style-src` **(样式源)**
    - **作用**： 定义了可以加载和应用的 CSS 样式表来源。
    - **涵盖**：
        - `href` 属性指向的 `<link rel="stylesheet">`。
        - `@import` 规则。
        - 内联样式 (Inline style)，即 `<style>...</style>` (默认禁止)。
        - `style` 属性，如 `style="color:red"` (默认禁止)。
    - **示例**： `style-src 'self' https://fonts.googleapis.com 'unsafe-inline';`
        - 注意： `unsafe-inline` 经常在这里被“妥协”使用，因为很多 UI 库会动态修改 `style` 属性。

6. `img-src` **(图像源)**
    - **作用**： 定义了可以加载的图像来源。
    - **涵盖**： `<img>`, `favicon`, 以及 CSS `background-image` 等。
    - **示例**： `img-src 'self' data: https://cdn.my-images.com;`
        - `data`: 允许使用 Base64 编码的内联图像。

7. `connect-src` **(连接源)**
    - **作用**： 定义了脚本（如 `fetch`, `XHR`, `WebSocket`）可以连接到的 URL。
    - **涵盖**： `fetch()`, `XMLHttpRequest`, `WebSocket`, `EventSource`。
    - **示例**： `connect-src 'self' https://api.my-service.com wss://chat.my-service.com;`

8. `font-src` **(字体源)**
    - **作用**： 定义了可以加载的字体文件来源。
    - **涵盖**： `src` 属性在 CSS `@font-face` 规则中指定的 URL。
    - **示例**： `font-src 'self' https://fonts.gstatic.com;`

9. `media-src` **(媒体源)**
    - **作用**： 定义了 `<audio>` 和 `<video>` 标签可以加载的媒体资源来源。
    - **示例**： `media-src 'self' https://videos.my-cdn.com;`

10. `object-src` **(对象源)**
    - **作用**： 定义了 `<object>`, `<embed>` 和 `<applet>` 标签可以加载的资源来源。
    - **强烈建议**： 这个指令非常危险，因为它可以加载 Flash 等插件。**最佳实践是始终将其设置为 `'none'`。**
    - **示例**： `object-src 'none';`

11. `frame-src` **(框架源)**
    - **作用**： 定义了可以嵌入到当前页面的 `<iframe>` 或 `<frame>` 中的 URL。
    - **示例**： `frame-src 'self' https://youtube.com;`
    - **注意**： `child-src` 是一个已废弃的指令，它曾经同时控制框架 (frames) 和 Web Workers。现在它已被 `frame-src` 和 `worker-src` 取代。

12.  `worker-src` **(Worker 源)**
    - **作用**： 定义了可以作为 `Worker`, `SharedWorker` 或 `ServiceWorker` 加载的 URL。
    - **示例**： `worker-src 'self';`


### 📚 第二部分：文档指令 (Document Directives)
这类指令控制 **文档（即整个页面）的属性**，而不是控制加载的具体资源。它们定义了页面可以或不可以使用哪些功能。

1. `base-uri` **(基础 URI)**
    - **作用**： 限制可以在文档的 <base> 标签中设置的 URL。
    - **为什么要用这个？**
        - `<base>` 标签是 HTML 中一个很“霸道”的标签。它会改变页面上所有 **相对路径**（如 `href="page2.html"` 或 `src="img.png"`）的解析基准。
    - **攻击场景（Basejacking）**： 如果一个攻击者能向你的页面注入一个 `<base>` 标签（例如通过 XSS），他就可以将页面上所有相对路径的链接（比如 `<script src="analytics.js">`）指向他自己的恶意服务器。
        - **例如**： 攻击者注入 `<base href="https://evil-attacker.com/">`。你页面上的 `<script src="analytics.js">` 就会被浏览器解析为 `https://evil-attacker.com/analytics.js`，从而加载恶意脚本。
    - **如何防御**： 使用 `base-uri` 来锁定这个基准。
    - **示例**： `base-uri 'self';`
        - 这告诉浏览器，`<base>` 标签（如果存在）的 `href` 属性只能指向你自己的域名（同源）。
        - **最佳实践**： 如果你的网站根本不使用 `<base>` 标签，设置 `base-uri 'none';` 是最安全的选择。

2. `sandbox` **(沙盒)**
    - **作用**： 这是一个 **极其强大** 的指令。它在你的页面上启用一个类似于 `<iframe>` 的沙盒环境，**极大地限制了** 页面的能力。
    - **目的**： 主要用于当你需要展示用户上传的 HTML 内容（例如一个 `.html` 文件）时，或者当你希望对某个页面进行“降权”处理时。
    - **工作方式**： 当你发送 `sandbox` 指令时，它会 **默认禁止 **页面执行几乎所有操作，包括：
        - 执行脚本
        - 提交表单
        - 打开新窗口 (`window.open` 或 `target="_blank"`)
        - 允许弹窗 (`alert`, `confirm`)
        - 使用 cookie (阻止同源访问)
        - ...等等
    - **如何“解禁”**： 你必须通过添加 **白名单值** 来逐个“放行”你认为安全的功能。
    - **示例（空值）**： `sandbox`;
        - 这是最严格的沙盒，页面几乎什么都干不了，只能显示静态的 HTML 和 CSS。
    - **示例（放行部分功能）**： `sandbox allow-forms allow-scripts allow-same-origin;`
        - `allow-forms`：允许页面提交表单。
        - `allow-scripts`：允许页面执行脚本。
        - `allow-same-origin`：**(关键)** 允许页面被视为“同源”，使其可以访问自己的 cookie、LocalStorage 等。（没有这个，页面会被视为一个唯一的、隔离的来源）。
    
    - 注意： `sandbox` 指令是 **不可回退** 的。它不会回退到 `default-src`。它要么存在，要么不存在。它通常用于你 **无法完全信任的特定页面**，而不是用于你的主站点。


### 📚 第三部分：导航指令 (Navigation Directives)





### 📚 第四部分：报告指令 (Reporting Directives)