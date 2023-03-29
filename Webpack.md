# webpack

### 1、有哪些常⻅的 Loader？
***file-loader：*** 把⽂件输出到⼀个⽂件夹中，在代码中通过相对URL去引⽤输出的⽂件    
***url-loader：*** 和file-loader类似，但是能在⽂件很⼩的情况下以base64的⽅式把⽂件内容注⼊到代码中去     
***source-map-loader：*** 加载额外的SourceMap⽂件，以⽅便断点调试      
***image-loader：*** 加载并且压缩图⽚⽂件     
***babel-loader：*** 把ES6转换成 ES5      
***css-loader：*** 加载CSS，⽀持模块化、压缩、⽂件导⼊等特性     
***style-loader：*** 把CSS代码注⼊到JavaScript中，通过DOM操作去加载CSS   
***eslint-loader：*** 通过ESLint检查JavaScript代码
***vue-loader：*** 将vue单文件组件代码解析为 vue runtime 可识别的组件模块

注意：在Webpack中，loader的执行顺序是从右向左执行的。因为webpack选择了compose这样的函数式编程方式，这种方式的表达式执行是从右向左的。


### 2、有哪些常⻅的 Plugin？
***define-plugin：*** 定义环境变量     
***uglifyjs-webpack-plugin：*** 通过UglifyES压缩ES6代码     
***webpack-parallel-uglify-plugin:*** 多核压缩，提⾼压缩速度      
***webpack-bundle-analyzer:*** 可视化webpack输出⽂件的体积 
***mini-css-extract-plugin:*** CSS提取到单独的⽂件中，⽀持按需加载    
***html-webpack-plugin：*** 简化html⽂件创建


### 3、bundle，chunk，module 是什么？
***bundle：*** 是由webpack打包出来的⽂件；     
***chunk：*** 代码块，⼀个chunk由多个模块组合⽽成，⽤于代码的合并和分割；     
***module：*** 是开发中的单个模块，在webpack的世界，⼀切皆模块，⼀个模块对应⼀个⽂件，webpack会从配置的entry中递归开始找出所有依赖的模块。


### 4、Loader 和 Plugin 的不同？
***不同的作⽤:***     
Loader直译为"加载器"。Webpack将⼀切⽂件视为模块，但是webpack原⽣是只能解析js⽂件，如果想将其他⽂件也打包的话，就会⽤到loader。 所以Loader的作⽤是让webpack拥有了加载和解析⾮JavaScript⽂件的能⼒。   
Plugin直译为"插件"。Plugin可以扩展webpack的功能，让webpack具有更多的灵活性。在Webpack运⾏的⽣命周期中会⼴播出许多事件，Plugin可以监听这些事件，在合适的时机通过Webpack提供的API改变输出结果。    
***不同的⽤法:***    
Loader在module.rules中配置，也就是说他作为模块的解析规则⽽存在。 类型为数组，每⼀项都是⼀个 Object ，⾥⾯描述了对于什么类型的⽂件（ test ），使⽤什么加载( loader )和使⽤的参数（ options ）     
Plugin在plugins中单独配置。类型为数组，每⼀项是⼀个plugin的实例，参数都通过构造函数传⼊。


### 5、webpack、rollup优劣？
webpack适⽤于⼤型复杂的前端站点构建: webpack有强⼤的loader和插件⽣态,打包后的⽂件实际上就是⼀个⽴即执⾏函数，这个⽴即执⾏函数接收⼀个参数，这个参数是模块对象，键为各个模块的路径，值为模块内容。⽴即执⾏函数内部则处理模块之间的引⽤，执⾏模块等,这种情况更适合⽂件依赖复杂的应⽤开发。     
rollup适⽤于基础库的打包，如vue、d3等: Rollup就是将各个模块打包进⼀个⽂件中，并且通过Tree-shaking来删除⽆⽤的代码, 可以最⼤程度上降低代码体积，但是rollup没有webpack如此多的如代码分割、按需加载等⾼级功能，其更聚焦于库的打包，因此更适合库的开发。


### 6、webpack热更新的实现原理？
webpack的热更新⼜称热替换（Hot Module Replacement），缩写为HMR。这个机制可以做到不⽤刷新浏览器⽽将新变更的模块替换掉旧的模块。   
原理：  

![](http://rr7byi9s5.hb-bkt.clouddn.com/webpack%E7%83%AD%E6%9B%B4%E6%96%B0.webp)  

先简述一下流程。这里涉及到：webpack、webpack-dev-server、浏览器
1. webpack-dev-server中的 webpack-dev-middleware 中间件调用了 webpack的watch方法，监听原文件的变化，根据配置文件进行模块的重新打包，并将打包后的代码通过js对象保存在内存中。
2. 在启动时 webpack-dev-server 就已经通过sockjs与浏览器端建立了一个webSocket长链接，并且监听webpack的compile的done事件，在compile完成后将新模块的hash值通过webSocket发送到浏览器。
3. 同时， webpack-dev-server 本身还会在初始化的时候使用webpack-dev-server/client包添加到entry，这样在构建出来的包里就存在处理scoket请求的client代码了。
4. 浏览器端接收socket发送过来的文件hash并存储，之后根据webpack/hot的配置决定是发送hash给webpack去做热更新还是直接重新刷新页面。（如果配置了hot，则将hash发送给webpack在浏览器端的代码去做热更新）
5. 如果做热更新则 浏览器上的webpack/hot/dev-server会调用check方法作对比，check之后会调用 hotDownloadUpdateChunk 方法获取新hash值所对应的代码块，然后将代码块交个 webpack/lib/HotModuleReplacement.runtime 进行热更新。
6. webpack/lib/HotModuleReplacement.runtime 中执行了hotApply方法，这个方法删除了更新前的代码缓存并将新的模块添加到modules中。 （如果在热更新过程中出现错误，热更新将回退到刷新浏览器）
7. 在入口文件中调用 HMR 的 accept 方法，即将变化的文件的返回值插入到页面。

另外 vue-loader 中对于加载的热更新，vue-loader在内部也执行了 module.hot.accept() ，这部分在 vue-hot-reload-api 中有介绍

### 7、Babel 的原理是什么?
babel的转译过程也分为三个阶段，这三步具体是： 
- 解析Parse: 将代码解析⽣成抽象语法树（AST），即词法分析与语法分析的过程； 
- 转换Transform: 对于AST进⾏变换⼀系列的操作，babel接受得到AST并通过 
- babel-traverse对其进⾏遍历，在此过程中进⾏添加、更新及移除等操作；   
⽣成 Generate: 将变换后的AST再转换为JS代码, 使⽤到的模块是 babel-generator。    
![](http://rr7byi9s5.hb-bkt.clouddn.com/blog-image/babel-banner.png)


### 8、如何⽤webpack 来优化前端性能？
⽤webpack优化前端性能是指优化webpack的输出结果，让打包的最终结果在浏览器运⾏快速⾼效。   
首先使用 webpack-bundle-analyzer插件分析打包结果，根据打包结果快速定位需要优化的内容。
***压缩代码：*** 删除多余的代码、注释、简化代码的写法等等⽅式。可以利⽤webpack的UglifyJsPlugin和ParallelUglifyPlugin（webpack5内置使用terser-webpack-plugin）来压缩JS⽂件，利⽤cssnano（css-loader?minimize）来压缩css。    
***利⽤CDN 加速:*** 在构建过程中，将引⽤的静态资源路径修改为CDN上对应的路径。可以利⽤webpack对于output参数和各loader的publicPath参数来修改资源路径。    
***Tree Shaking:*** 将代码中永远不会⾛到的⽚段删除掉。可以通过在启动webpack时追加参数 --optimize-minimize来实现。   
***Code Splitting:*** 将代码按路由维度或者组件分块(chunk),这样做到按需加载,同时可以充分利⽤浏览器缓存。
***提取公共第三⽅库:*** SplitChunksPlugin插件来进⾏公共模块抽取, 利⽤浏览器缓存可以⻓期缓存这些⽆需频繁变动的公共代码。


### 9、如何提⾼webpack 的构建速度？
1. 多⼊⼝情况下，使⽤CommonsChunkPlugin来提取公共代码 
2. 通过externals配置来提取常⽤库 
3. 利⽤DllPlugin和DllReferencePlugin预编译资源模块   
通过DllPlugin来对那些我们引⽤但是绝对不会修改的npm包来进⾏预编译，再通过DllReferencePlugin将预编译的模块加载进来。 
4. 使⽤thread-loader实现多线程加速编译 
5. 使⽤webpack-uglify-parallel来提升uglifyPlugin的压缩速度。    
原理上webpack-uglify-parallel采⽤了多核并⾏压缩来提升压缩速度.
6. 使⽤Tree-shaking来剔除多余代码


### 10、Webpack构建流程是怎样的？
***初始化参数：*** 从配置文件和 Shell 语句中读取与合并参数，得出最终的参数   
开始编译：用上一步得到的参数初始化 Compiler 对象，加载所有配置的插件，执行对象的 run 方法开始执行编译    
***确定入口：*** 根据配置中的 entry 找出所有的入口文件    
***编译模块：*** 从入口文件出发，调用所有配置的 Loader 对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理         
***完成模块编译：*** 在经过第4步使用 Loader 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系     
***输出资源：*** 根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk，再把每个 Chunk 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会    
***输出完成：*** 在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统。
最后，一切的配置都不如升个版本，新版本yyds。

### 11、如何编写Loader?
Loader支持链式调用，所以开发上需要严格遵循“单一职责”，每个Loader只负责自己需要负责的事情。    
- Loader运行在Node.js中，我们可以调用任意Node.js自带的API或者安装第三方模块进行调用
- Webpack传给Loader的原内容都是UTF-8格式编码的字符串，当某些场景下Loader处理二进制文件时，需要通过`exports.raw = true`告诉Webpack该Loader是否需要二进制数据
- 尽可能的异步化Loader，如果计算量很小，同步也可以
- Loader是无状态的，我们不应该在Loader中保留状态
- 使用loader-utils和schema-utils为我们提供的实用工具
- 加载本地Loader方法
  - Npm 
  - linkResolveLoader


### 12、如何编写Plugin? 
webpack在运行的生命周期中会广播出许多事件，Plugin可以监听这些事件，在特定的阶段钩入想要添加的自定义功能。Webpack的Tapable事件流机制保证了插件的有序性，使得整个系统扩展性良好。    
- compiler暴露了和Webpack整个生命周期相关的钩子
- compilation暴露了与模块和依赖有关的粒度更小的事件钩子
- 插件需要在其原型上绑定apply方法，才能访问compiler实例
- 传给每个插件的compiler和compilation对象都是同一个引用，若在一个插件中修改了它们身上的属性，会影响后面的插件
- 找出合适的事件点去完成想要的功能。    
  - emit事件发生时，可以读取到最终输出的资源、代码块、模块及其依赖，并进行修改(emit事件是修改Webpack输出资源的最后时机)
  - watch-run当依赖的文件发生变化时会触发
- 异步的事件需要在插件处理完任务时调用回调函数通知Webpack进入下一个流程，不然会卡住。
