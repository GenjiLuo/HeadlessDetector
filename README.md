# HeadlessDetector
Headless browser detector service

## 测试项 Tests
- **1. 通过User Agent信息检测**
使用无头Chrome浏览器，User Agent的值为：“Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) HeadlessChrome/59.0.3071.115 Safari/537.36”，无头模式下的UA会带有HeadlessChrome关键字
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
```
if(navigator.languages == "") {
    console.log("Chrome headless detected");
}
```
- **Webdriver:** 
    this property is true when running in a headless browser.
    相对于Headfull模式,Headless在navigator对象中注册了一个webdriver的属性对象（[请参阅Chromium代码](https://cs.chromium.org/chromium/src/out/Debug/gen/blink/bindings/core/v8/V8Navigator.cpp?rcl=0d3c47615a4f512b82fa0f8da682fb13332b8d32&l=405)）,以支持外部程序向chrome发送相关执行指令。因此，通过测试该属性是否存在，可以检测无头的Chrome。
```
if(navigator.webdriver) {
    console.log("Chrome headless detected");
}
```
- **Time elapse:** 
    it pops an `alert()` on page and if it's closed too fast, means that it's headless.

- **Chrome element:** 
    window.chrome是一个似乎为Chrome扩展开发人员提供功能的对象。虽然它在原始模式下可用，但在无头模式下不可用。

- **Permission:** 
目前无法在无头模式下处理权限。因此，它导致不一致状态，其中Notification.permission和navigator.permissions.query报告矛盾的值。

- **Devtool:** 
    `puppeteer` works on `devtools protocol`, this test checks if `devtool` is present or not.
    `puppeteer` 依赖 `devtools protocol`与headless进行通信, 通过检测当前通信中是否存在是`devtool`特征字，就可以判断是否使用了headless

- **6.通过加载失败的图片情况检测:**
6.通过加载失败的图片情况检测:**
all browser has a default `nonzero` broken image size, and this may not happen on a headless browser.
所有浏览器对于加载失败、损坏的图片都有默认值：`nonzero`,例如：Chrome 对于无法加载的图片的尺寸处理。在正常模式下，加载失败的图片的大小取决于浏览器的缩放情况，但绝不会等于零。而在无头模式下，此类图片的宽高都等于零。
```
var body = document.getElementsByTagName("body")[0];
var image = document.createElement("img");
image.src = "http://iloveponeydotcom32188.jg";
image.setAttribute("id", "fakeimage");
body.appendChild(image);
image.onerror = function(){
    if(image.width == 0 && image.height == 0) {
        console.log("Chrome headless detected");
    }
}
```
- **4.通过WebGL查看图形驱动信息检测**
WebGL是一个在HTML canvas中执行3D渲染的API，它可以查询到图形驱动的渲染器和发布者。

在Linux环境下，我利用Chrome正常模式获得的渲染器和发布者分别为：“Google SwiftShader” 和 “Google Inc”。而在无头模式下，我获得的是 “Mesa OffScreen” 和 “Brian Paul”，前者是不对任何窗口进行渲染的技术，后者是开发了 Mesa 开源图形库 的组织。
```
var canvas = document.createElement('canvas');
var gl = canvas.getContext('webgl');

var debugInfo = gl.getExtension('WEBGL_debug_renderer_info');
var vendor = gl.getParameter(debugInfo.UNMASKED_VENDOR_WEBGL);
var renderer = gl.getParameter(debugInfo.UNMASKED_RENDERER_WEBGL);

if(vendor == "Brian Paul" && renderer == "Mesa OffScreen") {
    console.log("Chrome headless detected");
}
```
然且，并不是所有的无头Chrome都能得到相同的发布者和渲染器信息，有的会带有正常浏览器版本的相关信息。但是，“Mesa OffScreen” 和 “Brian Paul” 代表的一定是无头模式。

- **Outer Dimension:** 
    the attributes `outerHeight` and `outerWidth` have value 0 on headless browser.

- **Connection Rtt:** 
    The attribute `navigator.connection.rtt`,if present, has value 0 on headless browser.

- **Mouse Move:** 
    The attributes `movementX` and `movementY` on every `MouseEvent` have value 0 on headless browser.

- **5.通过浏览器的Hairline功能检测:**
Modernizr 库可以测试浏览器中是否能展示 HTML 和 CSS 样式。我们在 Chrome 和 无头Chrome 之间目前只找到了一个区别，就是后者不支持 Hairline 功能，所以我们可以通过此功能的有无进行检测。
```
if(!Modernizr["hairline"]) {
    console.log("It may be Chrome headless");
}
```

# 附录
##针对Chrome 禁止浏览器返回webdriver对象,让百度、淘宝、天猫之类的网站无法通过该属性对象 判断为headless爬虫
```
Object.defineProperties(navigator,{webdriver:{get:()=>false}})
```
或
```
通过原型删除该属性
delete navigator.__proto__.webdriver;
```
