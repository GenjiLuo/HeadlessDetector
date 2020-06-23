# HeadlessDetector
Headless browser detector servic
[Demo](http://www.stathub.cn/headless/)

## 测试项 Tests
- **1. User Agent信息检测**
    使用无头Chrome浏览器，User Agent的值为：“Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) HeadlessChrome/59.0.3071.115 Safari/537.36”，无头模式下的UA会带有HeadlessChrome关键字
因此，我们可以用以下代码检测到它：
```
if (/HeadlessChrome/.test(window.navigator.userAgent)) {
    console.log("Chrome headless detected");
}
```
- **2.版本信息:** 
    无头模式下的版本信息中会带有HeadlessChrome关键字
```
if(/HeadlessChrome/.test(window.navigator.appVersion)){
    console.log("Chrome headless detected");
}
```

- **3. 浏览器插件信息检测:**
    navigator.plugins可以返回一个包含浏览器插件信息的数组。一般来说，在正常的Chrome上，可以找到一些默认插件，比如 Chrome PDF viewer或者 Google Native Client。而在无头模式下，这个返回的数组没有任何插件信息。但,对于其它浏览器这项检测不一定成功，如 Firefox(火狐)，没有默认的插件数。
```
if(window.navigator.plugins.length <=0){
    console.log("Chrome headless detected");
}
```

- **4.`Plugins`的属性检测:** 
    通过比较`PluginArray.prototype` 和 `navigator.plugins.__proto__` 及 当 plugins存在时,`Plugin.prototype` 和 `navigator.plugins[0].__proto__`是否一致，判断是否正处于headless浏览器环境下。

- **5.Mime Type:** 
    与**3. 浏览器插件信息检测**相似,headless没有`mime type`，navigator.mimeTypes 为0。

- **6.Mime Type Prototype:** 
    与**4.`Plugins`的属性检测**相似 ，如果`navigator.MimeType` 和 `navigator.MimeTypeArray`不一致，则，可以认为正处于headless浏览器环境下。

- **7.浏览器的语言信息检测 Languages:**
所有非Headless浏览器至少默认一种语言。因此，获取language的内容为空相当于可以认定其为headless浏览器。例如： 在Chrome中Javascript有两个属性可以获取到用户使用的语言：navigator.language 和 navigator.languages。前者返回浏览器的界面语言，后者返回一个包含用户的首选语言的字符串数组，而在无头模式下，navigator.languages 返回的是空字符串。
```
if(navigator.languages == "") {
    console.log("Chrome headless detected");
}
```
- **8.Webdriver属性检测:** 
    相对于Headfull模式,Headless在navigator对象中注册了一个webdriver的属性对象（[请参阅Chromium代码](https://cs.chromium.org/chromium/src/out/Debug/gen/blink/bindings/core/v8/V8Navigator.cpp?rcl=0d3c47615a4f512b82fa0f8da682fb13332b8d32&l=405)）,以支持外部程序向chrome发送相关执行指令。因此，通过测试该属性是否存在，可以检测无头的Chrome。
```
if(navigator.webdriver) {
    console.log("Chrome headless detected");
}
```
- **~~9.Time elapse:~~** 
    ~~通过`alert()`弹出,停留的时间不会太短，而，爬虫设计一般都会直接让chrome直接忽略弹窗。~~

- **10.Chrome 属性检测:** 
    window.chrome是一个似乎为Chrome扩展开发人员提供功能的对象。虽然它在原始模式下可用，但在无头模式下不可用。

- **11.Permission处理检测:** 
目前无法在无头模式下处理权限。因此，它导致不一致状态，其中Notification.permission和navigator.permissions.query报告矛盾的值。

- **12.Devtool特征检测:**
    `puppeteer` 依赖 `devtools protocol`与headless进行通信, 通过检测当前通信中是否存在是`devtool`特征字，就可以判断是否使用了headless

- **13.通过加载失败的图片情况检测:**
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
- **14.通过WebGL查看图形驱动信息检测**
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

- **15.Outer Dimension:** 
    获取整个浏览器窗口的高度`outerHeight`和`outerWidth`，包括侧边栏（如果存在）、窗口镶边（window chrome）和窗口调正边框，如果工作在headless下,这两个值为0。

- **16.Connection Rtt:** 
    `navigator.connection` 是只读的，提供一个NetworkInformation 对象来获取设备的网络连接信息。例如用户设备的当前带宽或连接是否被计量， 这可以用于基于用户的连接来选择高清晰度内容或低清晰度内容。`navigator.connection.rtt` 代表（估算的往返时间）,headless下该值为0。

- **17.Mouse Move:** 
    The attributes `movementX` and `movementY` on every `MouseEvent` have value 0 on headless browser.
     MouseEvent.`movementX`和`movementY` 是只读属性，它提供了当前事件和上一个mousemove事件之间鼠标在水平/垂直的移动值。换句话说，这个值是这样计算的 : currentEvent.movementX = currentEvent.screenX - previousEvent.screenX，`movementY`同理。
    根据**15.Outer Dimension:** 可得，`movementX` 和 `movementY`在headless下一定均为0。

- **18.通过浏览器的Hairline功能检测:**
    [Modernizr](https://modernizr.com/) 库可以测试浏览器中是否能展示 HTML 和 CSS 样式。我们在 Chrome 和 无头Chrome 之间目前只找到了一个区别，就是后者不支持 Hairline 功能，所以我们可以通过此功能的有无进行检测。
```
if(!Modernizr["hairline"]) {
    console.log("It may be Chrome headless");
}
```

# 附录
## 针对Chrome 禁止浏览器返回webdriver对象,让百度、淘宝、天猫之类的网站无法通过该属性对象 判断为headless爬虫
```
Object.defineProperties(navigator,{webdriver:{get:()=>false}})
```
或
```
通过原型删除该属性
delete navigator.__proto__.webdriver;
```
>补充说明：大麦网或淘宝网的滑块验证码首先就会检测环境，
>通常会利用sufei_data文件检测当前浏览器信息，其中检测webdriver代码如下
```
function r() {
            return "$cdc_asdjflasutopfhvcZLmcfl_"in u || f.webdriver
        }
        
// 完整的检测代码，这个文件会经常升级        
// https://g.alicdn.com/secdev/sufei_data/3.6.8/index.js
```
>因此在尝试拖动滑块的时候，先要修改该属性。不然如何修改路径都会提示错误，并要求重试。
>document.$cdc_asdjflasutopfhvcZLmcfl_通常在使用selenium时会出现

## chrome属性检测
>在无头模式下window.chrome属性是undefined，而在正常有界面模式下，定义如下:
```
csi: ƒ ()
embeddedSearch: {searchBox: {…}, newTabPage: {…}}
loadTimes: ƒ ()
app: (...)
runtime: (...)
webstore: (...)
get app: ƒ nativeGetter()
set app: ƒ nativeSetter()
get runtime: ƒ nativeGetter()
set runtime: ƒ nativeSetter()
get webstore: ƒ nativeGetter()
set webstore: ƒ nativeSetter()
```
>因此绕过检测修改属性即可
```
window.navigator.chrome = {
    runtime: {},
    // etc.
};
```
## Permissions检测
```
(async () => {
  const permissionStatus = await navigator.permissions.query({ name: 'notifications' });
  if(Notification.permission === 'denied' && permissionStatus.state === 'prompt') {
    // headless
  }
})();
```
>无头模式下Notification.permission与navigator.permissions.query会返回相反的值。因此绕过的方式如下。
```
// Pass the Permissions Test.
await page.evaluateOnNewDocument(() => {
  const originalQuery = window.navigator.permissions.query;
  return window.navigator.permissions.query = (parameters) => (
    parameters.name === 'notifications' ?
      Promise.resolve({ state: Notification.permission }) :
      originalQuery(parameters)
  );
});
```
## Plugins长度检测
>无头模式下navigator.plugins.length返回0,绕过方式如下:
```
Object.defineProperty(navigator, 'plugins', {
    get: () => [1, 2, 3, 4, 5],
  });
```
>注意：反爬除了检查长度，还会检查内容。如果你设置了长度，别忘了再设置内容。防止被反爬。
## The Languages检测
>navigator.languages检测方法,绕过方法:
```
Object.defineProperty(navigator, 'languages', {
    get: () => ['en-US', 'en'],
  });
```
# 参考
-[Web API 接口参考](https://developer.mozilla.org/zh-CN/docs/Web/API)

-[Detecting Chrome headless, the game goes on!](https://antoinevastel.com/bot%20detection/2019/07/19/detecting-chrome-headless-v3.html)

-[Detecting Chrome headless, new techniques](https://antoinevastel.com/bot%20detection/2018/01/17/detect-chrome-headless-v2.html)

-[Detecting Chrome Headless](https://antoinevastel.com/bot%20detection/2017/08/05/detect-chrome-headless.html)

-[FP-Scanner, a bot detection library based on browser fingerprinting](https://antoinevastel.com/bot%20detection/2018/11/13/fp-scanner-library-demo.html)

-[IT IS *NOT* POSSIBLE TO DETECT AND BLOCK CHROME HEADLESS](https://intoli.com/blog/not-possible-to-block-chrome-headless/)

-[MAKING CHROME HEADLESS UNDETECTABLE](https://intoli.com/blog/making-chrome-headless-undetectable/)

-[What is the list of possible values for navigator.platform as of today?](https://stackoverflow.com/questions/19877924/what-is-the-list-of-possible-values-for-navigator-platform-as-of-today)

-[detect-headless](https://github.com/infosimples/detect-headless)
