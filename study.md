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