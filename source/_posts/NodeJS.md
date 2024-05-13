---
title: Node.js相关

date: 2024-5-10 12:55:28

tags: [前端,个人知识库,Node.js]

categories: [前端,个人知识库,Node.js]

summary: '一篇针对 Node.js 相关知识点整合的文，用于剖析Vue的相关原理以及特性梳理。'
---



## Node.js



### Node.js相关



#### Node.js的三大特性

Node.js 的三大特性通常被总结为其非阻塞的 I/O 模型、单线程事件循环以及跨平台性。这些特性使得 Node.js 在处理高并发、I/O 密集型任务时表现出色，同时也为开发者提供了编写服务器端应用程序的灵活性和便捷性。

1. 非阻塞的 I/O 模型
   - Node.js 采用了非阻塞的 I/O 模型，这意味着当 Node.js 执行 I/O 操作（如读取文件、网络请求等）时，它不会等待该操作完成再继续执行后续代码，而是会立即返回一个“正在处理”的标识，然后继续执行后续代码。当 I/O 操作完成时，Node.js 会通过事件和回调函数的方式通知应用程序进行相应的处理。
   - 这种非阻塞的 I/O 模型使得 Node.js 能够高效地处理大量的并发请求，而不会像传统的多线程模型那样因为线程切换和同步阻塞而浪费大量的 CPU 时间。
2. 单线程事件循环
   - Node.js 采用了单线程的事件循环机制来处理异步操作。这意味着在 Node.js 中，所有的代码都是在一个线程中执行的，包括同步代码和异步代码的回调函数。
   - 当一个异步操作（如网络请求）被触发时，Node.js 会将其添加到事件队列中，并在适当的时候（如 I/O 操作完成）从事件队列中取出并执行相应的回调函数。
   - 这种单线程事件循环的机制避免了多线程编程中的线程同步和线程安全问题，使得 Node.js 的代码更加简洁、易于理解和维护。
3. 跨平台性
   - Node.js 是基于 V8 引擎（Google 开发的开源高性能 JavaScript 和 WebAssembly 引擎）构建的，因此具有跨平台的特性。只要安装了适合当前操作系统的 Node.js 运行时环境，就可以在任何支持 Node.js 的平台上运行 Node.js 应用程序。
   - 这种跨平台的特性使得 Node.js 成为一种非常流行的服务器端开发技术，可以在不同的操作系统和硬件平台上运行和部署应用程序。

除了以上三大特性外，Node.js 还具有其他许多优点，如开源、社区支持丰富、生态系统庞大等，这些优点都使得 Node.js 成为一种非常受欢迎的服务器端开发技术。



#### Node.js中事件循环的流程

Node.js中的事件循环是其非阻塞I/O模型的核心组成部分，它负责调度执行各种回调函数。以下是Node.js事件循环的基本流程：

1. 初始化

   - 当Node.js应用程序启动时，它会初始化事件循环，加载需要的模块，并执行全局代码。

2. 进入事件循环

   - 一旦初始化完成，Node.js就会进入事件循环，开始监听各种事件。

3. 事件循环的阶段

   - Node.js的事件循环分为多个阶段，每个阶段都有与之对应的事件队列。这些阶段按照特定的顺序执行，以处理不同类型的事件。

    

   Timers（定时器）

   - 此阶段执行setTimeout()和setInterval()的回调函数。

    

   Pending Callbacks（待定回调）

   - 执行延迟到下一个循环迭代的I/O回调。

    

   Idle, Prepare（空闲、准备）

   - 这两个阶段用于Node.js内部处理，通常不会对应用程序开发者产生影响。

    

   Poll（轮询）

   - 检索新的I/O事件；执行与I/O相关的回调（几乎所有回调，除了关闭回调、定时器回调和setImmediate()）；如果在此阶段没有可用的回调，并且脚本没有使用setImmediate()，则事件循环将等待回调被添加到队列中，或者直到定时器到期。

    

   Check（检查）

   - 执行setImmediate()的回调函数。

    

   Close Callbacks（关闭回调）

   - 执行例如socket.on('close', ...)的回调。

4. 执行回调

   - 在每个阶段，事件循环都会从相应的事件队列中取出事件并执行它们的回调函数。

5. 循环迭代

   - 一旦某个阶段的事件队列为空，事件循环就会进入下一个阶段。如果所有阶段都为空，并且没有定时器需要等待，则事件循环将暂停执行，直到出现新的I/O事件、定时器到期或者调用了process.nextTick()。

6. process.nextTick()

   - process.nextTick()是一个特殊的函数，它允许你将回调添加到下一个事件循环的顶部，无论当前事件循环的哪个阶段。这意味着在调用process.nextTick()后，你的回调函数会在任何I/O或定时器回调之前执行。

7. 退出事件循环

   - 当事件循环的队列为空，并且没有更多的工作要执行时（例如，没有挂起的定时器或未完成的I/O操作），Node.js将退出事件循环，并且应用程序将终止。

通过理解Node.js事件循环的工作流程，你可以更有效地编写异步代码，避免阻塞操作，并提高应用程序的性能和响应速度。



#### Node.js中整个异步I/O的流程

Node.js中整个异步I/O的流程主要涉及到异步操作的发起、执行以及结果的处理。以下是该流程的基本步骤：

1. 发起异步I/O请求
   - 在Node.js中，当需要执行I/O操作（如读取文件、网络请求等）时，主线程会发起一个异步I/O请求。这个请求会被放入一个任务队列中等待执行。
2. 线程池处理I/O请求
   - Node.js内部有一个线程池，用于处理I/O请求。当主线程发起异步I/O请求后，该请求会被推送到线程池中等待执行。线程池中的线程会负责实际的I/O操作，如读取文件内容、发送网络请求等。
3. I/O操作完成并通知主线程
   - 当线程池中的线程完成I/O操作后，它会将结果封装成一个事件或消息，并将其放入事件队列中。这个事件队列与主线程的事件循环相关联。
4. 事件循环处理事件队列
   - 主线程的事件循环会不断检查事件队列中是否有待处理的事件。如果有，它会取出事件并调用相应的回调函数来处理该事件。在Node.js中，这个回调函数通常是我们在发起异步I/O请求时指定的。
5. 回调函数处理I/O结果
   - 当主线程从事件队列中取出I/O操作的事件时，它会调用与该事件关联的回调函数。这个回调函数会接收I/O操作的结果作为参数，并进行相应的处理。例如，如果我们发起了一个读取文件的异步I/O请求，并在回调函数中接收到了文件内容，那么我们就可以在回调函数中处理这个文件内容。
6. 继续执行后续代码
   - 在Node.js中，由于使用了异步I/O机制，主线程在发起I/O请求后可以立即继续执行后续的代码，而无需等待I/O操作完成。这大大提高了程序的并发性和响应速度。

需要注意的是，Node.js中的异步I/O操作是基于事件循环和回调函数实现的。事件循环负责监听事件队列中的事件并调用相应的回调函数来处理这些事件；而回调函数则负责处理I/O操作的结果并进行后续的逻辑处理。这种机制使得Node.js能够高效地处理大量的并发请求和I/O操作。



#### 模块化规范中require和import的区别

在JavaScript的模块化规范中，`require` 和 `import` 是两种不同的模块导入（或称为“引入”）方式，它们主要存在于CommonJS和ES6（ECMAScript 2015）两种模块系统中。以下是它们之间的一些主要区别：

1. **来源**：

   - `require` 是 CommonJS 模块规范的一部分，主要在 Node.js 环境中使用。
   - `import` 是 ES6（也称为ES2015）模块规范的一部分，是现代前端JavaScript项目中的常用方式，但也逐渐被一些现代的后端工具（如Deno）采用。

2. **语法**：

   - `require` 是一个函数调用，用于在运行时动态地加载和解析模块。

   ```javascript
   javascript复制代码
   
   const moduleName = require('module-name');
   ```

   - `import` 是一个静态声明，在编译时解析模块依赖，因此它不能用于在运行时根据条件动态地导入模块。

   ```javascript
   import moduleName from 'module-name';  
   // 或者  
   import { export1, export2 } from 'module-name';
   ```

3. **默认导出**：

   - CommonJS 模块系统没有内建的默认导出机制，但可以通过将对象或值赋值给 `module.exports` 来导出。

   ```javascript
   javascript复制代码
   
   module.exports = myObject;
   ```

   - ES6 模块系统支持默认导出，可以使用 `export default` 语法。

   ```javascript
   javascript复制代码
   
   export default myObject;
   ```

4. **命名导出**：

   - CommonJS 可以通过 `module.exports` 导出多个命名属性。

   ```javascript
   module.exports.property1 = value1;  
   module.exports.property2 = value2;
   ```

   - ES6 同样支持命名导出，但语法更为简洁。

   ```javascript
   export const property1 = value1;  
   export const property2 = value2;
   ```

5. **动态导入**：

   - CommonJS 没有直接的动态导入机制（尽管可以通过某些方式模拟）。
   - ES6 提供了 `import()` 函数，允许在运行时（即在代码执行期间）动态地导入模块。

   ```javascript
   import('module-name')  
     .then(module => {  
       // 使用模块  
     })  
     .catch(err => {  
       // 处理错误  
     });
   ```

6. **循环依赖**：

   - CommonJS 处理循环依赖的方式可能导致一些问题，因为模块可能会在其依赖项完全解析之前就被部分导出。
   - ES6 模块规范对循环依赖有明确的处理机制，可以保证在循环依赖的情况下模块导出的一致性和正确性。

7. **静态分析**：

   - 由于 `import` 是静态的，它可以被静态分析工具（如构建工具、打包工具等）更有效地处理，从而优化打包和树摇（tree shaking）等过程。
   - `require` 是动态的，因此在某些情况下可能不如 `import` 那样易于静态分析。

8. **兼容性**：

   - `require` 在传统的Node.js环境中广泛支持。
   - `import` 在现代浏览器中广泛支持，但在Node.js中，直到较新的版本（v13及更高版本）才开始原生支持，之前需要使用转译器（如Babel）来支持。

在选择使用 `require` 还是 `import` 时，通常取决于你的项目环境和需求。在Node.js中，传统上使用 `require`，但随着Node.js对ES6模块的原生支持不断增强，`import` 的使用也越来越普遍。在前端项目中，`import` 几乎是唯一的选择。





#### pnpm原理

pnpm（Performance-focused Node Package Manager）是一个高性能的Node.js包管理器，它的原理主要基于硬链接和符号链接来管理依赖项，以提高安装速度和减少存储空间占用。以下是pnpm工作原理的详细解释：

1. **本地缓存**：pnpm在安装依赖项时，会先将它们缓存到本地存储空间中。这意味着当再次安装相同的依赖项时，pnpm会直接从本地缓存中读取，而不是重新从远程仓库下载。这种机制显著减少了网络带宽的使用，提高了安装速度。
2. **硬链接和符号链接**：pnpm使用硬链接和符号链接来构建`node_modules`目录。具体来说，它将所有依赖项保存到单个位置（称为内容可寻址存储，CAS），并使用硬链接将这些依赖项链接到项目的`node_modules`目录中。对于需要多个版本的依赖项，pnpm会使用符号链接来确保它们指向正确的版本。这种机制避免了重复的依赖项，进一步减少了存储空间的占用。
3. **依赖关系解析**：pnpm在解析依赖关系时，会首先检查项目根目录下的`package.json`文件，该文件包含了项目需要安装的依赖关系和其他配置信息。然后，pnpm会解析这些依赖关系，包括直接依赖和间接依赖，并根据这些依赖关系创建一个依赖关系树。这个依赖关系树是一个有向无环图，其中每个节点表示一个依赖项，边表示依赖关系。
4. **lockfile文件**：与npm和yarn类似，pnpm也使用lockfile文件来锁定依赖项的版本。当执行`pnpm install`命令时，如果存在一个lockfile文件（例如`pnpm-lock.yaml`），pnpm将优先使用此文件来安装包，以确保版本的一致性。
5. **命令和操作**：pnpm提供了与npm类似的命令和操作，如`pnpm install`（安装项目所需的所有包）、`pnpm add`（安装单个包并将其添加到项目的`dependencies`或`devDependencies`中）、`pnpm remove`（卸载项目中的包并将其从`dependencies`或`devDependencies`中删除）以及`pnpm update`（更新项目中的包）。此外，`pnpm run`命令也用于运行项目中的脚本。

通过以上原理，pnpm在保持与npm和yarn相似的功能和使用体验的同时，显著提高了安装速度和减少了存储空间占用，成为了一个高效且实用的Node.js包管理器。





#### node是单线程的，如何让他在多核CPU上跑满？

虽然Node.js本身基于单线程模型，但我们可以使用多种策略来使其在多核CPU上实现更高的利用率，以下是几种常见的方法：

1. **使用集群模块（Cluster Module）**：
   Node.js提供了一个内置的集群模块，它允许你创建多个子进程，每个子进程运行在一个独立的V8引擎实例上，并监听相同的端口。这样，你就可以将请求分发到不同的子进程，从而充分利用多核CPU资源。

   ```javascript
   const cluster = require('cluster');  
   const os = require('os');  
    
   if (cluster.isMaster) {  
     console.log(`Master ${process.pid} is running`);  
    
     // Fork workers.  
     for (let i = 0; i < os.cpus().length; i++) {  
       cluster.fork();  
     }  
    
     cluster.on('exit', (worker, code, signal) => {  
       console.log(`worker ${worker.process.pid} died`);  
     });  
   } else {  
     // Workers can share any TCP connection  
     // In this case, it is an HTTP server  
     require('./app.js');  
   }
   ```

2. **使用子进程（Child Processes）**：
   除了集群模块，你还可以使用`child_process`模块来创建子进程，每个子进程可以执行不同的任务或处理不同的请求。你可以通过`child_process.fork()`方法来创建一个新的Node.js进程，并通过进程间通信（IPC）来传递消息和数据。

3. **使用工作线程（Worker Threads）**：
   Node.js从v10.5.0版本开始支持工作线程（Worker Threads），这是一种在Node.js环境中运行JavaScript线程的方法。工作线程是独立于主线程的，它们有自己的V8实例，可以执行CPU密集型任务，而不会阻塞主线程。

   ```javascript
   const { Worker, isMainThread, parentPort, workerData } = require('worker_threads');  
    
   if (isMainThread) {  
     // 创建一个工作线程  
     const worker = new Worker(__filename, { workerData: 'hello' });  
    
     worker.on('message', (msg) => {  
       console.log('Received from worker:', msg);  
     });  
    
     worker.on('error', (err) => {  
       console.error('Worker error:', err);  
     });  
    
     worker.on('exit', (code) => {  
       if (code !== 0) {  
         console.error('Worker stopped with exit code', code);  
       }  
     });  
   } else {  
     // 在工作线程中  
     console.log('Worker started with', workerData);  
    
     // 执行一些CPU密集型任务  
    
     parentPort.postMessage('Hello from worker');  
   }
   ```

4. **使用外部服务**：
   对于某些任务，如数据库查询、图像处理或复杂的计算，你可以考虑将它们移到外部服务中，如微服务、队列系统（如RabbitMQ、Kafka）或云函数（如AWS Lambda）。这样，你就可以使用多个服务实例来处理这些任务，从而利用多核CPU和其他资源。

5. **优化代码和依赖项**：
   确保你的代码是高效的，并尽可能减少阻塞操作。使用异步I/O、事件驱动和回调来处理非阻塞操作。同时，优化你的依赖项，确保它们也是高效的，并尽可能减少不必要的依赖项。

请注意，在尝试使Node.js在多核CPU上跑满时，需要谨慎权衡性能和复杂性。过多的子进程或线程可能会增加系统开销和复杂性，并可能导致性能下降或资源争用问题。因此，建议在实际应用中逐步增加并行度，并监控性能和资源使用情况。
