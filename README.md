# HeadlessDetector
Headless browser detector service

## 测试项 Tests
- **1. 通过User Agent信息检测**
我们从User Agent开始，它是一个检测操作系统和浏览器的常用属性。在Linux下，使用无头Chrome浏览器，User Agent的值为：“Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) HeadlessChrome/59.0.3071.115 Safari/537.36”
因此，我们可以用以下代码检测到它：
```
if (/HeadlessChrome/.test(window.navigator.userAgent)) {
    console.log("Chrome headless detected");
}
```
- **App Version:** same as User Agent above.

- **2. 通过浏览器插件信息检测 Plugins:**
navigator.plugins可以返回一个包含浏览器插件信息的数组。一般来说，在正常的Chrome上，可以找到一些默认插件，比如 Chrome PDF viewer或者 Google Native Client。而在无头模式下，这个返回的数组没有任何插件信息。但,对于其它浏览器这项检测不一定成功，如 Firefox(火狐)，没有默认的插件数。

- **Plugins Prototype:** check if the `Plugin` and `PluginsArray` prototype are correct.


- **Mime Type:** similar to **Plugins** test, where headless browsers don't have any mime type

- **Mime Type Prototype:** check if the `MimeType` and `MimeTypeArray`prototype are correct.

- **Languages:** all headful browser has at least one language. So we can say that if it has no language it's headless.
- **3.通过浏览器的语言信息检测 Languages:**
所有非Headless浏览器至少默认一种语言。因此，获取language的内容为空相当于可以认定其为headless浏览器。例如： 在Chrome中Javascript有两个属性可以获取到用户使用的语言：navigator.language 和 navigator.languages。前者返回浏览器的界面语言，后者返回一个包含用户的首选语言的字符串数组，而在无头模式下，navigator.languages 返回的是空字符串。

- **Webdriver:** 
    this property is true when running in a headless browser.
    相对于Headfull模式,Headless在navigator对象中注册了一个webdriver的属性对象（[请参阅Chromium代码](https://cs.chromium.org/chromium/src/out/Debug/gen/blink/bindings/core/v8/V8Navigator.cpp?rcl=0d3c47615a4f512b82fa0f8da682fb13332b8d32&l=405)）,以支持外部程序向chrome发送相关执行指令。因此，通过测试该属性是否存在，可以检测无头的Chrome。

- **Time elapse:** 
    it pops an `alert()` on page and if it's closed too fast, means that it's headless.

- **Chrome element:** 
    it's specific for `chrome` browser that has an element `window.chrome`.

- **Permission:** 
    in headless mode `Notification.permission` and `navigator.permissions.query` report contradictory values.

- **Devtool:** 
    `puppeteer` works on `devtools protocol`, this test checks if `devtool` is present or not.

- **Broken Image:** 
    all browser has a default `nonzero` broken image size, and this may not happen on a headless browser.

- **Outer Dimension:** 
    the attributes `outerHeight` and `outerWidth` have value 0 on headless browser.

- **Connection Rtt:** 
    The attribute `navigator.connection.rtt`,if present, has value 0 on headless browser.

- **Mouse Move:** 
    The attributes `movementX` and `movementY` on every `MouseEvent` have value 0 on headless browser.

- **
