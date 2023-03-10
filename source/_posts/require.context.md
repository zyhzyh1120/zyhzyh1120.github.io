---
title: require.context
date: 2022-11-21 18:40:20
tags: require
categories: require
excerpt: require.context 是 webpack 中，用来创建自己的（模块）上下文。
---

## 初识 require.context

**require.context 是 webpack 中，用来创建自己的（模块）上下文**

webpack 会在构建的时候解析代码中的 require.context()

require.context 函数接收三个参数：

> 1.  要搜索的文件夹目录。
> 2.  是否还应该搜索它的子目录。
> 3.  以及一个匹配文件的正则表达式。


```js
// 语法
require.context(directory, (useSubdirectories = false), (regExp = /^\.\//));
```


```js
// 示例
// （创建了）一个包含了 test 文件夹（不包含子目录）下面的、所有文件名以 `.test.js` 结尾的、能被 require 请求到的文件的上下文。
const testFiles = require.context("./test", false, /\.test\.js$/);
```

通过打印, 可以看出返回的是一个函数，意思就是说，require.context 模块导出（返回）一个（require）函数。
这个函数有三个属性：

> 1. resolve：是一个函数，它返回请求被解析后得到的模块 id。
> 2. keys：也是一个函数，它返回一个数组，由所有可能被上下文模块处理的请求组成。
> 3. id：是上下文模块里面所包含的模块 id. 它可能在你使用 module.hot.accept 的时候被用到
调用 testFiles.keys() 可以打印出./test 目录下所有 test 文件集合

---

### vue 中的应用
*router 路由自动化挂载, 全局组件自动化注册, vuex 状态自动化挂载*

#### router 例子(router 文件夹内模块化创建,亲测可用):

```js
<script>
  const manageFiles = require.context('.', true, /\.js$/);
  console.log(manageFiles.keys()) // ['./a.js'] 返回一个数组，包含全部文件名

  let configRouters = [];
  manageFiles.keys().forEach(key => {
      if (key === './index.js') return    // 如果是当前文件，则跳过
      configRouters = configRouters.concat(manageFiles(key).default)  // 读取出文件中的default模块
  })

  export default configRouters // 抛出一个Vue-router期待的结构的数组
</script>
```

**参考博客:https://blog.csdn.net/viewyu12345/article/details/83012970**

#### 全局组件自动注册
```js
/**
 * 全局组件各个组件按文件夹区分，文件夹名称与组件名无关联，但建议与组件名保持一致
 * 文件夹内至少保留一个文件名为 index 的组件入口，例如 Verify.vue
 * 普通组件必须设置 name 并保证其唯一，自动注册会将组件的 name 设为组件名，可参考 SvgIcon 组件写法
 * 如果组件是通过 js 进行调用，则确保组件入口文件为 verifition-api.js，可参考 ExampleNotice 组件
 * 不自动注册 则名字为其他的
 */
import Vue from 'vue'

const componentsContext = require.context('./', true, /index.(vue|js)$/)
componentsContext.keys().forEach(file_name => {
  // 获取文件中的 default 模块
  const componentConfig = componentsContext(file_name).default
  if (/.vue$/.test(file_name)) {
      Vue.component(componentConfig.name, componentConfig)
  } else if (/.js/.test(file_name)) {
      Vue.use(componentConfig)
  }
})
```
