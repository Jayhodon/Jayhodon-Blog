---
title: 实用性前端代码片段【Note】

date: 2022-05-29 18:45:56

tags: [前端,个人知识库,JavaScript-Code]

categories: [前端, 个人知识库]


summary: '该文用于记录一些实用的 JavaScript 工具代码块'

---



。

## JavaScript



### 数据相关



#### 数组去重

```js
/**
 * 数组去重
 * @param {*} arr
 */
export function uniqueArray(arr) {
    if (!Array.isArray(arr)) {
        throw new Error('The first parameter must be an array')
    }
    if (arr.length == 1) {
        return arr
    }
    return [...new Set(arr)]
}
```



#### 解构

平常我们需要用到一个嵌套多层的对象中某些属性，会将其解构出来使用

```js
let obj = {
  part1: {
    name: '零一',
    age: 23
  }
}

const { part1: { name, age }, part1 } = obj
console.log(part1)   // {name: "零一", age: 23}
console.log(name, age)  // 零一  23
```



#### 数字分隔符

```js
const myMoney = 1_000_000_000_000	// 这样写是没问题的，而且数字分割开后也更直观！！
console.log(myMoney)  // 1000000000000
```



#### 英文字符串首字母大写

Javascript没有内置的首字母大写函数，因此我们可以使用以下代码。

```js
const capitalize = str => str.charAt(0).toUpperCase() + str.slice(1)
    
capitalize("follow for more") // Result: Follow for more
```



#### 校验数字是奇数还是偶数

```js
const isEven = num => num % 2 === 0;
    
console.log(isEven(2));  // Result: True
```



#### 求数字的平均值

使用`reduce`方法找到多个数字之间的平均值。

```js
const average = (...args) => args.reduce((a, b) => a + b) / args.length;
    
average(1, 2, 3, 4);
// Result: 2.5
```



#### 翻转字符串

可以使用 `split`、`reverse` 和 `join` 方法轻松反转字符串。

```js
const reverse = str => str.split('').reverse().join('');
    
reverse('hello world');     
// Result: 'dlrow olleh'
```



#### 打乱数组

可以使用`sort` 和 `random` 方法打乱数组

```js
const shuffleArray = (arr) => arr.sort(() => 0.5 - Math.random());
    
console.log(shuffleArray([1, 2, 3, 4]));
// Result: [ 1, 4, 3, 2 ]
```



#### 生成随机字符串

使用场景：用于前端生成随机的ID,毕竟现在的Vue和React都需要绑定key

```js
const str = Math.random().toString(36).substr(2, 10);
console.log(str);   // 'w5jetivt7e'
```



#### 保留到小数点以后n位

使用场景：JS的浮点数超长，有时候页面显示时需要保留2位小数

```js
// 保留小数点以后几位，默认2位
export function cutNumber(number, no = 2) {
    if (typeof number != 'number') {
        number = Number(number)
    }
    return Number(number.toFixed(no))
}
```



#### cleanObject - 去除对象中value为空的属性

去除对象中value为空(null,undefined,'')的属性,举个栗子：

```js
export const isFalsy = (value) => (value === 0 ? false : !value);

export const isVoid = (value) =>
  value === undefined || value === null || value === "";

export const cleanObject = (object) => {
  // Object.assign({}, object)
  if (!object) {
    return {};
  }
  const result = { ...object };
  Object.keys(result).forEach((key) => {
    const value = result[key];
    if (isVoid(value)) {
      delete result[key];
    }
  });
  return result;
};


let res=cleanObject({
    name:'',
    pageSize:10,
    page:1
})
console.log("res", res) //输入{page:1,pageSize:10}
```





### 颜色相关



#### 颜色RGB转十六进制

```js
const rgbToHex = (r, g, b) => "#" + ((1 << 24) + (r << 16) + (g << 8) + b).toString(16).slice(1);
    
rgbToHex(0, 51, 255); // Result: #0033ff
```



#### 生成随机十六进制颜色

可以使用 `Math.random` 和 `padEnd` 属性生成随机的十六进制颜色。

```js
const randomHex = () => `#${Math.floor(Math.random() * 0xffffff).toString(16).padEnd(6, "0")}`;
    
 console.log(randomHex());// Result: #92b008
```





### 日期相关



#### 检查日期是否合法

使用以下代码段检查给定日期是否有效。

```js
const isDateValid = (...val) => !Number.isNaN(new Date(...val).valueOf());
    
isDateValid("December 17, 1995 03:24:00");// Result: true
```



#### 查找日期位于一年中的第几天

```js
const dayOfYear = (date) =>
      Math.floor((date - new Date(date.getFullYear(), 0, 0)) / 1000 / 60 / 60 / 24);
    
dayOfYear(new Date());// Result: 272
```



#### 计算2个日期之间相差多少天

```js
const dayDif = (date1, date2) => Math.ceil(Math.abs(date1.getTime() - date2.getTime()) / 86400000)
    
dayDif(new Date("2020-10-21"), new Date("2021-10-22")) // Result: 366
```





### 浏览器相关



#### 从 URL 获取查询参数

可以通过传递 `window.location` 或原始 URL `goole.com?search=easy&page=3`轻松地从 url 检索查询参数

```js
const getParameters = (URL) => {
  URL = JSON.parse(
    '{"' +
      decodeURI(URL.split("?")[1])
        .replace(/"/g, '\\"')
        .replace(/&/g, '","')
        .replace(/=/g, '":"') +
      '"}'
  );
  return JSON.stringify(URL);
};

getParameters(window.location);
// Result: { search : "easy", page : 3 }
```

或者更为简单的：

```js
Object.fromEntries(new URLSearchParams(window.location.search))
// Result: { search : "easy", page : 3 }
```



#### 回到顶部

可以使用 `window.scrollTo(0, 0)` 方法自动滚动到顶部。将 `x` 和 `y` 都设置为 0。

```js
const goToTop = () => window.scrollTo(0, 0);
    
goToTop();
```



#### 休眠指定毫秒数

```js
/**
 * 休眠xxxms
 * @param {Number} milliseconds
 */
export function sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms))
}

//使用方式
const fetchData=async()=>{
 await sleep(1000)
}
```



#### 复制内容到剪贴板

```js
export function copyToBoard(value) {
    const element = document.createElement('textarea')
    document.body.appendChild(element)
    element.value = value
    element.select()
    if (document.execCommand('copy')) {
        document.execCommand('copy')
        document.body.removeChild(element)
        return true
    }
    document.body.removeChild(element)
    return false
}
```

>  原理：
>
> 1. 创建一个textare元素并调用select()方法选中
> 2. document.execCommand('copy')方法，拷贝当前选中内容到剪贴板。



借助`navigator.clipboard.writeText`可以很容易的讲文本复制到剪贴板

> 规范要求在写入剪贴板之前使用 Permissions API 获取“剪贴板写入”权限。但是，不同浏览器的具体要求不同，因为这是一个新的API。有关详细信息，请查看compatibility table and Clipboard availability in Clipboard。

```js
const copyToClipboard = (text) => navigator.clipboard.writeText(text);
    
copyToClipboard("Hello World");
```



#### 获取用户选择的文本

使用内置的`getSelection` 属性获取用户选择的文本。

```js
const getSelectedText = () => window.getSelection().toString();
    
getSelectedText();
```



#### 获取浏览器Cookie的值

通过`document.cookie` 来查找`cookie`值

```js
const cookie = name => `; ${document.cookie}`.split(`; ${name}=`).pop().split(';').shift();
    
cookie('_ga'); // Result: "GA1.2.1929736587.1601974046"
```



#### 清除全部Cookie

通过使用`document.cookie`访问cookie并将其清除，可以轻松清除网页中存储的所有cookie。

```js
const clearCookies = document.cookie.split(';').forEach(cookie => document.cookie = cookie.replace(/^ +/, '').replace(/=.*/, `=;expires=${new Date(0).toUTCString()};path=/`));
```





### 文件相关



#### 获取文件后缀名

使用场景：上传文件判断后缀名

```js
/**
 * 获取文件后缀名
 * @param {String} filename
 */
 export function getExt(filename) {
    if (typeof filename == 'string') {
        return filename
            .split('.')
            .pop()
            .toLowerCase()
    } else {
        throw new Error('filename must be a string type')
    }
}
```



#### 对象转化为FormData对象

使用场景：上传文件时我们要新建一个FormData对象，然后有多少个参数就append多少次，使用该函数可以简化逻辑

```js
/**
 * 对象转化为formdata
 * @param {Object} object
 */

 export function getFormData(object) {
    const formData = new FormData()
    Object.keys(object).forEach(key => {
        const value = object[key]
        if (Array.isArray(value)) {
            value.forEach((subValue, i) =>
                formData.append(key + `[${i}]`, subValue)
            )
        } else {
            formData.append(key, object[key])
        }
    })
    return formData
}
```

使用方式：

```js
let req={
    file:xxx,
    userId:1,
    phone:'15198763636',
    //...
}
fetch(getFormData(req))
```



#### 下载一个excel文档 

同时适用于word,ppt等浏览器不会默认执行预览的文档,也可以用于下载后端接口返回的流数据。

```js
//下载一个链接 
function download(link, name) {
    if(!name){
    	name=link.slice(link.lastIndexOf('/') + 1)
    }
    let eleLink = document.createElement('a')
    eleLink.download = name
    eleLink.style.display = 'none'
    eleLink.href = link
    document.body.appendChild(eleLink)
    eleLink.click()
    document.body.removeChild(eleLink)
}
//下载excel
download('http://111.229.14.189/file/1.xlsx')
```

##### 在浏览器中自定义下载一些内容

场景：我想下载一些DOM内容，我想下载一个JSON文件

```js
/**
 * 浏览器下载静态文件
 * @param {String} name 文件名
 * @param {String} content 文件内容
 */
function downloadFile(name, content) {
    if (typeof name == 'undefined') {
        throw new Error('The first parameter name is a must')
    }
    if (typeof content == 'undefined') {
        throw new Error('The second parameter content is a must')
    }
    if (!(content instanceof Blob)) {
        content = new Blob([content])
    }
    const link = URL.createObjectURL(content)
    download(link, name)
}
```

使用方式：

```js
downloadFile('1.txt','lalalallalalla')
downloadFile('1.json',JSON.stringify({name:'hahahha'}))
```

##### 下载后端返回的流

```js
 download('http://111.229.14.189/gk-api/util/download?file=1.jpg')
 download('http://111.229.14.189/gk-api/util/download?file=1.mp4')
```

##### 提供一个图片链接，点击下载

图片、pdf等文件，浏览器会默认执行预览，不能调用download方法进行下载，需要先把图片、pdf等文件转成blob，再调用download方法进行下载，转换的方式是使用axios请求对应的链接

```js
//可以用来下载浏览器会默认预览的文件类型，例如mp4,jpg等
import axios from 'axios'
//提供一个link，完成文件下载，link可以是  http://xxx.com/xxx.xls
function downloadByLink(link,fileName){
    axios.request({
        url: link,
        responseType: 'blob' //关键代码，让axios把响应改成blob
    }).then(res => {
 const link=URL.createObjectURL(res.data)
        download(link, fileName)
    })

}
```

> 注意：会有同源策略的限制，需要配置转发



### 操作系统相关



#### 解决ios audio无法自动播放、循环播放的问题

`ios`手机在使用`audio`或者`video`播放的时候，个别机型无法实现自动播放，可使用下面的代码`hack`。

```js
// 解决ios audio无法自动播放、循环播放的问题
var music = document.getElementById('video');
var state = 0;

document.addEventListener('touchstart', function(){
    if(state==0){
        music.play();
        state=1;
    }
}, false);

document.addEventListener("WeixinJSBridgeReady", function () {
    music.play();
}, false);

//循环播放
music.onended = function () {
    music.load();
    music.play();
}
```



#### 检查用户的设备是否处于暗模式

使用以下代码检查用户的设备是否处于暗模式。

```js
const isDarkMode = window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches
    
console.log(isDarkMode) 
// Result: True or False
```



## Css

### Css相关

#### calc

这是一个`css`属性，我一般称之为`css`表达式。可以计算`css`的值。最有趣的是他可以计算不同单位的差值。很好用的一个功能，缺点是不容易阅读。接盘侠没办法一眼看出`20px`是啥。

```css
div {
    width: calc(25% - 20px);
}
```



#### 使用css写出一个三角形角标

元素宽高设置为`0`，通过`border`属性来设置，让其它三个方向的`border`颜色为透明或者和背景色保持一致，剩余一条`border`的颜色设置为需要的颜色。

```css
div {
    width: 0;
    height: 0;
    border: 5px solid #transparent;
    border-top-color: red;
}
```



#### 水平垂直居中

我一般只使用两种方式`定位`或者`flex`，我觉得够用了。

```css
div {
    width: 100px;
    height: 100px;
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    margin: auto;
}
```

父级控制子集居中

```css
.parent {
    display: flex;
    justify-content: center;
    align-items: center;
}
```



#### css一行文本超出...

```css
overflow: hidden;
text-overflow:ellipsis;
white-space: nowrap;
```



#### 多行文本超出显示...

```css
display: -webkit-box;
-webkit-box-orient: vertical;
-webkit-line-clamp: 3;
overflow: hidden;
```



#### IOS手机容器滚动条滑动不流畅

```css
overflow: auto;
-webkit-overflow-scrolling: touch;
```



#### 修改滚动条样式

隐藏`div`元素的滚动条

```css
div::-webkit-scrollbar {
    display: none;
}
```

>  div::-webkit-scrollbar 滚动条整体部分div::-webkit-scrollbar-thumb 滚动条里面的小方块，能向上向下移动（或往左往右移动，取决于是垂直滚动条还是水平滚动条）
>
> div::-webkit-scrollbar-track 滚动条的轨道
>
> div::-webkit-scrollbar-button 滚动条的轨道的两端按钮，允许通过点击微调小方块的位置。
>
> div::-webkit-scrollbar-track-piece 内层轨道，滚动条中间部分
>
> div::-webkit-scrollbar-corner 边角，即两个滚动条的交汇处
>
> div::-webkit-resizer 两个滚动条的交汇处上用于通过拖动调整元素大小的小控件
>
> **注意此方案有兼容性问题**，一般需要隐藏滚动条时我都是用一个色块通过定位盖上去，或者将子级元素调大，父级元素使用overflow-hidden截掉滚动条部分。暴力且直接。



#### 隐藏页面元素

>  display-none: 元素不会占用空间，在页面中不显示，子元素也不会显示。
>
> opacity-0: 元素透明度将为`0`，但元素仍然存在，绑定的事件仍旧有效仍可触发执行。
>
> visibility-hidden：元素隐藏，但元素仍旧存在，占用空间，页面中无法触发该元素的事件。

