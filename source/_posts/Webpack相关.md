---


title: Webpack相关

date: 2022-05-28 18:45:56

tags: webpack


---


> **webpack** 是一个用于现代 JavaScript 应用程序的 *静态模块打包工具*。当 webpack 处理应用程序时，它会在内部构建一个依赖图，此依赖图对应映射到项目所需的每个模块，并生成一个或多个 *bundle*。

## Webpack构建流程



### 一、运行流程

`webpack` 的运行流程是一个串行的过程，它的工作流程就是将各个插件串联起来

在运行过程中会广播事件，插件只需要监听它所关心的事件，就能加入到这条`webpack`机制中，去改变`webpack`的运作，使得整个系统扩展性良好

从启动到结束会依次执行以下三大步骤：

- 初始化流程：从配置文件和 `Shell` 语句中读取与合并参数，并初始化需要使用的插件和配置插件等执行环境所需要的参数
- 编译构建流程：从 Entry 发出，针对每个 Module 串行调用对应的 Loader 去翻译文件内容，再找到该 Module 依赖的 Module，递归地进行编译处理
- 输出流程：对编译后的 Module 组合成 Chunk，把 Chunk 转换成文件，输出到文件系统

### 二、初始化流程

从配置文件和 `Shell` 语句中读取与合并参数，得出最终的参数

配置文件默认下为`webpack.config.js`，也或者通过命令的形式指定配置文件，主要作用是用于激活`webpack`的加载项和插件

关于文件配置内容分析，如下注释：

```js
var path = require('path');
var node_modules = path.resolve(__dirname, 'node_modules');
var pathToReact = path.resolve(node_modules, 'react/dist/react.min.js');

module.exports = {
  // 入口文件，是模块构建的起点，同时每一个入口文件对应最后生成的一个 chunk。
  entry: './path/to/my/entry/file.js'，
  // 文件路径指向(可加快打包过程)。
  resolve: {
    alias: {
      'react': pathToReact
    }
  },
  // 生成文件，是模块构建的终点，包括输出文件与输出路径。
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: '[name].js'
  },
  // 这里配置了处理各模块的 loader ，包括 css 预处理 loader ，es6 编译 loader，图片处理 loader。
  module: {
    loaders: [
      {
        test: /\.js$/,
        loader: 'babel',
        query: {
          presets: ['es2015', 'react']
        }
      }
    ],
    noParse: [pathToReact]
  },
  // webpack 各插件对象，在 webpack 的事件流中执行对应的方法。
  plugins: [
    new webpack.HotModuleReplacementPlugin()
  ]
};
// webpack 将 `webpack.config.js` 中的各个配置项拷贝到 `options` 对象中，并加载用户配置的 `plugins`
```

完成上述步骤之后，则开始初始化`Compiler`编译对象，该对象掌控者`webpack`声明周期，不执行具体的任务，只是进行一些调度工作

```js
class Compiler extends Tapable {
    constructor(context) {
        super();
        this.hooks = {
            beforeCompile: new AsyncSeriesHook(["params"]),
            compile: new SyncHook(["params"]),
            afterCompile: new AsyncSeriesHook(["compilation"]),
            make: new AsyncParallelHook(["compilation"]),
            entryOption: new SyncBailHook(["context", "entry"])
            // 定义了很多不同类型的钩子
        };
        // ...
    }
}

function webpack(options) {
  var compiler = new Compiler();
  ...// 检查options,若watch字段为true,则开启watch线程
  return compiler;
}
...
```

`Compiler` 对象继承自 `Tapable`，初始化时定义了很多钩子函数



### 三、编译构建流程

根据配置中的 `entry` 找出所有的入口文件

```js
module.exports = {
  entry: './src/file.js'
}
```

初始化完成后会调用`Compiler`的`run`来真正启动`webpack`编译构建流程，主要流程如下：

- `compile` 开始编译
- `make` 从入口点分析模块及其依赖的模块，创建这些模块对象
- `build-module` 构建模块
- `seal` 封装构建结果
- `emit` 把各个chunk输出到结果文件



### 四、**compile 编译**

执行了`run`方法后，首先会触发`compile`，主要是构建一个`Compilation`对象

该对象是编译阶段的主要执行者，主要会依次下述流程：执行模块创建、依赖收集、分块、打包等主要任务的对象

#### make 编译模块

当完成了上述的`compilation`对象后，就开始从`Entry`入口文件开始读取，主要执行`_addModuleChain()`函数，如下：

```js
_addModuleChain(context, dependency, onModule, callback) {
   ...
   // 根据依赖查找对应的工厂函数
   const Dep = /** @type {DepConstructor} */ (dependency.constructor);
   const moduleFactory = this.dependencyFactories.get(Dep);
   
   // 调用工厂函数NormalModuleFactory的create来生成一个空的NormalModule对象
   moduleFactory.create({
       dependencies: [dependency]
       ...
   }, (err, module) => {
       ...
       const afterBuild = () => {
        this.processModuleDependencies(module, err => {
         if (err) return callback(err);
         callback(null, module);
           });
    };
       
       this.buildModule(module, false, null, null, err => {
           ...
           afterBuild();
       })
   })
}
```

过程如下：

`_addModuleChain`中接收参数`dependency`传入的入口依赖，使用对应的工厂函数`NormalModuleFactory.create`方法生成一个空的`module`对象

回调中会把此`module`存入`compilation.modules`对象和`dependencies.module`对象中，由于是入口文件，也会存入`compilation.entries`中

随后执行`buildModule`进入真正的构建模块`module`内容的过程

#### 五、build module 完成模块编译

这里主要调用配置的`loaders`，将我们的模块转成标准的`JS`模块

在用`Loader` 对一个模块转换完后，使用 `acorn` 解析转换后的内容，输出对应的抽象语法树（`AST`），以方便 `Webpack`后面对代码的分析

从配置的入口模块开始，分析其 `AST`，当遇到`require`等导入其它模块语句时，便将其加入到依赖的模块列表，同时对新找出的依赖模块递归分析，最终搞清所有模块的依赖关系

### 六、输出流程

#### seal 输出资源

`seal`方法主要是要生成`chunks`，对`chunks`进行一系列的优化操作，并生成要输出的代码

`webpack` 中的 `chunk` ，可以理解为配置在 `entry` 中的模块，或者是动态引入的模块

根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 `Chunk`，再把每个 `Chunk` 转换成一个单独的文件加入到输出列表

#### emit 输出完成

在确定好输出内容后，根据配置确定输出的路径和文件名

```
output: {
    path: path.resolve(__dirname, 'build'),
        filename: '[name].js'
}
```

在 `Compiler` 开始生成文件前，钩子 `emit` 会被执行，这是我们修改最终文件的最后一个机会

从而`webpack`整个打包过程则结束了

### 小结

![小结](../images/Webpack%E7%9B%B8%E5%85%B3/webpack_end.png)


## Webpack相关


### **webpack的生命周期及钩子**

compiler 对象包含了Webpack 环境所有的的配置信息。

这个对象在启动 webpack 时被一次性建立，并配置好所有可操作的设置，包括 **options**，**loader** 和 **plugin**。当在 webpack 环境中应用一个插件时，插件将收到此 compiler 对象的引用。可以使用它来访问 webpack 的主环境。

compilation对象包含了当前的模块资源、编译生成资源、变化的文件等。

当运行webpack 开发环境中间件时，每当检测到一个文件变化，就会创建一个新的 compilation，从而生成一组新的编译资源。compilation 对象也提供了很多关键时机的回调，以供插件做自定义处理时选择使用。

compiler代表了整个webpack从启动到关闭的生命周期，而compilation 只是代表了一次新的编译过程。



### **webpack** **编译过程**

Webpack 的编译流程是一个串行的过程，从启动到结束会依次执行以下流程：

1. **初始化参数**：从配置文件和 Shell 语句中读取与合并参数，得出最终的参数；
2. **开始编译**：用上一步得到的参数初始化 Compiler 对象，加载所有配置的插件，执行对象的 run方法开始执行编译；
3. **确定入口**：根据配置中的 entry 找出所有的入口文件；
4. **编译模块**：从入口文件出发，调用所有配置的 Loader 对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理；
5. **完成模块编译**：在经过第4步使用 Loader 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系；
6. **输出资源**：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的Chunk，再把每个 Chunk 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会；
7. **输出完成**：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统。



### **优化项目的**webpack打包编译过程

> ​	**1. 构建打点**：构建过程中，每一个Loader 和 Plugin 的执行时长，在编译 JS、CSS 的 Loader 以及对这两类代码执行压缩操作的 Plugin上消耗时长 。一款工具：speed-measure-webpack-plugin
>
> ​	**2. 缓存**：大部分 Loader 都提供了cache 配置项。cache-loader ，将 loader 的编译结果写入硬盘缓存
>
> ​	**3. 多核编译**，happypack项目接入多核编译，理解为happypack 将编译工作灌满所有线程
>
> ​	**4. 抽离**，webpack-dll-plugin 将这些静态依赖从每一次的构建逻辑中抽离出去，静态依赖单独打包，Externals将不需要打包的静态资源从构建逻辑中剔除出去，使用CDN 引用
>
> ​	**5.** **tree-shaking**，虽然依赖了某个模块，但其实只使用其中的某些功能。通过 tree-shaking，将没有使用的模块剔除，来达到删除无用代码的目的。



### **首屏加载优化**

> **路由懒加载**：改为用import引用，以函数的形式动态引入，可以把各自的路由文件分别打包，只有在解析给定的路由时，才会下载路由组件；
>
> **element-ui按需加载**：引用实际上用到的组件 ；
>
> **组件重复打包**：CommonsChunkPlugin配置来拆包，把使用2次及以上的包抽离出来，放进公共依赖文件，首页也有复用的组件，也会下载这个公共依赖文件；
>
> **gzip**: 拆完包之后，再用gzip做一下压缩，关闭sourcemap。
>
> **UglifyJsPlugin:**  生产环境，压缩混淆代码，移除console代码
>
> CDN部署静态资源：静态请求打在nginx时，将获取静态资源的地址进行重定向CDN内容分发网络
>
> **移动端首屏**加载可以使用**骨架屏**，自定义**loading**，首页单独做**服务端渲染**。
>
> 如何进行前端性能优化(21种优化+7种定位方式)



### **webpack** **热更新机制**

热更新流程总结:

> - 启动本地server，让浏览器可以请求本地的**静态资源**
> - 页面首次打开后，服务端与客户端通过 websocket建立通信渠道，把下一次的 hash 返回前端
> - 客户端获取到hash，这个hash将作为下一次请求服务端 hot-update.js 和 hot-update.json的hash
> - 修改页面代码后，Webpack 监听到文件修改后，开始编译，编译完成后，发送 build 消息给客户端
> - 客户端获取到hash，成功后客户端构造hot-update.js script链接，然后插入主文档
> - hot-update.js 插入成功后，执行hotAPI 的 createRecord 和 reload方法，获取到 Vue 组件的 render方法，重新 render 组件， 继而实现 UI 无刷新更新。



### **webpack的 loader和plugin介绍，css-loader，style-loader的区别**

> **loader** 它就是一个转换器，将A文件进行编译形成B文件，
>
> **plugin** ，它就是一个扩展器，来操作的是文件，针对是loader结束后，webpack打包的整个过程，它并不直接操作文件，会监听webpack打包过程中的某些节点（run, build-module, program）
>
> **Babel** 能把ES6/ES7的代码转化成指定浏览器能支持的代码。
>
> css-loader 的作用是把 css文件进行转码style-loader: 使用 < style > 将css-loader内部样式注入到我们的HTML页面
>
> 先使用 css-loader转码，然后再使用 style-loader插入到文件



### **[如何编写一个 Webpack Plugin](https://segmentfault.com/a/1190000037513682)？**

webpack 插件的组成：

> - 一个 JS 命名函数或一个类（可以想下我们平时使用插件就是 new XXXPlugin()的方式）
> - 在插件类/函数的 (prototype) 上定义一个 apply 方法。
> - 通过 apply 函数中传入 compiler 并插入指定的事件钩子，在钩子回调中取到 compilation 对象
> - 通过 compilation 处理 webpack 内部特定的实例数据
> - 如果是插件是异步的，在插件的逻辑编写完后调用 webpack 提供的 callback



### Proxy

> webpack Proxy , 即wepack提供的代理服务，将客户端发送的请求转发给其他服务器。目的是为了解决开发模式下的跨域问题。想要实现代理首先需要一个中间服务器，webpack中提供服务器的工具为webpack-dev-server。webpack-dev-server是webpack官方推出的一款开发工具，将自动编译和自动刷新浏览器等一系列对开发友好的功能集成在了一起。【只适用在开发阶段】

在webpack配置对象属性中通过devServer属性提供，配置如下

```js
// ./webpack.config.js
const path = require('path')

module.exports = {
    // ...
    devServer: {
        contentBase: path.join(__dirname, 'dist'),
        compress: true,
        port: 8080,
        proxy: {
          '/api': {
            target: 'http://www.baidu.com/',
            pathRewrite: {'^/api' : ''},//// 如果接口本身没有/api需要通过pathRewrite来重写了地址,这里把/api转成''
            changeOrigin: true,     // target是域名的话，需要这个参数，
            secure: false,          // 设置支持https协议的代理
          },
        }
        // ...
    }
}
```

devServer里面proxy则是关于代理的配置，该属性为对象的形式，对象中每一个属性即一个代理的规则匹配。

>target：代理的API地址，就是需要跨域的API地址。
>pathRewrite：路径重写，也就是说会修改最终请求的API路径。
>secure：默认情况下不接收转发到https的服务器上，如果希望支持，可设置为false
>changOrigin：默认是false：请求头中host仍然是浏览器发送过来的host；如果设置成true：发送请求头中host会设置成target

proxy工作原理实质上是利用http-proxy-middleware这个http代理中间件，实现请求转发给其他服务器

```js
const express = require('express');
const proxy = require('http-proxy-middleware');

const app = express();

app.use('/api', proxy({target: 'http://www.example.org', changeOrigin: true}));
app.listen(8080);

// http://localhost:8080/api/foo/bar -> http://www.example.org/api/foo/bar
```

总的来说就是：

在开发阶段，webpack-dev-server 会启动一个本地开发服务器，所以我们的应用在开发阶段是独立运行在 localhost 的一个端口上，而后端服务又是运行在另外一个地址上；

所以在开发阶段中，由于浏览器同源策略的原因，当本地访问后端就会出现跨域请求的问题。

通过设置webpack proxy实现代理请求后，相当于浏览器与服务端添加一个代理者

当本地发送请求的时候，代理服务器响应该请求，并将请求转发到目标服务器，目标服务器响应数据后再将数据返回给代理服务器，最终由代理服务器将数据响应给本地。

[Webpack相关试题](https://juejin.cn/post/6844904094281236487)





## 参考文献

- [面试官：webpack原理都不会？](https://github.com/Cosen95/blog/issues/48)
- [细说 webpack 之流程篇 ](https://developer.aliyun.com/article/61047)
- [「吐血整理」再来一打Webpack面试题](https://juejin.cn/post/6844904094281236487)