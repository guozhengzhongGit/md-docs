# JsBridge 原理

## 起源

web 技术是目前最易编写和部署的 UI 构建方式，开发者可以使用 HTML、CSS、JS 快速构建一个系统，且部署到服务器后，用户无需主动更新就能获取到最新的 UI 和功能，这种开发、部署更新成本极低的技术手段成为了移动端混合开发的不二选择。而任何一个移动操作系统中都包含 web 容器，例如 webview，而 JS 又是 web 技术的核心，因此在移动平台运行 js 不用像运行别的语言需要添加额外的运行环境，所以 JS 理所应当肩负起与其他技术桥接的职责，JSBridge 应运而生

比如微信SDK，常见功能在网页中设置微信的分享内容

## 概念

如其名称**Bridge**描述的那样，是 native 和 web 页面之间双向通信的桥梁

前端视角来看多用于给 JS 提供了调用原生应用功能的接口

各种 web 页面可以通过 Bridge 调用 App 的原生能力，页面呈现使用 web UI，而相关按钮点击后，调用的是 Native 功能

## Hybrid 中的通信原理

### JS 调用 Native

主要有两种方式

1. 注入 API
2. 拦截 URL Scheme

#### 注入 API

通过 Webview 提供的接口，向 JS 的上下文环境中注入对象或者方法，其实就是添加到全局对象 window 上，让 JS 调用时可以直接执行相应的 native 代码逻辑，实现功能

**但是需要注意，使用这种方式时，JS 需要等到 Native 执行完对应逻辑后才能进行调用**

IOS 有两种 Webview

1. UIWebview

2. ==WKWebview（ios8+）==，提供了 `WKScriptMessageHandler`接口

Android 在 ==4.2== 之后提供了`JavascriptInterface` 接口

4.2 之前的接口有安全漏洞，多采用拦截 prompt 的方式来实现

> 问，对于注入 API 的方式来说，因为是 webview 向 window 上添加了方法才能调用，那么如何确定这些 API 已经注入成功了

#### 拦截 URLScheme

Native 加载 Webview 之后，web 发送的所有请求都会经过 webview 组件，因此我们可以自定义 JSBridge 通信的 URLScheme，在  web 端通过 iframe.src 的方式发送请求，native 拦截到以后获取所带参数进行对应操作

缺点：首先是建立请求本身就需要时间，其次要注意 url 长度

优点：兼容 IOS6

> 发送请求为什么用 iframe.src 不用其他的比如 location.href
>
> 如果连续多次修改 location.href 的值，native 只能接收到最后一次请求，前面的请求都会被忽略掉，造成调用丢失的情况

### Native 调用 JS

比较简单，web 把方法挂到全局对象 window 上，native 直接执行拼接 JS 字符串

IOS 的 UIWebview 有 `stringByEvaluatingJavaScriptFromString` 方法

IOS 的 WKWebview 有 `evaluateJavaScript` 方法

Android 4.4 之后有了与 IOS 的同名方法 evaluateJavaScript

Android 4.4 之前只能使用 loadUrl 一段 JS 代码的方式来实现，缺点是无法获取 JS 执行后的结果

## 参考链接

[JSBridge 原理](https://juejin.cn/post/6844903585268891662#comment)

[写给前端工程师的JSBridge原理](https://juejin.cn/post/6847902218763534349#comment)

[使用 JSBridge 与原生 IOS、Android 进行交互有 Demo](https://juejin.cn/post/6844903885555892232#comment)

