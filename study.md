### flow的使用
文件开头加上 // @flow  或 /* @flow*/

### rollup 打包
-w watch
-c 设置配置文件 -c script/config.js
--environment 设置环境变量 --environment TARGET:web-full-dev
--sourcemap  开启代码地图

####  不同的构建版本 dist/readme.md 
  >（了解完整版与运行时版本区别, esm 与 commonjs区别）
  >（基于vue-cli创建的项目，默认导入的vue版本是 .runtime.esm版本， vue-cli查看引入的vue版本 vue inspect > putput.js (把vue的配置输出到output我呢见中去可以看到)）

.min.js  对应生产版本 
.js       对应开发版本

full      完整版本（包含编译器和运行时）
runtime   运行时
编译器： 用来将模板字符串编译js渲染函数的代码，体积大，效率低
运行时： 创建vue实例，渲染并处理虚拟dom的代码 体积小，效率高
  
   不同的模块化方式
umd:  umd版本通用的模块版本，支持多种模块方式 。vue.js默认文件就是运行时 + 编译器的UMD版本  
commonjs: 用来配合老的打包工具 browserify 或 wbepack1
esmodule： vue从2.6开始会提供两个esm构建文件，可以被静态分析，利用这一点来进行tree shaking
esmodule 与 commonjs的差异:  参考阮一峰的教程

### 源码分析
解决flow 报错问题
配置文件入口 找到依赖
rollup 打包配置banner 打包后生成的文件的文件头
1.  同时设置template 和render 函数会发生什么？
入口文件中if(!options.render){编译模板} 由此可见 如果同时传入template 和render，是不会处理template的
  快速查看调用堆栈 利用devtools的call stack查看方法的调用过程

2. 完整版入口文件和运行时入口文件相比多了什么？
完整版重写了mount方法，然段是否render函数，如果没有会把template编译成render函数

3. 分析
src\core\global-api\index.js 注册全局组件指令和方法等
initExtend(Vue) ？？ 怎么用

 为什么vue采用异步渲染？
 如果不下用异步渲染 那么每次更新数据都会对当前组件进行重新渲染 为了性能考虑 vue会在本轮数据更新后 再去异步更新视图

 组件中的数据绑定的都是同一个watcher vue可以把相同的watch更新过滤掉 这是为了提高性能的一个考虑

 nextTick 实现原理

 ### zce caz脚手架
 # 3-2 参考答案



## 1、 请简述 Vue 首次渲染过程。

解析：

1. Vue 初始化，添加实例成员、静态成员，并在原型上挂载__patch__方法和$mount方法。

2. 初始化结束，调用new Vue()。在new Vue()的过程中，调用this.init()方法, 给vue的实例挂载各种功能。

3. 在 this.init() 内部最终会调用 entry-runtime-with-compiler.js中的 vm.$mount(),用于获取render函数。

4.  $mount 获取 render 过程: 如果用户没有传入render,会将template编译为render，如果template也没有，则将el中的内容作为模版，通过 compileToFunctions() 生成render。

5. 接下来调用 runtime/index.js 中的 $mount, 重新获取 el 并调用 mountComponent() 方法。
    mountComponent 用于触发 beforeMount，定义 updateComponent,创建watcher实例，触发mounted,并最终返回vm实例。

6.  创建完watcher的实例后会调用一次 watcher.get() 方法，该方法会调用updateComponent(), updateComponent()又会调用 vm.render() 以及vm.update(),vm._update()会调用vm.**patch**()挂载真实dom,并将真实dom记录于vm.$el中。

   

## 2、 请简述 Vue 响应式原理。

解析：

- 响应式处理整体过程为initState() => initData() => observe()

- 其中observe是响应式处理的入口，通过该方法为data对象转化为响应式对象。

- observe方法接收的是对象且该对象不是响应式时，会为该对象创建一个observe对象，会调用Observer类

- Observer类中判断该value是数组还是对象，进行不同的处理
  - 数组的响应化处理，是重写push,pop,sort等会修改原数组的方法，调用对应的notify.然后遍历数组中的成员，判断其类型决定是否调用observe方法
  - 对象的响应化处理，会调用walk方法，遍历对象中的每个属性，调用defineReactive
  
- defineReactive的核心是为每一个属性定义getter和setter,getter中收集依赖，setter中派发更新,即调用dep.notify。

- dep.notify()会调用watcher的update()方法。如果该watcher未被处理，会被添加到queue队列中，并调用flushSchedulerQueue()方法，该方法会触发对应的钩子函数以及调用watcher.run()更新视图。

  



## 3、 请简述虚拟 DOM 中 Key 的作用和好处。

解析：

​    作用： 标识节点在当前层级的唯一性。
​    好处： 在执行 updateChildren 对比新旧 Vnode 的子节点差异时，通过设置 key 可以进行更高效的比较，便于复用节点。 降低创建销毁节点成本，从而减少 dom 操作，提升更新 dom 的性能。

  

​      

## 4、 请简述 Vue 中模版编译过程。

解析：

- 模版编译入口函数是compileToFunctions,首先判断是否有缓存好的render函数，如果没有，则调用compile。compile用作于合并options, 它将用户传入的options和初始化的options合并起来。然后将template和合并好的options传递给baseCompile。
- 在 baseCompile 中完成了模版编译的核心部分：
  - 调用parse()将template转换成AST对象；
  - 调用optimize()优化 AST tree；
  - 调用 generate() 将优化后的AST tree转换成字符串形式的代码。
- 最后回到compileToFunctions，通过createFunction()将字符串形式的代码转换为函数,并将其挂载到Vue实例的options对应的属性中。