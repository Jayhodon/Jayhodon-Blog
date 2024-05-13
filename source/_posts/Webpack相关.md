---


title: Webpack相关

date: 2023-04-19 18:45:56

tags: [前端,webpack]

categories: [前端,个人知识库]

summary: '一篇针对WebPack的手作指北，用于了解、记录、剖析相关原理知识点。'
---


> ​		**webpack** 是一个用于现代 JavaScript 应用程序的 *静态模块打包工具*。当 webpack 处理应用程序时，它会在内部构建一个依赖图，此依赖图对应映射到项目所需的每个模块，并生成一个或多个 *bundle*。



### Webpack构建流程



webpack 整个庞大的体系大致可以抽象为三方面的知识：

> 1. **构建的核心流程**
> 2. **loader 的作用**
> 3. **plugin 架构与常用套路**

三者协作构成 webpack 的主体框架 ：

![主题架构](../images/Webpack%E7%9B%B8%E5%85%B3/webpack_arch.png)

​	

#### 核心流程解析

​		首先，我们要理解一个点，Webpack 最核心的功能：

​		也就是将各种类型的资源，包括图片、css、js等，转译、组合、拼接、生成 JS 格式的 bundler 文件。官网首页的动画很形象地表达了这一点：

![bundler](../images/Webpack%E7%9B%B8%E5%85%B3//webpack_bundle.png)

这个过程核心完成了 **内容转换 + 资源合并** 两种功能，实现上包含三个阶段：

1. 初始化阶段：

2. 1. **初始化参数**：从配置文件、 配置对象、Shell 参数中读取，与默认配置结合得出最终的参数
   2. **创建编译器对象**：用上一步得到的参数创建 `Compiler` 对象
   3. **初始化编译环境**：包括注入内置插件、注册各种模块工厂、初始化 RuleSet 集合、加载配置的插件等
   4. **开始编译**：执行 `compiler` 对象的 `run` 方法
   5. **确定入口**：根据配置中的 `entry` 找出所有的入口文件，调用 `compilition.addEntry` 将入口文件转换为 `dependence` 对象

3. 构建阶段：

4. 1. **编译模块(make)**：根据 `entry` 对应的 `dependence` 创建 `module` 对象，调用 `loader` 将模块转译为标准 JS 内容，调用 JS 解释器将内容转换为 AST 对象，从中找出该模块依赖的模块，再 递归 本步骤直到所有入口依赖的文件都经过了本步骤的处理
   2. **完成模块编译**：上一步递归处理所有能触达到的模块后，得到了每个模块被翻译后的内容以及它们之间的 **依赖关系图**

5. 生成阶段：

6. 1. **输出资源(seal)**：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 `Chunk`，再把每个 `Chunk` 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会
   2. **写入文件系统(emitAssets)**：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统

​	

​		单次构建过程自上而下按顺序执行，下面会展开聊聊细节，在此之前，对上述提及的各类技术名词不太熟悉的同学，可以先看看简介：

> - `Entry`：编译入口，webpack 编译的起点
> - `Compiler`：编译管理器，webpack 启动后会创建 `compiler` 对象，该对象一直存活知道结束退出
> - `Compilation`：单次编辑过程的管理器，比如 `watch = true` 时，运行过程中只有一个 `compiler` 但每次文件变更触发重新编译时，都会创建一个新的 `compilation` 对象
> - `Dependence`：依赖对象，webpack 基于该类型记录模块间依赖关系
> - `Module`：webpack 内部所有资源都会以“module”对象形式存在，所有关于资源的操作、转译、合并都是以 “module” 为基本单位进行的
> - `Chunk`：编译完成准备输出时，webpack 会将 `module` 按特定的规则组织成一个一个的 `chunk`，这些 `chunk` 某种程度上跟最终输出一一对应
> - `Loader`：资源内容转换器，其实就是实现从内容 A 转换 B 的转换器
> - `Plugin`：webpack构建过程中，会在特定的时机广播对应的事件，插件监听这些事件，在特定时间点介入编译过程

​		webpack 编译过程都是围绕着这些关键对象展开的，更详细完整的信息，可以参考 Webpack 知识图谱 。



#### 初始化阶段

##### 基本流程

初始化过程：

![webpack初始化](../images/Webpack%E7%9B%B8%E5%85%B3/webpack_init.png)

>  解 读：
>
> 1. 将 `process.args + webpack.config.js` 合并成用户配置
> 2. 调用 `validateSchema` 校验配置
> 3. 调用 `getNormalizedWebpackOptions + applyWebpackOptionsBaseDefaults` 合并出最终配置
> 4. 创建 `compiler` 对象
> 5. 遍历用户定义的 `plugins` 集合，执行插件的 `apply` 方法
> 6. 调用 `new WebpackOptionsApply().process` 方法，加载各种内置插件



​		主要逻辑集中在 `WebpackOptionsApply` 类，webpack 内置了数百个插件，这些插件并不需要我们手动配置，`WebpackOptionsApply` 会在初始化阶段根据配置内容动态注入对应的插件，包括：

- 注入 `EntryOptionPlugin` 插件，处理 `entry` 配置
- 根据 `devtool` 值判断后续用那个插件处理 `sourcemap`，可选值：`EvalSourceMapDevToolPlugin`、`SourceMapDevToolPlugin`、`EvalDevToolModulePlugin`
- 注入 `RuntimePlugin` ，用于根据代码内容动态注入 webpack 运行时



​		到这里，`compiler` 实例就被创建出来了，相应的环境参数也预设好了，紧接着开始调用 `compiler.compile` 函数：

```js
// 取自 webpack/lib/compiler.js 
compile(callback) {
    const params = this.newCompilationParams();
    this.hooks.beforeCompile.callAsync(params, err => {
      // ...
      const compilation = this.newCompilation(params);
      this.hooks.make.callAsync(compilation, err => {
        // ...
        this.hooks.finishMake.callAsync(compilation, err => {
          // ...
          process.nextTick(() => {
            compilation.finish(err => {
              compilation.seal(err => {...});
            });
          });
        });
      });
    });
  }
```

​		Webpack 架构很灵活，但代价是牺牲了源码的直观性，比如说上面说的初始化流程，从创建 `compiler` 实例到调用 `make` 钩子.

逻辑链路很长：

> - 启动 webpack ，触发 `lib/webpack.js` 文件中 `createCompiler` 方法
> - `createCompiler` 方法内部调用 `WebpackOptionsApply` 插件
> - `WebpackOptionsApply` 定义在 `lib/WebpackOptionsApply.js` 文件，内部根据 `entry` 配置决定注入 `entry` 相关的插件，包括：`DllEntryPlugin`、`DynamicEntryPlugin`、`EntryPlugin`、`PrefetchPlugin`、`ProgressPlugin`、`ContainerPlugin`
> - `Entry` 相关插件，如 `lib/EntryPlugin.js` 的 `EntryPlugin` 监听 `compiler.make` 钩子
> - `lib/compiler.js` 的 `compile` 函数内调用 `this.hooks.make.callAsync`
> - 触发 `EntryPlugin` 的 `make` 回调，在回调中执行 `compilation.addEntry` 函数
> - `compilation.addEntry` 函数内部经过一坨与主流程无关的 `hook` 之后，再调用 `handleModuleCreate` 函数，正式开始构建内容



#### 构建阶段

##### 基本流程

​			构建阶段从 `entry` 开始递归解析资源与资源的依赖，在 `compilation` 对象内逐步构建出 `module` 集合以及 `module` 之间的依赖关系，核心流程：

![webpack构建](../images/Webpack%E7%9B%B8%E5%85%B3/webpack_build.png)

>  构建阶段从入口文件开始：
>
> 1. 调用 `handleModuleCreate` ，根据文件类型构建 `module` 子类
> 2. 调用 loader-runner 仓库的 `runLoaders` 转译 `module` 内容，通常是从各类资源类型转译为 JavaScript 文本
> 3. 调用 acorn 将 JS 文本解析为AST
> 4. 遍历 AST，触发各种钩子
> 5. AST 遍历完毕后，调用 `module.handleParseResult` 处理模块依赖
> 6. 对于 `module` 新增的依赖，调用 `handleModuleCreate` ，控制流回到第一步
> 7. 所有依赖都解析完毕后，构建阶段结束

​	

​		这个过程中数据流 `module => ast => dependences => module` ，先转 AST 再从 AST 找依赖。这就要求 `loaders` 处理完的最后结果必须是可以被 acorn 处理的标准 JavaScript 语法，比如说对于图片，需要从图像二进制转换成类似于 `export default "data:image/png;base64,xxx"` 这类 base64 格式或者 `export default "http://xxx"` 这类 url 格式。

​		`compilation` 按这个流程递归处理，逐步解析出每个模块的内容以及 `module` 依赖关系，后续就可以根据这些内容打包输出。



##### 示例：层级递进

​	假如有如下图所示的文件依赖树：

![示例：层级递进](../images/Webpack%E7%9B%B8%E5%85%B3/webpack_example_1.png)

​		其中 `index.js` 为 `entry` 文件，依赖于 a/b 文件；a 依赖于 c/d 文件。初始化编译环境之后，`EntryPlugin` 根据 `entry` 配置找到 `index.js` 文件，调用 `compilation.addEntry` 函数触发构建流程，构建完毕后内部会生成这样的数据结构：

![第一步](../images/Webpack%E7%9B%B8%E5%85%B3/example_first_step.png)

​		此时得到 `module[index.js]` 的内容以及对应的依赖对象 `dependence[a.js]` 、`dependence[b.js]` 。OK，这就得到下一步的线索：a.js、b.js，根据上面流程图的逻辑继续调用 `module[index.js]` 的 `handleParseResult` 函数，继续处理 a.js、b.js 文件，递归上述流程，进一步得到 a、b 模块：

![第二步](../images/Webpack%E7%9B%B8%E5%85%B3/example_second_step.png)

​		从 a.js 模块中又解析到 c.js/d.js 依赖，于是再再继续调用 `module[a.js]` 的 `handleParseResult` ，再再递归上述流程：

![第三步](../images/Webpack%E7%9B%B8%E5%85%B3/example_third_step.png)

​		到这里解析完所有模块后，发现没有更多新的依赖，就可以继续推进，进入下一步。

##### 总结

> - Webpack 编译过程会将源码解析为 AST 吗？webpack 与 babel 分别实现了什么？
>
>   - 构建阶段会读取源码，解析为 AST 集合。
>
>   - Webpack 读出 AST 之后仅遍历 AST 集合；babel 则对源码做等价转换
>
> - Webpack 编译过程中，如何识别资源对其他资源的依赖？
>
>   - Webpack 遍历 AST 集合过程中，识别 `require/ import` 之类的导入语句，确定模块对其他资源的依赖关系
>
> - 相对于 grant、gulp 等流式构建工具，为什么 webpack 会被认为是新一代的构建工具？
>
>   - Grant、Gulp 仅执行开发者预定义的任务流；而 webpack 则深入处理资源的内容，功能上更强大

- 

#### 生成阶段

##### 基本流程

​		构建阶段围绕 `module` 展开，生成阶段则围绕 `chunks` 展开。经过构建阶段之后，webpack 得到足够的模块内容与模块关系信息，接下来开始生成最终资源了。代码层面，就是开始执行 `compilation.seal` 函数：

```js
// 取自 webpack/lib/compiler.js 
compile(callback) {
    const params = this.newCompilationParams();
    this.hooks.beforeCompile.callAsync(params, err => {
      // ...
      const compilation = this.newCompilation(params);
      this.hooks.make.callAsync(compilation, err => {
        // ...
        this.hooks.finishMake.callAsync(compilation, err => {
          // ...
          process.nextTick(() => {
            compilation.finish(err => {
              **compilation.seal**(err => {...});
            });
          });
        });
      });
    });
  }
```

​		`seal` 原意密封、上锁，我个人理解在 webpack 语境下接近于 **“将模块装进蜜罐”** 。`seal` 函数主要完成从 `module` 到 `chunks` 的转化，核心流程：

![seal](../images/Webpack%E7%9B%B8%E5%85%B3/webpack_seal.png)

>  简单梳理一下：
>
> 1. 构建本次编译的 `ChunkGraph` 对象；
> 2. 遍历 `compilation.modules` 集合，将 `module` 按 `entry/动态引入` 的规则分配给不同的 `Chunk` 对象；
> 3. `compilation.modules` 集合遍历完毕后，得到完整的 `chunks` 集合对象，调用 `createXxxAssets` 方法
> 4. `createXxxAssets` 遍历 `module/chunk` ，调用 `compilation.emitAssets` 方法将 `assets` 信息记录到 `compilation.assets` 对象中
> 5. 触发 `seal` 回调，控制流回到 `compiler` 对象



​		这一步的关键逻辑是将 `module` 按规则组织成 `chunks` ，webpack 内置的 `chunk` 封装规则比较简单：

- `entry` 及 entry 触达到的模块，组合成一个 `chunk`
- 使用动态引入语句引入的模块，各自组合成一个 `chunk`



​		`chunk` 是输出的基本单位，默认情况下这些 `chunks` 与最终输出的资源一一对应，那按上面的规则大致上可以推导出一个 `entry` 会对应打包出一个资源，而通过动态引入语句引入的模块，也对应会打包出相应的资源，我们来看个示例。

##### 示例：多入口打包

假如有这样的配置：

```js
const path = require("path");

module.exports = {
  mode: "development",
  context: path.join(__dirname),
  entry: {
    a: "./src/index-a.js",
    b: "./src/index-b.js",
  },
  output: {
    filename: "[name].js",
    path: path.join(__dirname, "./dist"),
  },
  devtool: false,
  target: "web",
  plugins: [],
};
```

实例配置中有两个入口，对应的文件结构：

![入口文件结构](../images/Webpack%E7%9B%B8%E5%85%B3/entry_file_arch.png)

`index-a` 依赖于c，且动态引入了 e；`index-b` 依赖于 c/d 。根据上面说的规则：

- **`entry` 及entry触达到的模块，组合成一个 chunk**
- **使用动态引入语句引入的模块，各自组合成一个 chunk**

生成的 `chunks` 结构为：

![chunks](../images/Webpack%E7%9B%B8%E5%85%B3/webpack_chunks.png)

​		也就是根据依赖关系，`chunk[a]` 包含了 `index-a/c` 两个模块；`chunk[b]` 包含了 `c/index-b/d` 三个模块；`chunk[e-hash]` 为动态引入 `e` 对应的 chunk。

​		不知道大家注意到没有，`chunk[a]` 与 `chunk[b]` 同时包含了 c，这个问题放到具体业务场景可能就是，一个多页面应用，所有页面都依赖于相同的基础库，那么这些所有页面对应的 `entry` 都会包含有基础库代码，这岂不浪费？为了解决这个问题，webpack 提供了一些插件如 `CommonsChunkPlugin` 、`SplitChunksPlugin`，在基本规则之外进一步优化 `chunks`结构。



##### `SplitChunksPlugin` 的作用

​		`SplitChunksPlugin` 是 webpack 架构高扩展的一个绝好的示例，我们上面说了 webpack 主流程里面是按 `entry / 动态引入` 两种情况组织 `chunks` 的，这必然会引发一些不必要的重复打包，webpack 通过插件的形式解决这个问题。

>  回顾 `compilation.seal` 函数的代码，大致上可以梳理成这么4个步骤：
>
> 1. 遍历 `compilation.modules` ，记录下模块与 `chunk` 关系
> 2. 触发各种模块优化钩子，这一步优化的主要是模块依赖关系
> 3. 遍历 `module` 构建 chunk 集合
> 4. 触发各种优化钩子

![优化流程](../images/Webpack%E7%9B%B8%E5%85%B3/webpack_hook_step.png)

​		上面 1-3 都是预处理 + chunks 默认规则的实现，不在我们讨论范围，这里重点关注第4个步骤触发的 `optimizeChunks` 钩子，这个时候已经跑完主流程的逻辑，得到 `chunks` 集合，`SplitChunksPlugin` 正是使用这个钩子，分析 `chunks` 集合的内容，按配置规则增加一些通用的 chunk ：

```js
module.exports = class SplitChunksPlugin {
  constructor(options = {}) {
    // ...
  }

  _getCacheGroup(cacheGroupSource) {
    // ...
  }

  apply(compiler) {
    // ...
    compiler.hooks.thisCompilation.tap("SplitChunksPlugin", (compilation) => {
      // ...
      compilation.hooks.optimizeChunks.tap(
        {
          name: "SplitChunksPlugin",
          stage: STAGE_ADVANCED,
        },
        (chunks) => {
          // ...
        }
      );
    });
  }
};
```

​		webpack 插件架构的高扩展性，使得整个编译的主流程是可以固化下来的，分支逻辑和细节需求“外包”出去由第三方实现，这套规则架设起了庞大的 webpack 生态。



##### 写入文件系统

​		经过构建阶段后，`compilation` 会获知资源模块的内容与依赖关系，也就知道“输入”是什么；而经过 `seal` 阶段处理后， `compilation` 则获知资源输出的图谱，也就是知道怎么“输出”：哪些模块跟那些模块“绑定”在一起输出到哪里。`seal` 后大致的数据结构：

```js
compilation = {
  // ...
  modules: [
    /* ... */
  ],
  chunks: [
    {
      id: "entry name",
      files: ["output file name"],
      hash: "xxx",
      runtime: "xxx",
      entryPoint: {xxx}
      // ...
    },
    // ...
  ],
};
```

​		`seal` 结束之后，紧接着调用 `compiler.emitAssets` 函数，函数内部调用 `compiler.outputFileSystem.writeFile` 方法将 `assets` 集合写入文件系统。



##### 资源形态流转

​		这里结合**资源形态流转**的角度重新考察整个过程，深理解：

![资源形态流转](../images/Webpack%E7%9B%B8%E5%85%B3/webpack_access.png)

- `compiler.make` 阶段：

- - `entry` 文件以 `dependence` 对象形式加入 `compilation` 的依赖列表，`dependence` 对象记录有 `entry` 的类型、路径等信息
  - 根据 `dependence` 调用对应的工厂函数创建 `module` 对象，之后读入 `module` 对应的文件内容，调用 `loader-runner` 对内容做转化，转化结果若有其它依赖则继续读入依赖资源，重复此过程直到所有依赖均被转化为 `module`

- `compilation.seal` 阶段：

- - 遍历 `module` 集合，根据 `entry` 配置及引入资源的方式，将 `module` 分配到不同的 `chunk`
  - 遍历 `chunk` 集合，调用 `compilation.emitAsset` 方法标记 `chunk` 的输出规则，即转化为 `assets` 集合

- `compiler.emitAssets` 阶段：

- - 将 `assets` 写入文件系统

- 

##### Plugin 解析

​		网上不少资料将 webpack 的插件架构归类为“事件/订阅”模式，我认为这种归纳有失偏颇。订阅模式是一种松耦合架构，发布器只是在特定时机发布事件消息，订阅者并不或者很少与事件直接发生交互。

​		举例来说，我们平常在使用 HTML 事件的时候很多时候只是在这个时机触发业务逻辑，很少调用上下文操作。

​		而 webpack 的钩子体系是一种强耦合架构，它在特定时机触发钩子时会附带上足够的上下文信息，插件定义的钩子回调中，能也只能与这些上下文背后的数据结构、接口交互产生 **side effect**，进而影响到编译状态和后续流程。

学习插件架构，需要理解三个关键问题：

##### What: 什么是插件

从形态上看，插件通常是一个带有 `apply` 函数的类：

```js
class SomePlugin {
    apply(compiler) {
    }
}
```

​		`apply` 函数运行时会得到参数 `compiler` ，以此为起点可以调用 `hook` 对象注册各种钩子回调。

​		例如：`compiler.hooks.make.tapAsync` ，这里面 `make` 是钩子名称，`tapAsync` 定义了钩子的调用方式，webpack 的插件架构基于这种模式构建而成，插件开发者可以使用这种模式在钩子回调中，插入特定代码。

​		webpack 各种内置对象都带有 `hooks` 属性，比如 `compilation` 对象：

```js
class SomePlugin {
    apply(compiler) {
        compiler.hooks.thisCompilation.tap('SomePlugin', (compilation) => {
            compilation.hooks.optimizeChunkAssets.tapAsync('SomePlugin', ()=>{});
        })
    }
}
```

钩子的核心逻辑定义在 Tapable 仓库，内部定义了如下类型的钩子：

```js
const {
        SyncHook,
        SyncBailHook,
        SyncWaterfallHook,
        SyncLoopHook,
        AsyncParallelHook,
        AsyncParallelBailHook,
        AsyncSeriesHook,
        AsyncSeriesBailHook,
        AsyncSeriesWaterfallHook
 } = require("tapable");
```



##### When: 什么时候会触发钩子

​		了解 webpack 插件的基本形态之后，接下来需要弄清楚一个问题：webpack 会在什么时间节点触发什么钩子？这一块我认为是知识量最大的一部分，毕竟源码里面有237个钩子，但官网只介绍了不到100个，且官网对每个钩子的说明都太简短，就我个人而言看完并没有太大收获，所以有必要展开聊一下这个话题。先看几个例子：

- `compiler.hooks.compilation` ：

- - 时机：启动编译创建出 compilation 对象后触发
  - 参数：当前编译的 compilation 对象
  - 示例：很多插件基于此事件获取 compilation 实例

- `compiler.hooks.make`：

- - 时机：正式开始编译时触发
  - 参数：同样是当前编译的 `compilation` 对象
  - 示例：webpack 内置的 `EntryPlugin` 基于此钩子实现 `entry` 模块的初始化

- `compilation.hooks.optimizeChunks` ：

- - 时机：`seal` 函数中，`chunk` 集合构建完毕后触发
  - 参数：`chunks` 集合与 `chunkGroups` 集合
  - 示例：`SplitChunksPlugin` 插件基于此钩子实现 `chunk` 拆分优化

- `compiler.hooks.done`：

- - 时机：编译完成后触发
  - 参数：`stats` 对象，包含编译过程中的各类统计信息
  - 示例：`webpack-bundle-analyzer` 插件基于此钩子实现打包分析



###### 触发时机

​		触发时机与 webpack 工作过程紧密相关，大体上从启动到结束，`compiler` 对象逐次触发如下钩子：

![compiler hook触发流程](../images/Webpack%E7%9B%B8%E5%85%B3/webpack_compiler_hooks.png)

而 `compilation` 对象逐次触发：

![compilation 对象触发流程](../images/Webpack%E7%9B%B8%E5%85%B3/webpack_compilation_obj.png)

​		所以，理解清楚前面说的 webpack 工作的主流程，基本上就可以捋清楚“什么时候会触发什么钩子”。

###### 参数

​		传递参数与具体的钩子强相关，官网对这方面没有做出进一步解释，我的做法是直接在源码里面搜索调用语句。

​		例如对于 `compilation.hooks.optimizeTree` ，可以在 webpack 源码中搜索 `hooks.optimizeTree.call` 关键字，就可以找到调用代码：

```js
// lib/compilation.js#2297
this.hooks.optimizeTree.callAsync(this.chunks, this.modules, err => {
});
```

​		结合代码所在的上下文，可以判断出此时传递的是经过优化的 `chunks` 及 `modules` 集合。

###### 找到示例

​		Webpack 的钩子复杂程度不一，我认为最好的学习方法还是带着目的去查询其他插件中如何使用这些钩子。例如，在 `compilation.seal` 函数内部有 `optimizeModules` 和 `afterOptimizeModules` 这一对看起来很对偶的钩子，`optimizeModules` 从字面上可以理解为用于优化已经编译出的 `modules` ，那 `afterOptimizeModules` 呢？

​	从 webpack 源码中唯一搜索到的用途是 `ProgressPlugin` ，大体上逻辑如下：

```js
compilation.hooks.afterOptimizeModules.intercept({
  name: "ProgressPlugin",
  call() {
    handler(percentage, "sealing", title);
  },
  done() {
    progressReporters.set(compiler, undefined);
    handler(percentage, "sealing", title);
  },
  result() {
    handler(percentage, "sealing", title);
  },
  error() {
    handler(percentage, "sealing", title);
  },
  tap(tap) {
    // p is percentage from 0 to 1
    // args is any number of messages in a hierarchical matter
    progressReporters.set(compilation.compiler, (p, ...args) => {
      handler(percentage, "sealing", title, tap.name, ...args);
    });
    handler(percentage, "sealing", title, tap.name);
  }
});
```

​		基本上可以猜测出，`afterOptimizeModules` 的设计初衷就是用于通知优化行为的结束。

​		`apply` 虽然是一个函数，但是从设计上就只有输入，webpack 不 care 输出，所以在插件中只能通过调用类型实体的各种方法来或者更改实体的配置信息，变更编译行为。例如：

- compilation.addModule ：添加模块，可以在原有的 module 构建规则之外，添加自定义模块
- compilation.emitAsset：直译是“提交资产”，功能可以理解将内容写入到特定路径



##### How: 如何影响编译状态

​		解决上述两个问题之后，我们就能理解“如何将特定逻辑插入 webpack 编译过程”，接下来才是重点 —— 如何影响编译状态？

​		强调一下，webpack 的插件体系与平常所见的 订阅/发布 模式差别很大，是一种非常强耦合的设计，hooks 回调由 webpack 决定何时，以何种方式执行；

​		而在 hooks 回调内部可以通过修改状态、调用上下文 api 等方式对 webpack 产生 **side effect**。

比如，`EntryPlugin` 插件：

```js
class EntryPlugin {
  apply(compiler) {
    compiler.hooks.compilation.tap(
      "EntryPlugin",
      (compilation, { normalModuleFactory }) => {
        compilation.dependencyFactories.set(
          EntryDependency,
          normalModuleFactory
        );
      }
    );

    compiler.hooks.make.tapAsync("EntryPlugin", (compilation, callback) => {
      const { entry, options, context } = this;

      const dep = EntryPlugin.createDependency(entry, options);
      compilation.addEntry(context, dep, options, (err) => {
        callback(err);
      });
    });
  }
}
```

上述代码片段调用了两个影响 `compilation` 对象状态的接口：

- `compilation.dependencyFactories.set`
- `compilation.addEntry`

​		操作的具体含义可以先忽略，这里要理解的重点是，webpack 会将上下文信息以参数或 `this` (compiler 对象) 形式传递给钩子回调，在回调中可以调用上下文对象的方法或者直接修改上下文对象属性的方式，对原定的流程产生 side effect。

​		所以想纯熟地编写插件，除了要理解调用时机，还需要了解我们可以用哪一些api，例如：

- `compilation.addModule`：添加模块，可以在原有的 `module` 构建规则之外，添加自定义模块
- `compilation.emitAsset`：直译是“提交资产”，功能可以理解将内容写入到特定路径
- `compilation.addEntry`：添加入口，功能上与直接定义 `entry` 配置相同
- `module.addError`：添加编译错误信息
- ...



###### Loader 介绍

​		Loader 的作用和实现比较简单，容易理解，所以简单介绍一下就行了。回顾 loader 在编译流程中的生效的位置：

![Loader](../images/Webpack%E7%9B%B8%E5%85%B3/webpack_loader.png)

​		流程图中， `runLoaders` 会调用用户所配置的 loader 集合读取、转译资源，此前的内容可以千奇百怪，但转译之后理论上应该输出标准 JavaScript 文本或者 AST 对象，webpack 才能继续处理模块依赖。

​		理解了这个基本逻辑之后，loader 的职责就比较清晰了，不外乎是将内容 A 转化为内容 B，但是在具体用法层面还挺多讲究的，有 pitch、pre、post、inline 等概念用于应对各种场景。

​		为了帮助理解，这里补充一个示例：Webpack 案例 -- vue-loader 原理分析。



#### 小结



![小结](../images/Webpack%E7%9B%B8%E5%85%B3/webpack_end.png)

#### 附录

##### 源码阅读技巧

- **避重就轻：**挑软柿子捏，比如初始化过程虽然绕，但是相对来说是概念最少、逻辑最清晰的，那从这里入手摸清整个工作过程，可以习得 webpack 的一些通用套路，例如钩子的设计与作用、编码规则、命名习惯、内置插件的加载逻辑等，相当于先入了个门

- **学会调试：**多用 `ndb` 单点调试功能追踪程序的运行，虽然 node 的调试有很多种方法，但是我个人更推荐 `ndb` ，灵活、简单，配合 `debugger` 语句是大杀器

- **理解架构：**某种程度上可以将 webpack 架构简化为 `compiler + compilation + plugins` ，webpack 运行过程中只会有一个 `compiler` ；而每次编译 —— 包括调用 `compiler.run` 函数或者 `watch = true` 时文件发生变更，都会创建一个 `compilation` 对象。理解这三个核心对象的设计、职责、协作，差不多就能理解 webpack 的核心逻辑了

- **抓大放小：** plugin 的关键是“钩子”，我建议战略上重视，战术上忽视！钩子毕竟是 webpack 的关键概念，是整个插件机制的根基，学习 webpack 根本不可能绕过钩子，但是相应的逻辑跳转实在太绕太不直观了，看代码的时候一直揪着这个点的话，复杂性会剧增，我的经验是：

- - 认真看一下 tapable 仓库的文档，或者粗略看一下 `tapable` 的源码，理解同步钩子、异步钩子、promise 钩子、串行钩子、并行钩子等概念，对 `tapable` 提供的事件模型有一个较为精细的认知，这叫战略上重视
  - 遇到不懂的钩子别慌，我的经验我连这个类都不清楚干啥的，要去理解这些钩子实在太难了，不如先略过钩子本身的含义，去看那些插件用到了它，然后到插件哪里去加 `debugger` 语句单点调试，等你缕清后续逻辑的时候，大概率你也知道钩子的含义了，这叫战术上忽视

- **保持好奇心：**学习过程保持旺盛的好奇心和韧性，善于 & 敢于提出问题，然后基于源码和社区资料去总结出自己的答案，问题可能会很多，比如：

- - loader 为什么要设计 pre、pitch、post、inline？
  - `compilation.seal` 函数内部设计了很多优化型的钩子，为什么需要区分的这么细？webpack 设计者对不同钩子有什么预期？
  - 为什么需要那么多 `module` 子类？这些子类分别在什么时候被使用？

##### `Module` 与 `Module` 子类

​		从上文可以看出，webpack 构建阶段的核心流程基本上都围绕着 `module` 展开，相信接触过、用过 Webpack 的读者对 `module` 应该已经有一个感性认知，但是实现上 `module` 的逻辑是非常复杂繁重的。

​		以 webpack@5.26.3 为例，直接或间接继承自 `Module` (`webpack/lib/Module.js` 文件) 的子类有54个：

​		要一个一个捋清楚这些类的作用实在太累了，我们需要抓住本质：`module` 的作用是什么？

​		`module` 是 webpack 资源处理的基本单位，可以认为 webpack 对资源的路径解析、读入、转译、分析、打包输出，所有操作都是围绕着 module 展开的。有很多文章会说 **module = 文件**， 其实这种说法并不准确，比如子类 `AsyncModuleRuntimeModule` 就只是一段内置的代码，是一种资源而不能简单等价于实际文件。

​		Webpack 扩展性很强，包括模块的处理逻辑上，比如说入口文件是一个普通的 js，此时首先创建 NormalModule 对象，在解析 AST 时发现这个文件里还包含了异步加载语句。

​		例如 `requere.ensure` ，那么相应地会创建 `AsyncModuleRuntimeModule` 模块，注入异步加载的模板代码。上面类图的 54 个 module 子类都是为适配各种场景设计的。

## loader



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



### 有哪些常见的Loader？你用过哪些Loader？

Webpack 中有许多常见的 Loader，它们用于处理不同类型的文件并将它们转换为 JavaScript 模块，以供 Webpack 打包。以下是一些常见的 Loader 及其用途：

> 
>
> 1. css-loader
>    - 用途：加载 CSS 文件，并将其转换为 JavaScript 模块。它支持模块化、压缩、文件导入等特性。
>    - 使用经验：通常与 `style-loader` 配合使用，将 CSS 样式插入到 DOM 中。
> 2. style-loader
>    - 用途：将 CSS 代码注入到 JavaScript 中，通过 DOM 操作去加载 CSS。
>    - 使用经验：常常与 `css-loader` 一起使用，形成 `style-loader!css-loader` 的组合，用于在 JavaScript 中直接处理 CSS。
> 3. less-loader和sass-loader
>    - 用途：分别用于将 Less 和 Sass/Scss 文件转换为 CSS 文件。
>    - 使用经验：当项目中使用了 Less 或 Sass 语法时，需要这些 Loader 将它们转换为标准的 CSS，然后再使用 `css-loader` 和 `style-loader` 进行处理。
> 4. file-loader
>    - 用途：把文件输出到一个文件夹中，并在代码中通过相对 URL 去引用输出的文件。
>    - 使用经验：常用于处理图片、字体等文件，确保它们在构建后能够被正确地引用。
> 5. url-loader
>    - 用途：和 `file-loader` 类似，但能在文件很小的情况下以 base64 的方式把文件内容注入到代码中去。
>    - 使用经验：这有助于减少小文件的 HTTP 请求数量，提高页面加载性能。
> 6. babel-loader
>    - 用途：将 ES6、ES7 等现代 JavaScript 语法转换为向后兼容的 JavaScript 版本（如 ES5），以便在旧版浏览器中运行。
>    - 使用经验：几乎在所有的现代前端项目中都会用到，是处理 JavaScript 代码不可或缺的工具。
> 7. eslint-loader
>    - 用途：通过 ESLint 检查 JavaScript 代码，确保代码符合一定的规范和质量标准。
>    - 使用经验：在开发过程中使用，可以帮助开发者发现并修复潜在的代码问题，提高代码质量。
> 8. svg-loader
>    - 用途：将 SVG 图片文件转换为 JavaScript 模块，以便在项目中更方便地使用。
>    - 使用经验：在处理 SVG 图标等资源时，使用 `svg-loader` 可以将 SVG 转换为组件，实现更灵活的图标使用方式。

我个人在项目中使用过上述的大部分 Loader，特别是 `css-loader`、`style-loader`、`less-loader`、`sass-loader`、`file-loader`、`url-loader` 和 `babel-loader`。它们在构建前端项目时提供了很大的便利性和灵活性。



### Webpack 怎么匹配到对应的loader的？

Webpack 通过配置文件（通常是 `webpack.config.js` 或其他类型的配置文件）中的 `module.rules` 数组来匹配和决定使用哪个 loader 来处理特定的文件。每个 `rule` 对象通常包含一个或多个条件（`test`、`include`、`exclude`、`resource`、`issuer` 等）和一个 `use` 数组，其中 `use` 数组定义了应当应用于匹配文件的 loader。

下面是一个简单的例子，展示了如何配置 Webpack 以使用 `babel-loader` 来处理 `.js` 和 `.jsx` 文件：

```javascript
module.exports = {  
  // ...  
  module: {  
    rules: [  
      {  
        test: /\.(js|jsx)$/, // 匹配以 .js 或 .jsx 结尾的文件  
        exclude: /node_modules/, // 排除 node_modules 目录下的文件  
        use: {  
          loader: 'babel-loader', // 使用 babel-loader  
          options: {  
            // 这里可以配置 babel-loader 的选项  
            presets: ['@babel/preset-env', '@babel/preset-react'],  
          },  
        },  
      },  
      // 可以添加更多规则...  
    ],  
  },  
  // ...  
};
```

在这个例子中，`test` 属性是一个正则表达式，用于匹配文件的路径。Webpack 会遍历项目中的所有文件，并检查文件的路径是否与 `test` 属性中的正则表达式匹配。如果匹配，则应用该 `rule` 中的 `use` 数组指定的 loader。

`exclude` 属性是另一个可选的正则表达式，用于排除某些文件。在这个例子中，我们排除了 `node_modules` 目录下的所有文件，因为通常我们不需要对这些文件进行转换。

`use` 属性是一个数组，可以包含一个或多个 loader。在这个例子中，我们只使用了一个 loader（`babel-loader`），但也可以指定多个 loader，它们会按照数组中的顺序从右到左应用（即最后一个 loader 最先执行）。

如果文件匹配了多个 `rule`，Webpack 会按照 `rules` 数组中的顺序应用第一个匹配的 `rule`。因此，通常会将更具体的规则放在数组的前面，以便它们能够优先匹配。

除了 `test`、`include` 和 `exclude` 之外，还可以使用其他条件来匹配文件，如 `resource`（匹配文件的绝对路径）和 `issuer`（匹配引发加载请求的文件路径）。这些条件提供了更细粒度的控制，以便更精确地指定哪些文件应该应用哪些 loader。



### 在实际工程中，配置文件上百行乃是常事，如何保证各个loader按照预想方式工作？

在实际工程中，当 Webpack 配置文件变得庞大且复杂时，确保各个 loader 按照预想方式工作确实是一个挑战。以下是一些建议，可以帮助你管理和维护配置文件，确保 loader 的正常工作：

> 
>
> 1. **模块化配置**：将配置文件拆分成多个小的、可重用的模块。每个模块负责一部分配置，如 loader 配置、插件配置、输出配置等。这样可以使配置文件更加清晰，也更容易维护和扩展。
> 2. **注释和文档**：在配置文件中添加足够的注释和文档，解释每个 loader 的用途、配置选项以及为何选择这些选项。这有助于其他开发者理解你的配置，并在需要时进行修改。
> 3. **使用默认值和预设**：对于常用的 loader 配置，可以使用默认值或预设来减少冗余代码。例如，对于 `babel-loader`，你可以使用 `@babel/preset-env` 预设来自动确定需要哪些转换和插件。
> 4. **验证和测试**：使用 Webpack 的验证工具（如 `webpack-merge` 和 `webpack-cli` 的 `--validate` 选项）来检查配置文件的正确性。此外，编写针对 Webpack 构建的测试用例也是一个好主意，以确保构建过程符合预期。
> 5. **持续集成和持续部署 (CI/CD)**：将 Webpack 构建集成到 CI/CD 流程中，以确保每次代码更改时都会进行构建和测试。这有助于及早发现配置问题或构建失败，并防止它们被合并到主分支中。
> 6. **代码审查**：在提交代码更改之前，让其他开发者对你的配置文件进行代码审查。这可以帮助你发现潜在的问题或改进点，并确保配置文件的质量。
> 7. **学习和保持更新**：Webpack 和相关的 loader、插件都在不断发展和更新。定期查看官方文档、博客文章和社区讨论，了解最新的功能和最佳实践，并将它们应用到你的项目中。
> 8. **使用工具进行性能优化**：对于大型项目，Webpack 的构建性能可能会成为瓶颈。使用工具（如 `webpack-bundle-analyzer`）来分析构建输出，找出性能瓶颈并进行优化。同时，确保你的 loader 和插件都是最新的，并且已经进行了适当的性能优化。

通过遵循这些建议，你可以更好地管理和维护 Webpack 配置文件，确保各个 loader 按照预想方式工作。



### 简单描述一下编写loader的思路？

编写 Webpack loader 的思路主要围绕将某种类型的文件转换为 Webpack 能够理解的模块。以下是一个简单的步骤描述，用于指导你编写 loader：

> 1. 确定需求
>    - 首先，明确你想要处理的文件类型（例如，`.txt` 文件、`.csv` 数据文件、自定义模板文件等）。
>    - 确定这些文件转换后的预期输出格式（通常是 JavaScript 模块）。
> 2. 创建 loader 文件
>    - 创建一个新的 JavaScript 文件，这个文件就是你的 loader。
>    - 确保 loader 文件导出一个函数，这个函数接收源文件的内容作为输入。
> 3. 处理输入内容
>    - 在 loader 函数中，读取传入的源文件内容（可能是一个字符串或 Buffer）。
>    - 根据文件类型，执行必要的解析、转换或操作。
> 4. 生成输出
>    - 将处理后的内容转换为一个 JavaScript 模块。这通常意味着将内容封装在一个 `module.exports` 语句中，以便 Webpack 可以将其视为一个模块。
>    - 如果需要，你还可以添加额外的逻辑来导出多个变量或函数。
> 5. 添加依赖和错误处理
>    - 如果 loader 需要额外的依赖项（如其他 npm 包），请确保在 loader 文件中正确地引入它们。
>    - 添加适当的错误处理逻辑，以便在输入无效或处理过程中发生错误时能够优雅地处理。
> 6. 测试 loader
>    - 编写测试用例来验证 loader 的功能。这可以包括单元测试、集成测试或端到端测试。
>    - 使用模拟输入和期望输出来验证 loader 是否按预期工作。
> 7. 配置 Webpack
>    - 在 Webpack 配置文件中，使用 `module.rules` 数组来指定你的 loader。
>    - 设置 `test` 属性以匹配你想要 loader 处理的文件类型。
>    - 使用 `use` 属性来指定你的 loader（可能需要包含 loader 的路径或名称）。
> 8. 优化和调试
>    - 如果 loader 的性能不佳或存在其他问题，请考虑进行优化和调试。
>    - 使用 Webpack 的性能分析工具（如 `webpack-bundle-analyzer`）来检查 loader 的性能。
>    - 使用 `console.log`、调试器或其他调试工具来跟踪和解决问题。
> 9. 文档和发布
>    - 为你的 loader 编写文档，解释其用途、配置选项和使用方法。
>    - 将 loader 发布到 npm 或其他代码托管平台，以便其他开发人员可以使用它。

请注意，以上步骤是一个简化的概述，具体实现可能会因你的具体需求和 loader 的复杂性而有所不同。在编写 loader 时，建议参考 Webpack 的官方文档和其他现有的 loader 实现，以获取更详细的信息和最佳实践。



### Webpack中babel-loader都做了哪些事情？

在Webpack中，babel-loader的主要作用是**在Webpack打包时，将ES6（以及其他更高级别的JavaScript语法，如ES7、JSX等）的代码转换成ES5版本的代码**。这是为了确保浏览器能够正确执行代码，因为尽管现代浏览器大多支持ES6语法，但仍有一些旧版浏览器或特定环境可能不支持。

babel-loader的工作原理是通过Babel这个JavaScript编译器来实现代码转换的。Babel是一个广泛使用的工具，可以将现代JavaScript代码转换成向后兼容的版本。babel-loader则是Webpack的一个插件，它会在Webpack打包过程中自动使用Babel工具对加载的JavaScript文件进行转换。

除了将ES6代码转换为ES5外，babel-loader还可以用来转换其他现代JavaScript特性，如箭头函数、模板字符串、解构赋值等。为了完成这些转换，babel-loader通常会与一些Babel插件和预设（preset）一起使用，如`@babel/preset-env`、`@babel/preset-react`等。这些插件和预设提供了各种转换规则和选项，可以根据需要进行配置。

在Webpack配置文件中，babel-loader通常会被配置在`module.rules`数组中，通过`test`属性指定需要转换的文件类型（如`.js`、`.jsx`等），然后通过`use`属性指定使用babel-loader进行转换。同时，还可以通过`options`属性为babel-loader传递配置选项，以指定需要使用的Babel插件和预设等。

总的来说，babel-loader在Webpack中扮演了一个重要的角色，它使得开发者能够使用现代JavaScript语法编写代码，同时确保这些代码能够在各种浏览器和环境中正常运行。



## Plugin



### 有哪些常见的Plugin？你用过哪些Plugin？

Webpack 有许多常见的插件，它们各自用于不同的构建和优化任务。以下是一些常见的 Webpack 插件及其用途：

> 
>
> 1. HtmlWebpackPlugin
>
>    ：
>
>    - 用途：简化 HTML 文件的创建，自动引入打包后的 JS、CSS 文件等。
>    - 特性：可以生成一个或多个 HTML 文件，并注入 CSS 和 JS 资源。
>    - 备注：这是我常用的插件之一，因为它能自动处理 HTML 文件的依赖关系。
>
> 2. CleanWebpackPlugin
>
>    ：
>
>    - 用途：在每次构建前清理输出目录（如 `dist` 目录），确保构建结果的纯净性。
>    - 特性：删除指定的文件或文件夹。
>    - 备注：我也经常使用这个插件，因为它可以确保构建过程不受之前构建结果的影响。
>
> 3. MiniCssExtractPlugin
>
>    ：
>
>    - 用途：将 CSS 提取到单独的文件中，而不是嵌入到 JS 文件中。
>    - 特性：支持按需加载和代码分割。
>    - 备注：这个插件在 Webpack 4 之后替代了 `extract-text-webpack-plugin`，是优化 CSS 加载的一个好选择。
>
> 4. TerserWebpackPlugin
>
>    ：
>
>    - 用途：压缩和混淆 JavaScript 代码。
>    - 特性：支持 ES6+ 语法，提供更好的压缩效果。
>    - 备注：在 Webpack 4 之后，`UglifyJsPlugin` 被 `TerserWebpackPlugin` 替代，因为它对 ES6+ 有更好的支持。
>
> 5. DefinePlugin
>
>    ：
>
>    - 用途：在编译时创建配置的全局常量。
>    - 特性：允许你在代码中直接使用这些常量，而不需要通过 `require` 或 `import` 引入。
>    - 备注：这个插件在配置环境变量时非常有用。
>
> 6. CompressionWebpackPlugin
>
>    ：
>
>    - 用途：对生成的资源文件进行压缩，如使用 gzip 或 Brotli 算法。
>    - 特性：可以显著减小文件大小，提高网页加载速度。
>    - 备注：这个插件在构建生产环境的包时非常有用。
>
> 7. CopyWebpackPlugin
>
>    ：
>
>    - 用途：复制文件或目录到构建目录。
>    - 特性：可以复制静态资源文件，如图片、字体等。
>    - 备注：在需要复制非代码文件到构建目录时，这个插件很有用。
>
> 8. ProvidePlugin
>
>    ：
>
>    - 用途：自动加载模块，并在所有模块中可用。
>    - 特性：如上面的例子所示，它可以自动为所有模块提供 jQuery 等库。
>    - 备注：这个插件在需要全局引入某个库时很有用。
>
> 9. BundleAnalyzerPlugin
>
>    ：
>
>    - 用途：可视化 Webpack 打包后的文件大小和组成。
>    - 特性：生成一个交互式的树状图，展示各个模块的大小和依赖关系。
>    - 备注：这个插件在分析和优化打包结果时非常有用。

除了上述插件外，Webpack 还有许多其他插件，用于实现不同的功能和优化任务。我个人使用过上述的大部分插件，特别是 HtmlWebpackPlugin、CleanWebpackPlugin、MiniCssExtractPlugin 和 TerserWebpackPlugin，它们在我的日常开发工作中非常有用。



### **[如何编写一个 Webpack Plugin](https://segmentfault.com/a/1190000037513682)？**

webpack 插件的组成：

> - 一个 JS 命名函数或一个类（可以想下我们平时使用插件就是 new XXXPlugin()的方式）
> - 在插件类/函数的 (prototype) 上定义一个 apply 方法。
> - 通过 apply 函数中传入 compiler 并插入指定的事件钩子，在钩子回调中取到 compilation 对象
> - 通过 compilation 处理 webpack 内部特定的实例数据
> - 如果是插件是异步的，在插件的逻辑编写完后调用 webpack 提供的 callback



## 文件指纹

### 文件指纹是什么？怎么用？

文件指纹是指**文件打包后输出的文件名的后缀**，主要用于版本管理。具体来说，文件指纹的用途如下：

> 1. 验证文件完整性：当你从网络上下载了软件后，想确保此软件没有被人修改过（如添加了木马、病毒、非官方插件），或在下载中被破坏，可以使用文件指纹验证（MD5）技术进行确认。通过计算文件的MD5值（一种常见的文件指纹形式），然后与原始文件的MD5值进行比对，可以判断文件是否完整且未被篡改。
> 2. 加速页面访问：在Web开发中，文件指纹通常用于缓存管理。当一个项目需要发布时，如果某个文件被修改过，它的文件指纹也会相应改变。对于未修改的文件，用户在访问时仍然可以使用浏览器缓存的版本，无需重新加载，从而加速页面访问。

使用文件指纹时，通常需要在构建过程中生成它，并将其添加到文件名中。这样，当文件发生更改时，文件指纹也会发生变化，从而确保浏览器能够加载到最新的文件版本。在Webpack等构建工具中，可以通过配置相应的插件或选项来生成和使用文件指纹。

请注意，虽然文件指纹在验证文件完整性和加速页面访问方面非常有用，但它并不能完全防止恶意攻击。因此，在处理敏感数据时，仍需要采取其他安全措施来确保数据的安全性。



### JS的文件指纹设置

在JavaScript（JS）项目中，文件指纹（通常被称为内容哈希、哈希值或chunkhash）的设置通常与构建工具（如Webpack）相关。文件指纹主要是为了确保当文件内容发生变化时，其文件名也会相应改变，这样可以有效地利用浏览器缓存。以下是如何在Webpack中设置文件指纹的一般步骤：

**1. 使用`[chunkhash]`或`[contenthash]`**

在Webpack的输出配置（`output`）中，你可以使用特定的占位符来生成具有文件指纹的文件名。两种常见的占位符是`[chunkhash]`和`[contenthash]`：

- `[chunkhash]`：基于chunk的内容生成哈希值。当chunk的内容发生变化时，哈希值也会改变。
- `[contenthash]`：基于模块内容生成哈希值。当模块的内容发生变化时，哈希值会改变。

例如：

```javascript
// webpack.config.js  
module.exports = {  
  // ...  
  output: {  
    filename: '[name].[contenthash].js',  
    chunkFilename: '[name].[contenthash].chunk.js',  
    // ...  
  },  
  // ...  
};
```

**2. 配置loader以启用哈希**

对于某些loader（如`css-loader`和`mini-css-extract-plugin`），你可能需要配置它们以在生成的文件名中包含哈希值。这通常是通过在loader的选项中设置`filename`或`outputFilename`来实现的。

**3. 使用缓存头**

为了有效地利用浏览器缓存，你还需要确保服务器正确地设置了缓存头。当文件内容没有变化时，浏览器应该使用缓存的版本，而不是重新从服务器下载。

**4. 清理旧的缓存文件**

由于文件指纹的存在，旧的缓存文件可能会占用不必要的空间。因此，你可能需要定期清理旧的缓存文件。这可以通过在构建过程中添加删除旧文件的步骤来实现，或者使用如`clean-webpack-plugin`之类的插件来自动处理。

**5. 注意事项**

- 当使用文件指纹时，请确保在开发环境（dev）和生产环境（prod）之间保持一致。不要在生产环境中使用没有文件指纹的文件名，因为这可能会导致缓存问题。
- 文件指纹的长度可以根据你的需求进行调整。较长的哈希值可以提供更好的唯一性保证，但也会使文件名更长。较短的哈希值可能会增加哈希冲突的风险，但可以使文件名更简洁。



### CSS的文件指纹设置

在CSS的文件指纹设置中，通常我们是为了确保当CSS文件的内容发生变化时，其文件名也能随之改变，以便有效地利用浏览器缓存。这通常与构建工具（如Webpack）的loader和插件配置相关。以下是如何在Webpack中设置CSS文件指纹的一般步骤：

**1. 使用`mini-css-extract-plugin`**

`mini-css-extract-plugin` 是一个Webpack插件，用于将CSS提取到单独的文件中。它允许你通过配置来设置CSS文件的名称和文件指纹。

首先，你需要在项目中安装这个插件：

```bash
bash复制代码

npm install --save-dev mini-css-extract-plugin
```

然后，在你的Webpack配置文件中引入并使用它：

```javascript
// webpack.config.js  
const MiniCssExtractPlugin = require('mini-css-extract-plugin');  
  
module.exports = {  
  // ...  
  module: {  
    rules: [  
      {  
        test: /\.css$/,  
        use: [  
          {  
            loader: MiniCssExtractPlugin.loader,  
            options: {  
              // 这里可以配置loader的选项，但通常不需要为文件指纹配置  
            },  
          },  
          'css-loader', // 或者其他你需要的CSS预处理loader  
        ],  
      },  
      // ... 其他规则 ...  
    ],  
  },  
  plugins: [  
    new MiniCssExtractPlugin({  
      filename: '[name].[contenthash].css', // 这里设置CSS文件的名称和文件指纹  
      chunkFilename: '[id].[contenthash].css', // 如果CSS在chunk中，可以这样设置  
    }),  
    // ... 其他插件 ...  
  ],  
  // ...  
};
```

**2. 配置其他相关loader**

如果你使用了CSS预处理器（如Sass、Less等），你可能还需要确保它们生成的CSS也带有文件指纹。这通常可以通过确保它们生成的CSS被`mini-css-extract-plugin`捕获来实现。

**3. 清理旧的缓存文件**

和JavaScript文件一样，当CSS文件的内容发生变化时，旧的缓存文件可能会占用不必要的空间。因此，你可能需要定期清理旧的缓存文件。你可以通过添加删除旧文件的步骤或使用如`clean-webpack-plugin`之类的插件来自动处理。

**4. 注意事项**

- 确保在开发环境和生产环境之间使用相同的文件指纹设置，以避免缓存问题。
- 文件指纹的长度可以根据你的需求进行调整。较长的哈希值可以提供更好的唯一性保证，但也会使文件名更长。较短的哈希值可能会增加哈希冲突的风险，但可以使文件名更简洁。
- 如果你在开发环境中使用了热模块替换（HMR），请确保它不会干扰CSS文件的文件指纹设置。

### 图片的文件指纹设置

在Webpack中，为图片设置文件指纹通常是为了确保当图片内容发生变化时，其文件名也能随之改变，以便有效地利用浏览器缓存。这通常与Webpack的loader配置相关，特别是`file-loader`或`url-loader`。

以下是如何在Webpack中设置图片文件指纹的一般步骤：

1. **安装必要的loader**：
   如果你还没有安装`file-loader`或`url-loader`，你需要先安装它们。这两个loader都可以用来处理文件，但`url-loader`在文件大小小于指定限制时，可以将文件转换为Base64编码的URL。

   ```bash
   npm install --save-dev file-loader  
   # 或者  
   npm install --save-dev url-loader
   ```

2. **配置loader**：
   在你的Webpack配置文件中，为处理图片文件的规则添加相应的loader。在loader选项中，你可以设置文件名模板，其中可以包含`[hash]`、`[chunkhash]`或`[contenthash]`来生成文件指纹。

   使用`file-loader`的示例：

   ```javascript
   module.exports = {  
     // ...  
     module: {  
       rules: [  
         {  
           test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,  
           use: [  
             {  
               loader: 'file-loader',  
               options: {  
                 name: '[name].[contenthash:8].[ext]', // 使用contenthash并限制长度为8  
                 outputPath: 'images/', // 输出到dist目录下的images文件夹  
                 publicPath: 'images/', // 引用图片时的公共路径  
               },  
             },  
           ],  
         },  
         // ... 其他规则 ...  
       ],  
     },  
     // ...  
   };
   ```

   使用`url-loader`的示例（与`file-loader`类似，但增加了文件大小限制）：

   ```javascript
   {  
     test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,  
     use: [  
       {  
         loader: 'url-loader',  
         options: {  
           limit: 8192, // 小于8kb的图片转为base64编码  
           name: '[name].[contenthash:8].[ext]',  
           outputPath: 'images/',  
           publicPath: 'images/',  
         },  
       },  
     ],  
   }
   ```

3. **清理旧的缓存文件**：
   和JS、CSS文件一样，当图片内容发生变化时，旧的缓存文件可能会占用不必要的空间。因此，你可能需要定期清理旧的缓存文件。你可以通过添加删除旧文件的步骤或使用如`clean-webpack-plugin`之类的插件来自动处理。

4. **注意事项**：

   - 确保在开发环境和生产环境之间使用相同的文件指纹设置，以避免缓存问题。
   - 文件指纹的长度可以根据你的需求进行调整。较长的哈希值可以提供更好的唯一性保证，但也会使文件名更长。较短的哈希值可能会增加哈希冲突的风险，但可以使文件名更简洁。
   - 如果你的项目中有大量的小图片，使用`url-loader`将它们转换为Base64编码的URL可能会更有效率，因为它们不需要额外的HTTP请求。但是，请注意不要将过大的文件转换为Base64，因为这会增加文件大小并降低性能。



## Babel



### 聊一聊Babel原理吧~

Babel是一个非常流行的JavaScript编译器工具，它的主要作用是将新版本的JavaScript代码转换为旧版本的代码，以便能够在旧版本的浏览器或环境中运行。以下是Babel的工作原理：

> 1. **解析（Parse）**：当Babel接收到源代码时，它会使用一个叫做解析器的工具（如`@babel/parser`）将源代码转换为抽象语法树（AST）。在这个过程中，解析器会识别代码中的语法结构，并将其转换为对应的节点类型。AST是一种数据结构，用于表示源代码的抽象语法结构，树上的每个节点都表示源代码中的一种结构。
> 2. **转换（Transform）**：Babel使用遍历器（如`@babel/traverse`）对AST进行遍历，找到需要转换的节点。在这个过程中，Babel会使用访问者模式（Visitor Pattern）来应用插件对AST进行转换。插件是Babel进行转换操作的核心，它们可以修改、删除或添加AST节点，从而实现代码的转换。例如，一个插件可能将ES6中添加的类属性转换为向后兼容的版本，或将对象展开操作符转换为ES5代码。
> 3. **生成（Generate）**：在转换过程完成后，Babel使用生成器（如`@babel/generator`）将转换后的AST重新生成为可执行的JavaScript代码。生成器还可以创建Source Map映射，以便在开发过程中进行调试。

整个过程中，Babel的配置（如插件和预设）起着至关重要的作用。在`babel.config.js`中设置的一些插件和预设，就是在转换过程中使用的。例如，`@babel/preset-env`是一个常用的预设，它可以根据目标环境自动确定需要转换的ECMAScript特性和polyfill。

总的来说，Babel通过解析、转换和生成三个阶段实现了JavaScript代码的转换，使得开发者可以使用新版本的JavaScript语法和功能，同时确保代码能在旧版本的浏览器或环境中运行。



### Webpack 中 babel 属于什么，以什么样的方式存在?

在Webpack中，Babel主要作为一个JavaScript编译器工具存在，用于将ES6及更新版本的JavaScript代码转换为旧版本的代码（如ES5），以确保这些新特性能在较旧的浏览器或环境中运行。

在Webpack中，Babel通常以`babel-loader`的形式存在，这是一个Webpack的loader，用于在Webpack打包过程中处理JavaScript文件。`babel-loader`会读取源代码，通过Babel进行转换，然后输出转换后的代码。

具体来说，你需要在Webpack的配置文件（通常是`webpack.config.js`）中配置`babel-loader`。这通常包括在`module.rules`数组中添加一个新的规则，指定要处理的文件类型（通常是`.js`或`.jsx`文件），以及使用`babel-loader`作为处理这些文件的loader。

同时，你还需要在项目中安装并配置Babel及其相关插件和预设。例如，`@babel/preset-env`是一个常用的预设，它可以根据目标环境自动确定需要转换的ECMAScript特性和polyfill。

总之，Babel在Webpack中扮演着将新版本的JavaScript代码转换为旧版本代码的角色，以确保代码的兼容性和可运行性。



### 介绍一下core.js

core.js是JavaScript标准库的polyfill（垫片/补丁），它主要用于做兼容处理，以支持最新的ECMAScript标准中新增的特性。

具体来说，core.js的主要功能包括：

> 1. 提供对最新的ECMAScript标准中新增特性的支持，如Promise、Symbol、Proxy、Reflect等。
> 2. 提供对新的数据结构和数据类型的支持，如Map、Set、WeakMap、WeakSet、Symbol、TypedArray等。
> 3. 提供多种方式让开发者方便地处理HTTP请求，例如使用XMLHttpRequest（XHR）对象发送和接收HTTP请求，或使用Promise在请求完成后处理结果。
> 4. 提供多种数据存储方式，例如使用IndexedDB进行键值存储和索引查询，或使用Web SQL支持事务操作和索引查询。
> 5. 提供URL构建工具，可以将JavaScript对象转换为HTTP请求的URL，用于构建动态网页、数据存储、图片处理等任务。

core.js广泛应用于需要向下兼容的新特性应用的场景。例如，在老旧浏览器上启用ES6+的类、箭头函数、模块系统，或者利用Symbol进行元编程，以及借助structuredClone实现深拷贝等等。这个库也可以与构建工具一起使用，确保跨平台项目的代码一致性。

简单来说，core.js就是将新功能的ES API转换为大部分现代浏览器都可以支持运行的API补丁包集合，使得开发者能够更轻松地编写跨浏览器兼容的代码，并提供更好的用户体验。





## Webpack相关



### Webpack中具体要配哪些东西？

在Webpack中，你需要配置一系列的设置来满足你的项目需求。以下是一些常见的Webpack配置项：

> 
>
> 1. Entry（入口）
>
>    ：
>
>    - 指定Webpack打包的入口文件。这通常是你的应用程序的起点，如`index.js`。
>    - 可以配置单入口或多入口，多入口通常用于构建多个独立的应用程序或页面。
>
> 2. Output（出口）
>
>    ：
>
>    - 指定打包后文件的输出位置。
>    - 包括`path`（打包文件的绝对路径）和`filename`（打包后的文件名）。
>
> 3. Mode（模式）
>
>    ：
>
>    - 指定构建模式，可以是`development`（开发模式）或`production`（生产模式）。
>    - 开发模式下不会压缩代码，并启用source map；生产模式下会自动压缩代码并优化输出。
>
> 4. Loaders（加载器）
>
>    ：
>
>    - 用于处理非JavaScript文件（如CSS、图片、字体等）。
>    - 通过配置相应的loader（如`css-loader`、`style-loader`、`file-loader`等）来告诉Webpack如何处理这些文件。
>
> 5. Plugins（插件）
>
>    ：
>
>    - 用于扩展Webpack的功能。
>    - 常见的插件包括`HtmlWebpackPlugin`（用于生成HTML文件并自动注入打包后的JS文件）、`CleanWebpackPlugin`（用于在每次构建前清理输出目录）等。
>
> 6. Resolve（解析）
>
>    ：
>
>    - 配置Webpack如何查找模块。
>    - 可以配置模块解析的根目录、别名（alias）等。
>
> 7. DevServer（开发服务器）
>
>    ：
>
>    - 用于在开发过程中启动一个本地服务器。
>    - 可以配置端口号、代理、热更新（HMR）等。
>
> 8. Optimization（优化）
>
>    ：
>
>    - 用于配置代码优化相关的选项。
>    - 包括分割代码（如使用`SplitChunksPlugin`）、压缩JS代码（如使用`TerserPlugin`）等。
>
> 9. Performance（性能）
>
>    ：
>
>    - 用于设置文件大小限制和性能提示。
>    - 当文件大小超过限制时，Webpack会给出警告或错误提示。
>
> 10. Module.rules（模块规则）
>
>     ：
>
>     - 用于配置如何处理项目中的不同文件类型。
>     - 通过在`rules`数组中定义一系列的匹配条件和对应的loader来处理不同类型的文件。

需要注意的是，以上配置项并非都需要在每个项目中都进行配置，你可以根据项目的实际需求来选择性地配置。同时，Webpack也提供了很多默认的配置选项，你可以通过覆盖这些默认选项来定制你的构建过程。


### **webpack的生命周期及钩子**

​		compiler 对象包含了Webpack 环境所有的的配置信息。

​		这个对象在启动 webpack 时被一次性建立，并配置好所有可操作的设置，包括 **options**，**loader** 和 **plugin**。当在 webpack 环境中应用一个插件时，插件将收到此 compiler 对象的引用。可以使用它来访问 webpack 的主环境。

​		compilation对象包含了当前的模块资源、编译生成资源、变化的文件等。

​		当运行webpack 开发环境中间件时，每当检测到一个文件变化，就会创建一个新的 compilation，从而生成一组新的编译资源。compilation 对象也提供了很多关键时机的回调，以供插件做自定义处理时选择使用。

​		compiler代表了整个webpack从启动到关闭的生命周期，而compilation 只是代表了一次新的编译过程。



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

### **webpack** **热更新机制**

热更新流程总结:

> - 启动本地server，让浏览器可以请求本地的**静态资源**
> - 页面首次打开后，服务端与客户端通过 websocket建立通信渠道，把下一次的 hash 返回前端
> - 客户端获取到hash，这个hash将作为下一次请求服务端 hot-update.js 和 hot-update.json的hash
> - 修改页面代码后，Webpack 监听到文件修改后，开始编译，编译完成后，发送 build 消息给客户端
> - 客户端获取到hash，成功后客户端构造hot-update.js script链接，然后插入主文档
> - hot-update.js 插入成功后，执行hotAPI 的 createRecord 和 reload方法，获取到 Vue 组件的 render方法，重新 render 组件， 继而实现 UI 无刷新更新。

Webpack 的热更新（Hot Module Replacement，简称 HMR）原理主要包括以下几个步骤：

> 
>
> 1. **文件变化监测**：Webpack 使用文件系统通知（如 Node.js 的 `fs.watch` 或 `fs.watchFile`）来监视项目文件的更改。当文件发生变化时，Webpack 会捕获到这些变化。
> 2. **重新编译模块**：一旦 Webpack 检测到文件发生变化，它会重新编译受影响的模块。这个过程包括模块的重新解析、依赖的重建以及代码的重新编译等步骤。
> 3. **构建新模块版本**：重新编译的模块会与先前的版本进行比较，并构建新的模块版本。Webpack 会根据模块的依赖关系，确保只重新编译和构建必要的模块，而不是整个项目。
> 4. **通知更新**：Webpack 通过 WebSocket 或轮询机制将新的模块版本通知给运行时环境（通常是 Webpack Dev Server）。这个通知包含了更新模块的详细信息，如模块的标识符和更新内容。
> 5. **应用更新**：运行时环境接收到新的模块版本后，会将其应用于当前运行的应用程序。这个过程会根据模块的类型和内容进行不同的处理。对于样式类的模块，客户端会直接替换页面上的样式，实现样式的实时更新；对于 JavaScript 模块，客户端会先卸载旧模块，再加载并执行新模块，以此更新页面内容。这种动态的模块加载和替换过程使得页面内容能够在不刷新整个页面的情况下得以更新，从而提供了更加流畅的开发体验。
> 6. **保持应用状态**：在更新页面内容的过程中，Webpack 会努力保持应用程序的当前状态，以确保用户体验的连续性。例如，输入框的输入内容、复选框的选中状态等都会保留下来，不会被重置。

总的来说，Webpack 的热更新原理是通过实时监测文件变化、重新编译模块、构建新模块版本、通知更新以及应用更新等步骤，实现页面内容的实时更新，同时保持应用程序的状态不变。这种技术可以大大提高开发效率，减少不必要的页面刷新和重新加载时间。



### Webpack中的文件监听原理

Webpack中的文件监听原理主要涉及对源代码文件的持续监控，并在检测到文件变化时自动触发构建过程以生成新的输出文件。具体来说，其原理如下：

> 1. **轮询机制**：Webpack使用轮询机制来检查文件的变化。这意味着Webpack会定期询问文件系统，查看指定的文件或目录是否有任何更改。
> 2. **时间戳对比**：Webpack会记录每个文件的最后修改时间戳。在每次轮询时，Webpack会检查文件的当前时间戳是否与之前记录的时间戳不同。如果时间戳发生了变化，说明文件已经被修改。
> 3. **缓存与延迟**：当检测到文件变化时，Webpack并不会立即触发构建过程。相反，它会将文件修改的信息缓存起来，并等待一段时间（这个时间可以通过配置`aggregateTimeout`选项来调整）。这是因为在短时间内可能会有多个文件被连续修改，Webpack通过等待和缓存可以避免不必要的重复构建，从而提高构建效率。
> 4. **批量处理**：在等待期间，如果又有其他文件被修改，Webpack会将这些修改也缓存起来。当等待时间结束后，Webpack会一次性处理所有缓存的修改，并触发构建过程生成新的输出文件。
> 5. **排除特定文件或目录**：Webpack允许你指定不需要监听的文件或目录。这可以通过在`watchOptions`中设置`ignored`选项来实现，支持使用正则表达式进行匹配。这可以进一步提高文件监听的效率和准确性。

总之，Webpack的文件监听原理是通过轮询机制检查文件的变化，并使用时间戳对比和缓存机制来避免不必要的重复构建。同时，通过排除特定文件或目录来提高文件监听的效率和准确性。



### Wepack中的代码分割的本质是什么？

Webpack中的代码分割（Code Splitting）的本质是将一个大型的代码库分割成多个小块（chunks），并在需要时按需加载这些小块。这样做有几个主要的好处：

> 1. **提高性能**：通过按需加载代码，可以减少首次加载时用户需要下载的代码量，从而提高应用的启动速度。当用户与页面进行交互并触发某些功能时，再加载这些功能所需的代码，可以进一步提高应用的响应速度和用户体验。
> 2. **更好的缓存管理**：由于代码被分割成了多个小块，每个小块都可以单独缓存。当用户再次访问应用时，只需要加载那些已经改变的小块，而不是整个代码库。这可以显著提高缓存的利用率和应用的加载速度。
> 3. **代码组织**：代码分割有助于将代码按照功能或页面进行组织，使得代码结构更加清晰和易于维护。

Webpack支持多种方式进行代码分割，包括：

> 1. **基于入口点的分割**：通过配置多个入口点（entry points），Webpack可以为每个入口点生成一个独立的输出文件。这种方式适合将不同的页面或功能拆分成不同的入口点，从而实现基本的代码分割。
> 2. **动态导入（Dynamic Imports）**：Webpack支持ES6的动态`import()`语法，允许在运行时按需加载模块。这是实现更细粒度代码分割的关键技术，非常适合实现懒加载或条件加载。
> 3. **使用SplitChunksPlugin**：Webpack内置了SplitChunksPlugin插件，用于优化和分割公共的依赖模块。该插件可以自动分析出公共的依赖模块，并将它们提取到已有的入口chunk中，或者生成一个新的chunk。

通过这些方式，Webpack可以根据项目的实际需求进行代码分割，从而提高应用的性能和可维护性。



### dev的时候webpack做了什么事情？

在开发（dev）时，Webpack 主要做了以下事情：

> 1. **实时监听**：Webpack 使用其 watch 模式来实时监听源代码文件的变化。当检测到文件被修改时，Webpack 会自动触发构建过程。
> 2. **模块打包**：Webpack 会根据项目的配置文件（如 webpack.config.js），将项目中的各个模块（如 JavaScript、CSS、图片等）打包成浏览器可以识别的格式。这个过程中，Webpack 会处理模块的依赖关系，确保它们按照正确的顺序被加载。
> 3. **代码转换**：对于非 JavaScript 文件（如 CSS、图片等），Webpack 会使用相应的 loader（加载器）进行转换，以便它们可以被 JavaScript 引用。例如，css-loader 可以将 CSS 文件转换为 JavaScript 模块，file-loader 可以将图片文件转换为 URL 或 base64 编码的字符串。
> 4. **插件处理**：Webpack 允许开发者使用插件（plugin）来扩展其功能。在开发过程中，这些插件可以用于执行各种任务，如压缩代码、优化资源加载、生成 HTML 文件等。
> 5. **热模块替换（Hot Module Replacement, HMR）**：当与 webpack-dev-server 一起使用时，Webpack 支持热模块替换功能。这意味着在开发过程中，当源代码文件被修改时，Webpack 可以只替换被修改的模块，而不是重新加载整个页面。这可以大大提高开发效率。
> 6. **提供开发服务器**：webpack-dev-server 是一个基于 Express 的开发服务器，它集成了 Webpack 的打包功能和自动刷新浏览器等功能。这使得开发者可以在本地快速启动一个服务器，进行开发、调试和测试。webpack-dev-server 会在内存中创建一个静态资源服务器，并在启动后自动打开浏览器。当代码被修改时，webpack-dev-server 会自动重新编译和刷新页面。

总的来说，Webpack 在开发过程中主要负责实时监听文件变化、模块打包、代码转换、插件处理和提供开发服务器等任务，以支持快速、高效的前端开发。



### 什么是神奇注释或者魔术注释？

**神奇注释或魔术注释（Magic Comment）是一种特殊的注释，用于在代码中嵌入特定的指令或元数据**。这些注释通常被特定的工具或库识别并处理，以实现一些特殊的功能或效果。

在Webpack中，魔术注释被用于为动态导入的模块指定特定的名字，以方便Webpack在编译过程中进行优化和分割。例如，通过在import语句中添加特定格式的注释，可以为动态导入的模块指定一个chunk名，以便在打包后生成具有特定名称的bundle文件。

请注意，魔术注释并不是一种通用的编程概念，而是特定于某些工具或库的特定功能。因此，在不同的上下文或环境中，魔术注释的具体含义和用法可能会有所不同。



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

​		devServer里面proxy则是关于代理的配置，该属性为对象的形式，对象中每一个属性即一个代理的规则匹配。

>target：代理的API地址，就是需要跨域的API地址。
>pathRewrite：路径重写，也就是说会修改最终请求的API路径。
>secure：默认情况下不接收转发到https的服务器上，如果希望支持，可设置为false
>changOrigin：默认是false：请求头中host仍然是浏览器发送过来的host；如果设置成true：发送请求头中host会设置成target

​		proxy工作原理实质上是利用http-proxy-middleware这个http代理中间件，实现请求转发给其他服务器

```js
const express = require('express');
const proxy = require('http-proxy-middleware');

const app = express();

app.use('/api', proxy({target: 'http://www.example.org', changeOrigin: true}));
app.listen(8080);

// http://localhost:8080/api/foo/bar -> http://www.example.org/api/foo/bar
```

总的来说就是：

​		在开发阶段，webpack-dev-server 会启动一个本地开发服务器，所以我们的应用在开发阶段是独立运行在 localhost 的一个端口上，而后端服务又是运行在另外一个地址上；

​		所以在开发阶段中，由于浏览器同源策略的原因，当本地访问后端就会出现跨域请求的问题。

​		通过设置webpack proxy实现代理请求后，相当于浏览器与服务端添加一个代理者

​		当本地发送请求的时候，代理服务器响应该请求，并将请求转发到目标服务器，目标服务器响应数据后再将数据返回给代理服务器，最终由代理服务器将数据响应给本地。



### 说一说Loader和Plugin的区别？

Loader和Plugin在Webpack中扮演着不同的角色，它们之间的主要区别体现在以下几个方面：

> 1. **功能定位**：
>    - Loader：专注于处理资源的加载和转换。由于Webpack本身只能处理JavaScript模块，Loader的作用就是将其他类型的文件（如CSS、图片、字体等）转换成Webpack能够处理的JavaScript模块。Loader就像一个翻译官，负责将其他类型的资源“翻译”成Webpack能理解的JavaScript代码。
>    - Plugin：负责扩展Webpack的功能。Plugin可以监听Webpack运行过程中的各种事件，并在合适的时机通过Webpack提供的API改变输出结果。Plugin不仅可以处理资源文件，还可以优化构建过程、生成额外的文件等。
> 2. **配置方式**：
>    - Loader：在`module.rules`中配置，作为模块的解析规则。Loader的配置通常是一个数组，数组中的每个元素都是一个对象，指定了需要处理的文件类型、使用的Loader以及传递给Loader的参数等。
>    - Plugin：在`plugins`中单独配置，也是以数组的形式存在。数组中的每个元素都是一个Plugin的实例，参数都通过构造函数传入。
> 3. **执行顺序**：
>    - Loader：Loader的执行顺序是从下到上、从右到左的链式调用。在一个规则中，多个Loader会按照配置的顺序依次执行，上一个Loader的输出会作为下一个Loader的输入。
>    - Plugin：Plugin的执行顺序则取决于Webpack的事件流和Plugin的监听事件。不同的Plugin会监听不同的事件，并在事件触发时执行相应的逻辑。
> 4. **使用场景**：
>    - Loader：通常用于处理资源的加载和转换，如将CSS文件转换成JavaScript模块、将图片转换成Base64编码等。
>    - Plugin：则更多地用于扩展Webpack的功能，如生成HTML文件、压缩代码、优化资源加载等。

总结来说，Loader和Plugin在Webpack中各自扮演着不同的角色，它们共同协作以实现Webpack的强大功能。Loader负责资源的加载和转换，而Plugin则负责扩展Webpack的功能和优化构建过程。



### source map是什么？生产环境怎么用？

**Source Map是一个信息文件，里面储存着位置信息，即存储着代码压缩混淆前后的对应关系**。它是一种数据格式，通常使用.map扩展名，本质上是一个JSON文本文档，其MIME类型也一般设为application/json。

在生产环境下，Source Map的使用需要考虑安全性和性能。一般来说，不建议在生产环境下直接启用Source Map，因为这可能会暴露源代码给潜在的攻击者。但是，如果你确实需要在生产环境下查看报错行数或展示源码，可以采用以下几种方式：

> 1. **使用`nosource-source-map`方式**：这种方式只会将代码定位到多少行，但不会直接跳转源码。这样可以提高代码的安全性，避免遭到攻击。你可以在webpack的配置文件（如webpack.config.js）中设置`devtool: 'nosource-source-map'`来实现这一点。
> 2. **删除`devtool`配置**：相当于关闭了Source Map。在生产环境下，如果你不需要查看源码或报错行数，可以直接删除`devtool`配置。
> 3. **仅在需要时启用Source Map**：你可以考虑只在遇到问题时才启用Source Map，例如在错误日志中提供一个链接，让用户或开发者可以手动下载和查看Source Map文件。

请注意，无论你选择哪种方式，都需要权衡安全性和性能之间的关系。在生产环境下，你应该始终优先考虑安全性。



[Webpack相关试题](https://juejin.cn/post/6844904094281236487)



## Vite



### 介绍一下vite

Vite是一款基于ES模块的前端构建工具，旨在提供快速、高效、简洁的开发体验。以下是关于Vite的详细介绍：

1. **基本概念和特点**：

- Vite是一个下一代前端开发与构建工具，它使用浏览器原生的ES模块作为入口，减少了构建时间。
- Vite提供了开箱即用的配置，同时其插件API和JavaScript API带来了高度的可扩展性，并有完整的类型支持。
- Vite内置了HMR（模块热更新）到Vue.js单文件组件（SFC）和React Fast Refresh中，使得开发者可以实时更新代码并预览修改效果。
- Vite支持多种前端框架和语言，如Vue、React、Angular等，以及TypeScript、CoffeeScript和Sass等前端语言。

1. **主要优势**：

- **快速构建**：Vite使用了优化过的构建器，可以快速地构建和优化应用程序。其构建速度比传统工具如Webpack快得多，因为它避免了静态打包和编译的过程。
- **高效**：Vite的优化过的构建器可以减少构建时间和运行时内存占用。它还支持多种静态资源类型，如JavaScript、CSS、HTML等，可以减少静态资源的加载时间。
- **简洁**：Vite的配置非常简洁，可以轻松地配置和定制。它还提供了一个易于使用的CLI（命令行界面），可以轻松地创建和管理Vite项目。
- **社区支持**：Vite拥有庞大的社区支持，有许多开源项目和插件可供使用，可以满足用户的不同需求。

1. **安装和使用**：

- 你可以使用npm或yarn来安装Vite。安装完成后，你需要创建一个入口文件（如“index.html”），并在其中引入你的JavaScript代码。
- 编写JavaScript代码时，你可以创建Vue、React或其他框架的应用程序，并享受Vite带来的快速构建和实时预览功能。

总的来说，Vite是一个快速、高效、简洁的构建工具，旨在为用户提供一个更好的开发体验，提高开发效率和应用程序性能。



### 解释一下Vite的原理

Vite的原理主要基于现代浏览器的原生ES模块（ESM）加载特性，以及快速热模块替换（HMR）机制。以下是关于Vite原理的详细解释：

> 1. 原生ES模块加载：在开发环境中，Vite利用现代浏览器支持的ES模块特性来处理文件。这意味着它不预先打包文件，而是直接将源文件作为模块发送给浏览器。当浏览器请求一个文件时（比如一个JavaScript模块或Vue组件），Vite仅处理该特定文件，并实时地将其转换成浏览器可理解的格式。这种按需加载避免了传统打包工具在开发环境中的全部重构建过程，减少了初始化加载时间并提高了模块更新的速度。
> 2. 快速热模块替换（HMR）：Vite实现了高效的热模块替换机制。这意味着在开发过程中，当源代码发生变化时，Vite可以仅重新加载发生变化的模块，而不是整个应用程序。这大大提高了开发效率，使得开发者可以更快地看到代码更改的效果。

此外，Vite的工作原理和架构还可以大致分为以下几个部分：

> - 入口文件处理器：负责处理入口文件，将入口文件转换为ES模块。
> - 构建器：负责对ES模块进行构建和优化。
> - 模块缓存：负责将构建好的模块缓存到内存中，以便在应用程序运行时快速加载。
> - 静态资源处理器：负责对静态资源进行处理，如CSS预处理器、图片处理等。

总的来说，Vite通过利用现代浏览器的原生ES模块加载特性和快速热模块替换机制，为用户提供了一个快速、高效、简洁的构建流程，以提高开发效率和应用程序性能。



### Vite 和 Webpack 的区别

Vite和Webpack是两款不同的前端构建工具，它们在多个方面存在显著的区别。以下是它们之间的主要差异：

1. **构建速度和冷启动时间**：

- Vite：由于采用了基于浏览器原生ES模块的开发模式，Vite在开发时可以快速启动应用，减少了冷启动时间。它直接服务源代码，具有极快的冷启动时间（即启动首次打包时）。
- Webpack：需要先进行打包才能启动开发服务器，因此冷启动时间相对较长。它需要先打包代码，转换为浏览器可识别的模块格式，才能实现运行。

1. **按需编译**：

- Vite：可以根据需要动态地编译模块，每个模块都可以独立地进行编译和缓存。这意味着它只需要重新编译修改过的模块，而不是整个应用程序，从而提高了开发效率。
- Webpack：会将所有模块打包到一个文件中，无法实现按需编译。

1. **配置文件**：

- Vite：使用更简单的JSON配置文件，只包含必要的启动信息，配置相对更简单。
- Webpack：使用复杂的JavaScript配置文件，需要配置loader、plugin等组件，配置更加复杂。

1. **热模块替换（HMR）**：

- Vite：可以直接替换JS模块，无需重新加载页面，使得开发过程中的代码更新更加快速和流畅。
- Webpack：在替换模块后需要刷新页面才能使HMR生效，这可能会打断开发流程。

1. **TypeScript支持**：

- Vite：内置支持TypeScript，可以直接导入TS文件。
- Webpack：需要安装相应的loader才能导入TS文件。

1. **生态环境**：

- Webpack：拥有更成熟的生态环境，在社区中拥有广泛的支持和丰富的插件库。
- Vite：虽然已经获得了很多关注，但其生态系统仍然不太完善，正在发展阶段。

1. **功能特性**：

- Webpack：是一个功能更加全面的打包工具，支持各种Loader和插件，可以处理多种类型的文件和资源。
- Vite：设计初衷是专注于开发环境下的快速构建，因此对一些高级特性的支持相对较少。

综上所述，Vite和Webpack各有优劣，选择哪个工具取决于项目的具体需求和开发者的个人偏好。对于需要快速启动和高效开发的小型项目或组件库，Vite可能是一个更好的选择；而对于大型应用的构建和优化，Webpack可能更加适合。



## webpack优化



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



### Webpack常见的优化方案

Webpack常见的优化方案主要包括以下几个方面：

> 
>
> 1. 利用Tree Shaking
>
>    ：
>
>    - Tree Shaking是一个术语，用于描述移除JavaScript上下文中的未引用代码（死代码）。在Webpack中，你可以通过配置`production`模式（该模式下默认启用Tree Shaking）或利用像`terser-webpack-plugin`这样的插件来进一步优化结果。
>
> 2. 优化加载速度
>
>    ：
>
>    - 使用`MiniCssExtractPlugin`将CSS提取到单独的文件中，以避免阻塞JavaScript的加载。
>    - 使用`babel-loader`的缓存机制，以减少不必要的转换操作。
>
> 3. 并行和分布式构建
>
>    ：
>
>    - 利用`thread-loader`或`happypack`等插件将任务分配给多个子进程并行处理，从而加速构建过程。
>    - 在多核CPU的机器上，这可以显著提高构建速度。
>
> 4. 压缩文件体积
>
>    ：
>
>    - 使用`terser-webpack-plugin`来压缩JavaScript代码。
>    - 使用`cssnano`等工具来压缩CSS代码。
>    - 对于图片和其他资源，可以使用像`image-webpack-loader`或`url-loader`这样的加载器来优化。
>
> 5. 使用缓存
>
>    ：
>
>    - 配置Webpack的缓存功能，如`cache-loader`，以便在构建过程中重用未更改的文件，从而加快构建速度。
>    - 利用`hard-source-webpack-plugin`这样的插件来实现持久化缓存，进一步提高构建性能。
>
> 6. 代码分割
>
>    ：
>
>    - 利用Webpack的代码分割功能，将代码拆分成多个bundle，以便按需加载。
>    - 这可以通过配置`optimization.splitChunks`选项来实现。
>
> 7. 优化输出
>
>    ：
>
>    - 使用`optimization.minimize`选项来开启代码压缩。
>    - 利用`optimization.runtimeChunk`选项来将运行时代码拆分为单独的bundle。
>    - 配置`output.filename`和`output.chunkFilename`选项来生成具有唯一哈希的文件名，以便实现长期缓存。
>
> 8. 使用DLL Plugin
>
>    ：
>
>    - DLL Plugin可以将一些不经常变动的库或框架代码与业务代码分开打包，从而加速构建过程。
>
> 9. 开发服务器配置
>
>    ：
>
>    - 配置`devServer`选项来优化开发体验，如启用热更新（HMR）、设置代理等。
>
> 10. 外部化（Externals）
>
>     ：
>
>     - 通过配置`externals`选项，可以将一些已经在运行环境中存在的库（如jQuery、React等）排除在构建过程之外，从而减少打包后的文件体积。
>
> 11. 使用Webpack Bundle Analyzer
>
>     ：
>
>     - 通过这个插件，你可以分析打包后的文件，找出哪些模块占用了较大的体积，从而有针对性地进行优化。
>
> 

请注意，这些优化方案应根据具体项目需求和场景灵活应用。不同的项目可能需要采用不同的优化策略。



### Webpack优化之减少打包时间

Webpack优化以减少打包时间，通常涉及以下几个方面的策略：

> 
>
> 1. **使用最新版本的Webpack**：
>    - 确保你正在使用最新版本的Webpack和相关插件，因为新版本通常包含性能改进和错误修复。
>
> 2. **减少打包作用域**：
>    - 优化loader的配置，减少不必要的文件转换。通过`include`和`exclude`属性来指定loader应该处理哪些文件。
>    - 例如，如果你只需要转换src目录下的JS文件，你可以配置babel-loader来只处理这个目录下的文件。
>
> 3. **启用缓存**：
>    - 使用Webpack的内置缓存机制或配置loader（如babel-loader）的缓存选项，以避免重复转换未更改的文件。
>    - 例如，设置babel-loader的`cacheDirectory`选项为`true`可以启用缓存。
>
> 4. **使用Tree Shaking**：
>    - 在Webpack配置中启用Tree Shaking功能，以消除未使用的代码。这可以通过在`production`模式下运行Webpack来实现，因为该模式下默认启用Tree Shaking。
>
> 5. **代码分割**：
>    - 使用Webpack的代码分割功能将代码拆分成多个bundle，以便按需加载。这可以减少初始加载时间，因为用户只需加载他们当前需要的代码。
>
> 6. **优化Loader和插件**：
>    - 选择性能良好的Loader和插件，并通过配置参数和选项来提高其性能。
>    - 例如，使用`hard-source-webpack-plugin`或`cache-loader`来缓存loader的结果。
>
> 7. **并行构建**：
>    - 使用Webpack的`thread-loader`或`happypack`插件将任务分发给多个子进程并行处理，以提高构建速度。
>
> 8. **优化文件体积**：
>    - 虽然这主要关注减少打包后的文件大小，但压缩代码和资源（如使用`terser-webpack-plugin`压缩JS代码，使用`cssnano`压缩CSS代码）也可以间接地减少打包时间，因为它们减少了需要处理的文件大小。
>
> 9. **使用DLL Plugin**：
>    - 将一些不经常变动的库或框架代码与业务代码分开打包，这样当业务代码发生变化时，不需要重新打包这些库或框架。
>
> 10. **分析和监控**：
>     - 使用Webpack Bundle Analyzer来分析和可视化打包后的文件，找出哪些模块占用了较大的体积和时间，从而有针对性地进行优化。
>     - 监控构建过程，查找可能的瓶颈和性能问题。

请注意，不同的项目可能需要采用不同的优化策略。建议根据你的具体项目需求和场景来选择合适的优化方案。



### Webpack优化之减少打包体积

Webpack 优化以减少打包体积的策略主要包括以下几个方面：

> 1. 代码分割 (Code Splitting)
>    - 通过配置 Webpack 的代码分割功能，将项目代码分割成多个块（chunks），并在需要时按需加载。这可以通过 `import()` 语法动态导入模块或使用 Webpack 的 `SplitChunksPlugin` 插件来实现。
> 2. Tree Shaking
>    - Tree Shaking 可以移除 JavaScript 上下文中的未引用代码（死代码）。在 Webpack 中，可以通过启用 `production` 模式下的默认 Tree Shaking 功能或使用 `sideEffects` 属性来进一步指定哪些文件或模块具有副作用，从而避免不必要的代码被打包。
> 3. 压缩和优化代码
>    - 使用代码压缩插件，如 `TerserWebpackPlugin`（用于压缩 JavaScript 代码）和 `OptimizeCSSAssetsPlugin` 或 `cssnano`（用于压缩 CSS 代码）。这些插件可以删除注释、缩短变量名、合并重复的代码等，从而减小文件体积。
> 4. 移除不必要的插件和加载器
>    - 检查 Webpack 配置，移除不再需要或从未使用的插件和加载器，以减少构建过程中的不必要的处理。
> 5. 使用作用域内的 CSS
>    - 使用如 `MiniCssExtractPlugin` 这样的插件将 CSS 提取到单独的文件中，并通过 `<link>` 标签引入。这样可以利用浏览器的并行加载能力，并且只对实际使用的 CSS 进行加载。
> 6. 优化图片和其他资源
>    - 使用适当的图片加载器（如 `file-loader` 或 `url-loader`）和图片优化插件（如 `image-webpack-loader`）来压缩和优化图片。此外，还可以使用 SVG Sprite 插件来合并 SVG 图标，减少 HTTP 请求。
> 7. 利用缓存
>    - 使用 Webpack 的内置缓存机制或第三方插件（如 `hard-source-webpack-plugin`）来缓存 loader 的结果，避免重复处理未更改的文件。
> 8. 外部化依赖
>    - 将一些库或框架通过 CDN 引入，而不是将它们打包到项目中。这可以通过 Webpack 的 `externals` 配置选项来实现。
> 9. 使用 Webpack Bundle Analyzer
>    - 这个插件可以帮助你分析打包后的文件，找出哪些模块或依赖“贡献”出了最多的体积，从而有针对性地进行优化。
> 10. 优化 Webpack 配置
>     - 仔细检查和优化 Webpack 的配置文件（如 `webpack.config.js`），确保所有配置选项都是必要的，并且没有不必要的重复或冗余。

请注意，不同的项目可能需要采用不同的优化策略。在实施任何优化之前，最好先分析项目的具体需求和瓶颈，然后有针对性地进行优化。



### 如何对bundle体积进行监控和分析？

对Webpack打包后的bundle体积进行监控和分析，可以通过以下几种方法：

> 1. **使用Webpack Bundle Analyzer**
>
>    Webpack Bundle Analyzer是一个可视化的Webpack包分析工具，它可以将应用程序的bundle以树状图的形式展示，并显示每个模块的大小、包含的依赖关系等信息。通过这个工具，你可以快速定位到哪些模块或依赖项占用了较大的体积，从而进行针对性的优化。
>
>    使用方法：
>
>    - 首先，安装Webpack Bundle Analyzer：`npm install --save-dev webpack-bundle-analyzer`
>    - 在Webpack配置文件中（通常是`webpack.config.js`）引入并使用`BundleAnalyzerPlugin`插件。
>    - 运行Webpack构建命令，并在构建完成后，Webpack Bundle Analyzer将自动启动一个服务器，在浏览器中展示bundle分析的结果。
>
> 2. **使用Bundlephobia**
>
>    Bundlephobia是一个在线工具，它可以帮助你分析npm包在不同环境下的真实大小和性能消耗。你可以将需要使用的npm包搜索到Bundlephobia上，并进行大小和依赖关系的分析，以便进行选择和优化。
>
>    使用方法：
>
>    - 访问Bundlephobia的官方网站（https://bundlephobia.com/）
>    - 在搜索框中输入npm包的名称，点击搜索按钮。
>    - 查看npm包的大小、依赖关系等信息，并根据需要进行选择和优化。
>
> 3. **手动分析Webpack打包后的文件**
>
>    你也可以通过手动分析Webpack打包后的文件来监控bundle体积。Webpack会将打包后的文件输出到指定的目录中，你可以通过查看这些文件的大小和内容来了解bundle的体积和组成。
>
>    具体步骤：
>
>    - 找到Webpack打包后的文件输出目录（通常在`dist`或`build`目录下）。
>    - 使用文件大小查看工具（如Windows的资源管理器、Linux的`du`命令等）来查看打包后的文件大小。
>    - 使用文本编辑器或开发者工具打开打包后的文件，查看其内容和结构，了解哪些模块或依赖项占用了较大的体积。

请注意，以上方法都可以帮助你监控和分析Webpack打包后的bundle体积，但具体选择哪种方法取决于你的项目需求和个人偏好。在进行分析时，重点关注那些体积较大的模块或依赖项，并根据实际情况进行优化。









## 参考文献

- 「 Tecvan: 一文吃透 Webpack 核心原理  」
- [面试官：webpack原理都不会？](https://github.com/Cosen95/blog/issues/48)
- [细说 webpack 之流程篇 ](https://developer.aliyun.com/article/61047)
- [「吐血整理」再来一打Webpack面试题](https://juejin.cn/post/6844904094281236487)