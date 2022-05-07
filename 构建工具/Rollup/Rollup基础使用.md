# Rollup

#### 打包工具的前世今生

为了完成越来越复杂的网页应用，模块化发挥了越来越重要的作用

在 ES2015 发布之前，JavaScript 语言本身并不支持模块化，在众多的非官方模块化规范中，CommonJS 借势 Node.js 的流行成为了应用最为广泛的方案，但浏览器里是无法直接运行 CommonJS 模块代码的，因此必须对代码进行处理，模块打包工具随之产生

#### Rollup

Rollup 可以直接打包符合 ES2015 模块化规范的代码，而并不需要将代码通过 Babel 转化为 CommonJS 模块化规范的形式

与 webpack 不同，Rollup 并不会为每个模块创建独立的函数作用域，而是将所有的代码放置于同一个作用域中，这使得代码的运行更有效率

得益于 ES2015 模块化规范的静态化导入特性，Rollup 可以通过 Tree-Shaking 对代码进行精简，从而得到体积更小的打包代码

有时候我们的代码也会作为一个模块供其他模块使用，Rollup 虽然是一个 ES2015 模块打包器，但也支持多种规范的模块导出，比如 `AMD`、`CommonJS`、`ES2015`、`Global` 及 `UMD` 五种规范的打包文件

> 在使用 Global 及 UMD 规范时，导出的模块可能会被挂载到全局变量上，这时需要设置一个变量名用于模块挂载

#### 一份基础配置文件

```js
// rollup.config.js

import json from "@rollup/plugin-json";
import {terser} from 'rollup-plugin-terser';
import { nodeResolve } from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import { babel } from '@rollup/plugin-babel';

export default {
  input: ['src/entry.js'],
  output: [
    {
      file: 'dist/bundle-cjs.js',
      format: 'cjs'
    },
    {
      file: 'dist/bundle-es.js',
      format: 'es'
    },
    {
      file: 'dist/bundle.js',
      format: 'umd',
      name: 'gzz'
    },
    {
      file: 'dist/bundle.min.js',
      format: 'iife',
      name: 'gzz',
      plugins: [terser()]
    }

  ],
  plugins: [commonjs(), nodeResolve(), babel({ babelHelpers: 'bundled' }), json()]
}
```

介绍一下使用到的插件

现在采用 common.js 规范的 npm 包也很多，而 Rollup 只能识别 ES6 的模块语法，因此需要 @rollup/plugin-commonjs 插件

@rollup/plugin-node-resolve 插件是让 Rollup 可以找到安装到 node_modules 里的外部依赖

@rollup/plugin-babel 插件就是将新的 ES 语法编译为低版本浏览器也能识别的代码

rollup-plugin-terser 压缩代码

@rollup/plugin-json 插件让 Rollup 可以支持 json 文件