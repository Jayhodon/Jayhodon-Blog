---


title: Webpack相关

date: 2023-04-19 18:45:56

tags: [前端,webpack]

categories: [前端,个人知识库]
---


> ​		**webpack** 是一个用于现代 JavaScript 应用程序的 *静态模块打包工具*。当 webpack 处理应用程序时，它会在内部构建一个依赖图，此依赖图对应映射到项目所需的每个模块，并生成一个或多个 *bundle*。



### Webpack构建流程



webpack 整个庞大的体系大致可以抽象为三方面的知识：

> 1. **构建的核心流程**
> 2. **loader 的作用**
> 3. **plugin 架构与常用套路**

三者协作构成 webpack 的主体框架：

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



## Webpack相关


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

[Webpack相关试题](https://juejin.cn/post/6844904094281236487)





## 参考文献

- 「 Tecvan: 一文吃透 Webpack 核心原理  」
- [面试官：webpack原理都不会？](https://github.com/Cosen95/blog/issues/48)
- [细说 webpack 之流程篇 ](https://developer.aliyun.com/article/61047)
- [「吐血整理」再来一打Webpack面试题](https://juejin.cn/post/6844904094281236487)