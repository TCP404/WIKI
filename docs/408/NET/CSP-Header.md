# Content Security Policy (CSP)

!!! info "Reference"

    https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Security-Policy


## 什么是 CSP

**内容安全策略 (CSP)** 是一个 HTTP 响应头 (HTTP Response Header)。

它的核心思想是：**“只信任我告诉你的内容来源”**。

作为一个网站的开发者，你可以通过发送一个名为 **Content-Security-Policy/Content-Security-Policy-Report-Only** 的 HTTP 头部，来告诉浏览器一个“白名单”。这个白名单精确地定义了哪些 **资源**（如脚本、样式表、图片、字体等）是可信的，并允许被加载和执行。

如果浏览器在页面上发现一个试图加载的资源（比如一个来自 `evil-attacker.com` 的脚本）不在这个白名单上，浏览器将 **拒绝加载并执行** 它。

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

- 默认情况下，所有资源只能从同源（`'self'`）加载。
- 脚本只能从同源和 `https://trusted.cdn.com` 加载。
- 图片可以从同源加载，或者使用 data URI。
- 如果页面尝试加载一个来自 `https://evil-attacker.com` 的脚本，浏览器会阻止这个请求，并在控制台中记录一条 CSP 违规的警告。
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

!!! Note ""

    这类指令控制着 **可以加载哪些类型的资源** 以及 **从哪里加载**。它们是 CSP 的核心，也是对抗 XSS 的主要防线。


1. **`default-src` (默认源)**, 这是 CSP 的 **基石和回退 (Fallback)** 指令。
    - **作用**：为所有其他未明确设置的抓取指令（如 `script-src`, `style-src` 等）提供一个默认值。

    !!! example

        ```http
        Content-Security-Policy:
            default-src 'self' https://trusted.com;
        ```

        **解释**：这意味着，如果你的策略里没有 `script-src`，那么浏览器就会使用 `default-src` 的值来限制脚本，只允许加载来自 `'self'` (同源) 和 `https://trusted.com` 的脚本。

2. **`script-src` (脚本源)**, 这是 CSP 中最关键的指令，是防止 XSS 的核心防线。
    - **作用**： 定义了哪些来源的 JavaScript 脚本可以被执行。
    - **涵盖**：
        - `<script src="...">` 外部脚本。
        - `<script>...</script>` 内联脚本块 (默认禁止)。
        - 内联事件处理器，如 `onclick=""` (默认禁止)。
        - `eval()` 和 `new Function()` 等 (默认禁止)。

    !!! Example

        ```http
        Content-Security-Policy:
            script-src 'self' https://apis.google.com 'nonce-RANDOM123';
        ```

        **解释**： 这允许脚本来自：同源、`apis.google.com`、以及 `nonce` 属性匹配的内联脚本。

3. **`script-src-elem` (脚本-元素)**
    - **作用**： 只控制 `<script>` 元素。
    - **涵盖**： 
        - `<script src="...">` (外部脚本)
        - `<script>...</script>` (内联脚本块)
    - **不涵盖**： onclick 等事件处理器。

4. **`script-src-attr` (脚本-属性)**
    - **作用**： 只控制内联事件处理器属性。
    - **涵盖**：
        - 所有 `on...` 事件处理器, 如`onclick="..."`, `onmouseover="..."`
        - `javascript:` 伪协议（例如 `href="javascript:alert(1)"`）。
    - **不涵盖**： `<script>` 元素。

??? note "回退 (Fallback) 机制"

    **回退 (Fallback) 机制** 浏览器在寻找一个资源的策略时，会按顺序检查：

    1. **特定指令**： 比如加载一个脚本，浏览器会先找 `script-src`。
    2. **回退到 `default-src`**： 如果 `script-src` 不存在，浏览器就会去找 `default-src`。
    3. **默认允许**： 如果两者都不存在，浏览器会默认允许所有来源（即没有 CSP 保护）。

??? tip "**`script-src-elem`** VS **`script-src-attr`**"

    | HTML 代码示例                                      | 它是“元素”还是“属性”？ | 优先控制它的指令 | 如果优先指令不存在，回退给谁？ |
    | -------------------------------------------------- | ---------------------- | ---------------- | ------------------------------ |
    | A. `<script src="https:"//a.com/app.js"></script>` | 元素 (Element)         | script-src-elem  | script-src                     |
    | B. `<script>alert(1);</script>`                    | 元素 (Element)         | script-src-elem  | script-src                     |
    | C. `<button onclick="doWork();">`                  | 属性 (Attribute)       | script-src-attr  | script-src                     |
    | D. `<div onmouseover="showTip();"></div>`          | 属性 (Attribute)       | script-src-attr  | script-src                     |
    | E. `<a href="javascript:alert(1)">`                | 属性 (Attribute)       | script-src-attr  | script-src                     |
    | F. `eval()`                                        |                        | script-src       | default-src                    |


7. **`connect-src` (连接源)**
    - **作用**： 定义了脚本（如 `fetch`, `XHR`, `WebSocket`）可以连接到的 URL。
    - **涵盖**： `fetch()`, `XMLHttpRequest`, `WebSocket`, `EventSource`。
    - **示例**： `connect-src 'self' https://api.my-service.com wss://chat.my-service.com;`

??? Tip "**`script-src` VS `connect-src`**"

    - **`script-src`** 是管控 **来自哪里的脚本能运行**, 在页面加载期间检查
    - **`connect-src`** 是管控 **运行的代码能和哪里通信**, 在页面加载完毕, 要发请求前检查

    `script-src` 就像你的厨房(网页)允许哪里来的厨师(代码)进来干活(执行), 是 "**准入许可**", 用于防止恶意代码被执行

    `connect-src` 就像厨房(网页)里的厨师(脚本)允许和哪些食材供应商(连接源)通信, 是"**通信许可**", 用于防止已运行的代码将数据泄露到恶意服务器, 或从恶意服务器获取数据.

    !!! example

        假设你的网站 `my-website.com` 想使用谷歌分析, 但是你这么配置:

        ```http
        Content-Security-Policy:
            script-src 'self' https://www.google-analytics.com;
            connect-src 'self';
        ```

        那么:

        1. **阶段一: 加载(受 `script-src` 管控)**
            - 浏览器看到 `<script src="https://www.google-analytics.com/analytics.js">`。
            - 浏览器检查 `script-src` 策略：`https://www.google-analytics.com` 在白名单里。
            - 结果： ✅ 脚本成功加载并执行。
        2. **阶段二：运行 (受 `connect-src` 管控)**
            - `analytics.js` 脚本开始运行。它收集用户信息，然后尝试打包数据，通过一个 `fetch` (或 `XHR`) 请求将其发送到谷歌的收集服务器（例如 `https://stats.g.doubleclick.net` 或其他 `google.com` 的子域名）。
            - 浏览器检查 `connect-src` 策略：`https://stats.g.doubleclick.net` 不在白名单里 (白名单里只有 `'self'`)。
            - 结果： ❌ 网络请求被阻止。

        最终结果： 谷歌分析的脚本成功 **运行** 了，但它无法 **上报** 任何数据，因此它 **实际上失效了**。


5. **`style-src` (样式源)**
    - **作用**： 定义了可以加载和应用的 CSS 样式表来源。
    - **涵盖**：
        - `href` 属性指向的 `<link rel="stylesheet">`。
        - `@import` 规则。
        - `<style>...</style>` 内联样式 (默认禁止)。
        - `style` 属性，如 `style="color:red"` (默认禁止)。
    
    !!! example

        ```http
        Content-Security-Policy:
            style-src 'self' https://fonts.googleapis.com 'unsafe-inline';
        ```

        **解释**: 这允许样式来自：同源、`fonts.googleapis.com`、以及内联样式。

        **注意**： `unsafe-inline` 经常在这里被“妥协”使用，因为很多 UI 库会动态修改 `style` 属性。

6. **`img-src` (图像源)**
    - **作用**： 定义了可以加载的图像来源。
    - **涵盖**： `<img>`, `favicon`, 以及 CSS `background-image` 等。

    !!! example

        ```http
        Content-Security-Policy:
            img-src 'self' data: https://cdn.my-images.com;
        ```

        **解释**: `data`: 允许使用 Base64 编码的内联图像。


7. **`font-src` (字体源)**
    - **作用**： 定义了可以加载的字体文件来源。
    - **涵盖**： `src` 属性在 CSS `@font-face` 规则中指定的 URL。
    - **示例**： `font-src 'self' https://fonts.gstatic.com;`

8. **`media-src` (媒体源)**
    - **作用**： 定义了 `<audio>` 和 `<video>` 标签可以加载的媒体资源来源。
    - **示例**： `media-src 'self' https://videos.my-cdn.com;`

9.  **`frame-src` (框架源)**
    - **作用**： 定义了可以嵌入到当前页面的 `<iframe>` 或 `<frame>` 中的 URL。
    - **示例**： `frame-src 'self' https://youtube.com;`
    - **注意**： `child-src` 是一个已废弃的指令，它曾经同时控制框架 (frames) 和 Web Workers。现在它已被 `frame-src` 和 `worker-src` 取代。

10. **`object-src` (对象源)**
    - **作用**： 定义了 `<object>`, `<embed>` 和 `<applet>` 标签可以加载的资源来源。
    - **示例**： `object-src 'none';`
    - **强烈建议**： 这个指令非常危险，因为它可以加载 Flash 等插件。**最佳实践是始终将其设置为 `'none'`。**

11. **`worker-src` (Worker 源)**
    - **作用**： 定义了可以作为 `Worker`, `SharedWorker` 或 `ServiceWorker` 加载的 URL。
    - **示例**： `worker-src 'self';`

??? Question "worker 是什么?"

    JS 是单线程的,如果有一些耗时的任务要做时, 页面浏览起来就会比较卡, 所以现代版本的 JS 允许主线程创建 Worker 线程并通过消息通信.

    Worker 线程不能打卡本机文件系统(`file://`), 所以它加载的脚本必须来自网络.


### 📚 第二部分：文档指令 (Document Directives)

!!! Note ""

    这类指令控制 **文档（即整个页面）的属性**，而不是控制加载的具体资源。它们定义了页面可以或不可以使用哪些功能。

#### **`base-uri` (基础 URI)**

- **作用**： 限制可以在文档的 `<base>` 标签中设置的 URL。
- **为什么要用这个？**
  - `<base>` 标签是 HTML 中一个很“霸道”的标签。它会改变页面上所有 **相对路径**（如 `href="page2.html"` 或 `src="img.png"`）的解析基准。

!!! info ""

    === "**攻击场景（Basejacking）**"

        如果一个攻击者能向你的页面注入一个 `<base>` 标签（例如通过 XSS），他就可以将页面上所有相对路径的链接（比如 `<script src="analytics.js">`）指向他自己的恶意服务器。

        **例如**： 

        攻击者注入 `<base href="https://evil-attacker.com/">`。

        你页面上的 `<script src="analytics.js">` 就会被浏览器解析为 `https://evil-attacker.com/analytics.js`，从而加载恶意脚本。

    === "**如何防御**"

        使用 `base-uri` 来锁定这个基准。

        **示例**： `base-uri 'self';`

        这告诉浏览器，`<base>` 标签（如果存在）的 `href` 属性只能指向你自己的域名（同源）。


!!! Tip "最佳实践"

    如果你的网站根本不使用 `<base>` 标签，设置 `base-uri 'none';` 是最安全的选择。

---

#### **`sandbox` (沙盒)**

- **作用**： 这是一个 **极其强大** 的指令。它在你的页面上启用一个类似于 `<iframe>` 的沙盒环境，**极大地限制了** 页面的能力。
- **目的**： 主要用于当你需要展示用户上传的 HTML 内容（例如一个 `.html` 文件）时，或者当你希望对某个页面进行“降权”处理时。
- **工作方式**： 当你发送 `sandbox` 指令时，它会 **默认禁止** 页面执行几乎所有操作，包括：
    - 执行脚本
    - 提交表单
    - 打开新窗口 (`window.open` 或 `target="_blank"`)
    - 允许弹窗 (`alert`, `confirm`)
    - 使用 cookie (阻止同源访问)
    - ...等等
- **如何“解禁”**： 你必须通过添加 **白名单值** 来逐个“放行”你认为安全的功能。

!!! example

    === "空值"

        ```http
        Content-Security-Policy:
            sandbox;
        ```

        - 这是最严格的沙盒，页面几乎什么都干不了，只能显示静态的 HTML 和 CSS。

    === "**放行部分功能**"

        ```http
        Content-Security-Policy:
            sandbox allow-forms allow-scripts allow-same-origin;
        ```

        - `allow-forms`：允许页面提交表单。
        - `allow-scripts`：允许页面执行脚本。
        - `allow-same-origin`：**(关键)** 允许页面被视为“同源”，使其可以访问自己的 cookie、LocalStorage 等。（没有这个，页面会被视为一个唯一的、隔离的来源）。

!!! warning ""

    `sandbox` 指令是 **不可回退** 的。它不会回退到 `default-src`。它要么存在，要么不存在。它通常用于你 **无法完全信任的特定页面**，而不是用于你的主站点。

---

### 📚 第三部分：导航指令 (Navigation Directives)

!!! note ""

    这类指令控制 **用户可以从当前页面导航到哪里**，或者 **谁可以导航到当前页面**。


#### **`form-action` (表单提交)**

- **作用**： 限制 `<form>` 标签的 `action` 属性可以指向的 URL。

!!! info ""

    === "**攻击场景**"

        - 假设攻击者通过 XSS 向你的登录页面注入了一个 **新的、假的登录表单**。这个假表单看起来和你的真表单一样，但它的 `action` 属性指向 `https://evil-attacker.com/steal-password.php`。
        - 当用户在你的（受信任的）域名上输入用户名和密码并点击“登录”时，他们的凭证就会被发送到攻击者的服务器。

    === "**如何防御**"

        使用 `form-action` 来锁定表单可以提交的目的地。

        !!! example

            ```http
            Content-Security-Policy:
                form-action 'self' https://login-partner.com;
            ```

            这告诉浏览器，页面上的所有 `<form>` 只能提交到：

            1. `'self'` (你自己的域名)
            2. `https://login-partner.com` (例如，如果你使用了像 OAuth 这样的第三方登录服务)

            任何试图提交到其他地方的表单都会被 **阻止**。

!!! warning ""

    **注意**： 如果没有设置 `form-action`，浏览器会回退到 `default-src`。


#### **`frame-ancestors` (框架祖先)**

**这是一个极其重要的、用于防止点击劫持 (Clickjacking) 的指令。**

**作用**： 控制 **哪些网站** 可以将你的页面嵌入到 `<iframe>`、`<frame>`、`<object>` 或 `<embed>` 中。

!!! info ""

    === "**如何点击劫持 (Clickjacking)**"

        攻击者在他们自己的网站 `evil.com` 上设置一个页面。

        他们使用一个透明的 `<iframe>` 将你的网站（比如 my-bank.com）整个嵌入进来。

        他们在这个透明的 `<iframe>` 上层放置一个看起来无害的按钮，比如“点我赢奖品”。

        用户访问 `evil.com`，以为自己点击的是“赢奖品”按钮。

        实际上，他们点击的是 **你网站上** 的按钮（比如“确认转账”或“删除账户”），因为你的网站被透明地覆盖在下面。

    === "**如何防御**"

        !!! example ""

            === "`'none';` (最佳实践) "

                ```http
                Content-Security-Policy:
                    frame-ancestors 'none';
                ```

                这告诉浏览器：“任何人都不准把我的网站嵌入到 `<iframe>` 中。” 这能完全杜绝点击劫持攻击。

            === "`'self'`"

                ```http
                Content-Security-Policy:
                    frame-ancestors 'self';
                ```

                只允许你自己的网站（同源）嵌入自己。

            === "domain"

                ```http
                Content-Security-Policy:
                    frame-ancestors https://partner.com;
                ```

                只允许 partner.com 嵌入你的网站。


!!! warning "重要区别"

    - `frame-src` (抓取指令) 控制你的页面 **可以嵌入谁**。
    - `frame-ancestors` (导航指令) 控制 **谁可以嵌入你**。

    另外，`frame-ancestors` 不会回退到 `default-src`。它要么存在，要么不存在。它也取代了旧的、非标准的 `X-Frame-Options` 头部。


### 📚 第四部分：报告指令 (Reporting Directives)

!!! info ""

    这类指令定义了当 CSP 策略被违反时，浏览器应该做什么。它们不会改变策略的执行，只是告诉浏览器“如果出了问题，请通知我”。

#### **`report-uri` (报告 URI)** - [已废弃]

- **作用**： 告诉浏览器将 CSP 违规报告 (JSON 格式) `POST` 到哪个 URL。
- **状态**： **已废弃 (Deprecated)**。
- **示例**： `report-uri /csp-endpoint;`
- **问题**：
    - 尽管它已被废弃，但它仍然是目前兼容性最好的报告指令。
    - 许多浏览器（尤其是旧版本）只支持 `report-uri`。
    - **因此，在现实世界中，你通常需要同时设置 `report-uri` 和 `report-to` 以实现最大兼容性。**

- **Payload**: 

    ```http
    POST /csp-endpoint HTTP/1.1

    Host: your-site.com
    Content-Type: application/csp-report
    User-Agent: Mozilla/5.0 ...
    Content-Length: 178

    {
        "csp-report": {
            "document-uri": "https://your-site.com/page",
            "referrer": "",
            "violated-directive": "script-src-elem",
            "effective-directive": "script-src-elem",
            "original-policy": "default-src 'self'; report-uri /csp-endpoint",
            "disposition": "enforce",
            "blocked-uri": "inline",
            "line-number": 23,
            "script-sample": "<script>alert(1)</script>"
        }
    }
    ```

#### **`report-to` (报告至)**

- **作用**： 这是 `report-uri` 的 **现代替代品**。它旨在与新的 **Reporting API** 协同工作。
- **工作方式**： `report-to` 不直接指定一个 URL，而是指定一个你在另一个 HTTP 头部 (`Report-To(旧)` 或 `Reporting-Endpoints`) 中 **预先定义好的“报告组”的名称**。
- **这样做的好处**：
    - **解耦**： 你可以在一个地方 (`Report-To` / `Reporting-Endpoints` 头部) 定义所有的报告端点（不仅是 CSP，还包括证书透明度、网络错误等），然后在 CSP 策略中通过名称引用它们。
    - **异步**： Reporting API 允许浏览器更智能地批量、异步地发送报告，减少了对你服务器的“报告风暴”。
    - **功能更强**： Reporting API 是一个更广泛的规范，用于报告各种 Web 问题。

- **Payload**:

    ```http
    POST /csp-endpoint HTTP/1.1

    Host: reporter.example.com
    Content-Type: application/reports+json
    Origin: https://your-site.com
    User-Agent: Mozilla/5.0 ...
    Content-Length: 302

    [
        {
            "type": "csp-violation",
            "age": 45100,
            "url": "https://your-site.com/page",
            "user_agent": "Mozilla/5.0 ...",
            "body": {
            "document-uri": "https://your-site.com/page",
            "violated-directive": "script-src-elem",
            "disposition": "enforce",
            "blocked-uri": "inline",
            "script-sample": "<script>alert(1)</script>"
            }
        }
    ]
    ```

??? example

    浏览器兼容性策略 (最佳实践)： 由于 report-to 和 Reporting-Endpoints 还很新，你应该同时提供两者。支持 report-to 的浏览器会优先使用它；不支持的浏览器会回退到 report-uri。

    第一步: 在你的响应中设置 Report-To 头部，定义一个名为 default (或任何你喜欢的名字) 的报告组。

    第二步: 在你的响应中设置 Reporting-Endpoints 头部

    ```HTTP
    Report-To: {
        "group": "csp-reports",
        "max_age": 10886400,
        "endpoints": [
            { "url": "https://reporter.example.com/csp-endpoint" }
        ]
    }
    Reporting-Endpoints: csp-reports="https://reporter.example.com/csp-endpoint"
    ```
    第三步: 在你的 CSP 策略中，使用 report-to 指令引用这个组名。

    第四步: 在你的 CSP 策略中, 使用 report-uri 指令应用这个地址

    ```HTTP
    Content-Security-Policy:
        ...;
        report-uri https://reporter.example.com/csp-endpoint;
        report-to csp-reports;
    ```

    最终你的响应头看起来是这样的:

    ```http
    Report-To: {
        "group": "csp-reports",
        "max_age": 10886400,
        "endpoints": [
            { "url": "https://reporter.example.com/csp-endpoint" }
        ]
    }
    Reporting-Endpoints: csp-reports="https://reporter.example.com/csp-endpoint"
    Content-Security-Policy:
        ...;
        report-uri https://reporter.example.com/csp-endpoint;
        report-to csp-reports;
    ```


### 📚 第五部分：其他指令 (Other Directives)

!!! note ""

    这组指令主要用于处理从 HTTPS 到 HTTP 的降级问题，即 **“混合内容” (Mixed Content)**。

??? question "什么是混合内容？"

    当你的主页面是通过 `https` 安全加载的，但页面上的某些资源（如图片 `<img>`、脚本 `<script>`）却是通过 `http` 不安全加载的，这就叫混合内容。
    现代浏览器会默认 **阻止** “活动”混合内容（如 `http` 脚本），但可能仍会加载“被动”混合内容（如 `http` 图片），这会导致浏览器地址栏的安全锁 🔒 消失，并显示“不安全”警告。

#### **`upgrade-insecure-requests` (升级不安全请求)**

- **作用**： 这是一个 **“自动修复”** 指令。它告诉浏览器：“如果你在这个 `https` 页面上发现任何 `http://` 的请求，请在发送它之前，自动将其重写为 `https://**`。”
- **示例**： `upgrade-insecure-requests;` (它没有值)
- **优点**：
    - 这是一个非常强大的工具，尤其是在迁移一个大型旧网站从 `http` 到 `https` 时。
    - 假设你的旧文章里有成千上万个硬编码的 `http://` 图片链接。你不需要去数据库里修改每一条，只需要添加这个 CSP 指令，浏览器就会自动尝试用 `https://` 加载它们。
- **缺点**：
    - 如果对方服务器不支持 `https` 怎么办？ 如果浏览器将 `http://example.com/img.png` 升级为 `https://example.com/img.png`，但 `example.com` 的服务器并没有配置 `https` 证书，那么这个请求会 **失败**。资源将无法加载。
- **另一个作用**： 它还会自动将所有导航到 `http://` 站点的链接（当用户点击时）重定向到 `https` 版本。


#### **`block-all-mixed-content` (阻止所有混合内容)**

- **作用**： 这是一个 **“严格模式”** 指令。它告诉浏览器：“不要尝试升级，直接阻止所有 `http://` 资源（包括图片、脚本等）在我的 `https://` 页面上加载。”
- **示例**： `block-all-mixed-content;`
- **优点**：
    - 提供最强的安全性，确保你的 `https` 页面上 100% 没有不安全的内容。
- **缺点**：
    - 如果你意外地引用了一个 `http` 资源（即使是支持 `https` 的），它也会被阻止，而不是像 `upgrade-insecure-requests` 那样被“自动修复”。
- **注意**：
    - 根据 CSP Level 3 规范，如果一个策略同时包含了 `upgrade-insecure-requests` 和 `block-all-mixed-content`，`upgrade-insecure-requests` 会被忽略，`block-all-mixed-content` 优先。
    - 然而，现代浏览器（Chrome/Firefox）已经默认会阻止“活动”混合内容（如脚本）。如果你设置了 `upgrade-insecure-requests`，它实际上已经隐含了“阻止那些无法被升级的混合内容”的意思，因此 `block-all-mixed-content` 在很大程度上已经被 `upgrade-insecure-requests` 覆盖或废弃了。
- **最佳实践**： 优先使用 `upgrade-insecure-requests`。
  

*[Reporting API] https://developer.mozilla.org/en-US/docs/Web/API/Reporting_API