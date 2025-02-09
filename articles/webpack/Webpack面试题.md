## Webpack面试题

[点击这里](https://zhuanlan.zhihu.com/p/44438844/)



### webpack基本配置

- 拆分和merge
- 启动服务，webpack-dev-server
- 处理ES6， bable-loader, 需要配置.bablerc文件, preset : bable/preset-env 
- 处理样式： 比如css-loader,需要知道执行顺利是从后往前，也就是说，postcss-loader -->> css-loader -->> style--loader
- 处理图片 file-loader(dev)  url-loader(线上 )  base64格式打包到bundle中去





### webpack高级配置



#### 如何配置多个入口文件

- entry入口配置多个bundle，这里的话，比如index，other，会打两个包。
- output配置这个输出的文件



```
output: {
		filename: '[name].[contentHash].js'
}

new HtmlWebpackPlugin({
            template: 'src/index.html'  // 以src/目录下的index.html为模板打包
            chunks: ['index'] // 引用哪些chunks
}
```





#### 如何抽离css以及压缩css



```

// loader需要借助的就是 MiniCssExtractPlugin.loader
new MiniCssExtractPlugin({  
		filename: 'css/main.[contentHash].css'
})
// 压缩css
optimization: {
	minimizer: [TerserJSPlugin(), OptimizeCssAssetsPlugin()]
}
// 
```



#### 抽离公共代码和第三方库代码

通过这个splitChunks完成





### 前端代码为何要进行构建和打包







### loader和plugin的区别







### module chunk bundle区别

- module，指的是源码文件，或者说webpack中万物皆模块。
- chunk，多模块合成的，比如entry，import语法，splitChunk
- bundle， 最后输出的文件。 比如.css , .js, .jpg



### webpack如何实现懒加载的

import



### webpack常见的性能优化

- 优化打包速度和构建速度，体验感和效率。
- 优化产出代码。



#### 优化babel-loader

```
rules: [{
            test: /\.js$/,
            use: ['babel-loader?cacheDirectory'], // 开启缓存
            include: path.resolve(__dirname, 'src') // 明确打包范围
        }]

```





#### IgnorePlugin(生产环境 prod)

1. 这是webpack内置插件
2. 这个插件的作用是：忽略第三方包指定目录，让这些指定目录不要被打包进去。





#### happyPack多进程打包

我们知道js是单线程的，所以我们如果可以开启多进程打包。

提高构建速度，利用多核cpu。

> Happypack 只是作用在 loader 上，使用多个进程同时对文件进行编译。
>
> 在使用 Webpack 对项目进行构建时，会对大量文件进行解析和处理。当文件数量变多之后，Webpack 构件速度就会变慢。由于运行在 Node.js 之上的 Webpack 是单线程模型的，所以 Webpack 需要处理的任务要一个一个进行操作。



由于 JavaScript 是单线程模型，要想发挥多核 CPU 的能力，只能通过多进程去实现，而无法通过多线程实现。



```js
module.exports = {
    ...
    module: {
        rules: [
            test: /\.js$/,
            // use: ['babel-loader?cacheDirectory'] 之前是使用这种方式直接使用 loader
            // 现在用下面的方式替换成 happypack/loader，并使用 id 指定创建的 HappyPack 插件
            use: ['happypack/loader?id=babel'],
            // 排除 node_modules 目录下的文件
            exclude: /node_modules/
        ]
    },
    plugins: [
        ...,
        new HappyPack({
            /*
             * 必须配置
             */
            // id 标识符，要和 rules 中指定的 id 对应起来
            id: 'babel',
            // 需要使用的 loader，用法和 rules 中 Loader 配置一样
            // 可以直接是字符串，也可以是对象形式
            loaders: ['babel-loader?cacheDirectory']
        })
    ]
}
```



关于开启多进程打包

- 项目较大，打包速度慢，开启多进程能提高速度。
- 项目较小，打包速度快，开启多进程会降低速度(进程开销)



#### 配置热更新（本地开发development）

自动刷新：整个页面刷新，速度较慢，状态丢失。

热更新：新代码失效，网页不刷新，状态不丢失。



本地开发环境中，我们需要去增加开启热更新之后的逻辑代码

// HotModuleReplacementPlugin() 需要这个插件

然后在devServer ：{

​	hot: true

}

```
if (module.hot) {
	module.hot.accept(['./main'], () => {
	
	})
}


```



#### DllPlugin 动态连接库插件（不能用于生产的环境中）

- 前端框架 vue react，体积大，构建慢
- 较稳定，不常升级版本。
- 同一个版本只构建一次，不用每次都重新构建。

DllPlugin 打包出dll文件

DllReferencePlugin   使用dll文件



链接: https://segmentfault.com/a/1190000016567986

使用dll时，可以把构建过程分成dll构建过程和主构建过程（实质也就是如此），所以需要两个构建配置文件，例如叫做`webpack.config.js`和`webpack.dll.config.js`。



引用的过程，需要的在index中引用产出物

```
<body>

  <div id="app"></div>

  <!--引用dll文件-->

  <script src="../../dist/dll/react.dll.js" ></script>

</body>
```





我们对比一下 DLL 和前端常接触的网络缓存，一张表就看明白了：

| DLL                                         | 缓存                                   |
| :------------------------------------------ | :------------------------------------- |
| 1.把公共代码打包为 DLL 文件存到硬盘里       | 1.把常用文件存到硬盘/内存里            |
| 2.第二次打包时动态链接 DLL 文件，不重新打包 | 2.第二次加载时直接读取缓存，不重新请求 |
| 3.打包时间缩短                              | 3.加载时间缩短                         |



不错的链接：https://mp.weixin.qq.com/s/jtIbVc9Bl50TIs7YilWbFg







### Scope Hosting

作用域提升，在生产环境下。

我们先来简单分析一下：（没有开启Scope Hoisting ）

**现象**：构建后的代码存在大量的闭包代码



webpack mode为production默认开启（手动加入插件，防止代码压缩）

es6语法,cjs不支持

```javascript
 plugins: [
      // 开启 Scope Hoisting 功能
      new webpack.optimize.ModuleConcatenationPlugin()
  ]
```

简单理**scope hosting**就是把多个作用域用一个作用域取代，以减少内存消耗并减少包裹块代码，从每个模块有一个包裹函数变成只有一个包裹函数包裹所有的模块，但是有一个前提就是，当模块的引用次数大于1时，比如被引用了两次或以上，那么这个效果会无效，也就是被引用多次的模块在被webpack处理后，会被独立的包裹函数所包裹



链接：https://blog.csdn.net/liuhua_2323/article/details/103433533





### webpack常见的性能优化-优化产出代码

- 体积更小
- 合理分包，不重复加载。
- 速度更快，内存使用更小。



有一下几个方面：

- 小图片 base64 编码
- bundle + hash --- contenthash
- 懒加载
- 提取公共代码，splitChunks
- 使用cdn加速，publicPath
- 使用production - 自动开启代码压缩
- Scope Hosting





### ES6 Module 和  Commonjs区别

- ES6 静态引用，编译时引入。值引用
- Commonjs动态引用，执行时引入，值拷贝。

Tree-shaking是在webpack打包时执行的，webpack打包只是静态分析，只是一个编译。

webpack只适合于ES6,原因在于webpack只是编译打包，







## bable面试题

Babel 是一个工具链，主要用于将 ECMAScript 2015+ 版本的代码转换为向后兼容的 JavaScript 语法，以便能够运行在当前和旧版本的浏览器或其他环境中。下面列出的是 Babel 能为你做的事情：

- 语法转换
- 通过 Polyfill 方式在目标环境中添加缺失的特性 (通过 [@babel/polyfill](https://www.babeljs.cn/docs/babel-polyfill) 模块)
- 源码转换 (codemods)





更多的链接: https://www.babeljs.cn/docs/





### core-js 和 regenerator

core-js是标准的库，提供polyfill的集合，但是不支持generator函数语法，处理异步的函数。

所以这个bable-polyfill就是两者的集合。

Bable 7.4版本之后被弃用了。推荐直接使用bable/core-js 和bable/regenerator 



### preset 和 plugin是什么

- 代码转换功能以插件的形式出现，插件是小型的 JavaScript 程序，用于指导 Babel 如何对代码进行转换。

- 你甚至可以编写自己的插件将你所需要的任何代码转换功能应用到你的代码上。

- 例如将 ES2015+ 语法转换为 ES5 语法，我们可以使用诸如 `@babel/plugin-transform-arrow-functions` 之类的官方插件。



**preset-env**， 我们可以使用一个 "preset" （即一组预先设定的插件）。





### babel-runtime和babel-polyfill的区别

[@babel/polyfill](https://www.babeljs.cn/docs/babel-polyfill) 模块包含 [core-js](https://github.com/zloirock/core-js) 和一个自定义的 [regenerator runtime](https://github.com/facebook/regenerator/blob/master/packages/regenerator-runtime/runtime.js) 来模拟完整的 ES2015+ 环境。

这意味着你可以使用诸如 `Promise` 和 `WeakMap` 之类的新的内置组件。



> 对于软件库/工具的作者来说，这可能太多了。如果你不需要类似 `Array.prototype.includes` 的实例方法，可以使用 [transform runtime](https://www.babeljs.cn/docs/babel-plugin-transform-runtime) 插件而不是对全局范围（global scope）造成污染的 `@babel/polyfill`。



**按需引入**



幸运的是，我们所使用的 `env` preset 提供了一个 `"useBuiltIns"` 参数，当此参数设置为 `"usage"` 时，就会加载上面所提到的最后一个优化措施，也就是只包含你所需要的 polyfill。使用此新参数后，修改配置如下：

```json
{
  "presets": [
    [
      "@babel/env",
      {
        "targets": {
          "edge": "17",
          "firefox": "60",
          "chrome": "67",
          "safari": "11.1",
        },
        "useBuiltIns": "usage",
      }
    ]
  ]

```



我们使用 `@babel/cli` 从终端运行 Babel，利用 `@babel/polyfill` 来模拟所有新的 JavaScript 功能，而 `env` preset 只对我们所使用的并且目标浏览器中缺失的功能进行代码转换和加载 polyfill。



**babel-runtime**为了解决polyfill全局污染的问题

```json
{
  "plugins": [
    [
      "@babel/plugin-transform-runtime",
      {
        "absoluteRuntime": false,
        "corejs": false,
        "helpers": true,
        "regenerator": true,
        "useESModules": false,
        "version": "7.0.0-beta.0"
      }
    ]
  ]
}
```





## 前端为何要进行打包和构建

**代码方面**

- 更快的构建速度，加载更快-->> 体积更小-->> Tree Sharking  压缩 合并
- 可以使用更高级的语法或语言（TS， ES6+，模块化）
- 兼容性和错误检查（postcss, Polyfill, eslint）

**开发流程上**

- 统一的开发流程，高效。
- 统一的构建流程和产出代码更规范。
- 构建规范，提测，上线。 





## 常用的loader和plugin

**https://webpack.docschina.org/loaders/**



## babel和webpack区别

- babel js新语法的编译工具，不关心模块化。
- webpack 打包构建工具，是多个loader 和 plugin集合。 



## 如何产出一个lib

```
output：{
	 // lib名称
	 filename: 'lodash',
	 // 输出的目录
	 path: distPath,
	 // lib的全局变量
	 library: 'lodash',
}
```





## 为何Proxy不能被polyfill





## import最终被webpack编译打包成什么

import modulename from 'xxxModule' 和import('moduleName') 区别。



*我们都知道`webpack`的打包过程大概流程是这样的：*



> - 合并`webpack.config.js`和命令行传递的参数，形成最终的配置
> - 解析配置，得到`entry`入口
> - 读取入口文件内容，通过`@babel/parse`将入口内容（code）转换成`ast`
> - 通过`@babel/traverse`遍历`ast`得到模块的各个依赖
> - 通过`@babel/core`（实际的转换工作是由`@babel/preset-env`来完成的）将`ast`转换成`es5 code`
> - 通过循环伪递归的方式拿到所有模块的所有依赖并都转换成`es5`



*求职者*，答：

- import`经过`webpack`打包以后变成一些`Map`对象，`key`为模块路径，`value`为模块的可执行函数；
- 代码加载到浏览器以后从入口模块开始执行，其中执行的过程中，最重要的就是`webpack`定义的`__webpack_require__`函数，负责实际的模块加载并执行这些模块内容，返回执行结果，其实就是读取`Map`对象，然后执行相应的函数；
- 当然其中的异步方法（import('xxModule')）比较特殊一些，它会单独打成一个包，采用动态加载的方式，具体过程：当用户触发其加载的动作时，会动态的在`head`标签中创建一个`script`标签，然后发送一个`http`请求，加载模块，模块加载完成以后自动执行其中的代码，主要的工作有两个，更改缓存中模块的状态，另一个就是执行模块代码。



### [bable](https://mp.weixin.qq.com/s/9kItjKBmnaYSpBGnGpFg1Q)

