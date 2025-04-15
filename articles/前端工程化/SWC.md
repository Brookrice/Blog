# SWC


## 什么是swc

![swc](../../images/前端工程化/swc.jpg)

SWC 既可用于编译，也可用于打包。对于编译，它使用现代 JavaScript 功能获取 JavaScript / TypeScript 文件并输出所有主流浏览器支持的有效代码。

`SWC在单线程上比 Babel 快 20 倍，在四核上快 70 倍。`

简单点来说swc实现了和babel一样的功能，但是它比babel快。

为什么快?

1. 编译型 Rust 是一种编译型语言，在编译时将代码转化为机器码（底层的 CPU 指令）。这种机器码在执行时非常高效，几乎不需要额外的开销。
2. 解释型 JavaScript 是一种解释型语言，通常在浏览器或 Node.js 环境中通过解释器运行。尽管现代的 JavaScript 引擎（如 V8 引擎）使用了 JIT（即时编译）技术来提高性能，但解释型语言本质上还是需要更多的运行时开销。

swc官网 `https://swc.rs/`

## 核心功能

1. JavaScript/TypeScript 转换 可以将现代 JavaScript（ES6+）和 TypeScript 代码转换为兼容旧版 JavaScript 环境的代码。这包括语法转换（如箭头函数、解构赋值等）以及一些 polyfill 的处理
2. 模块打包 SWC 提供了基础的打包功能，可以将多个模块捆绑成一个单独的文件
3. SWC 支持代码压缩和优化功能，类似于 Terser。它可以对 JavaScript 代码进行压缩，去除不必要的空白、注释，并对代码进行优化以减小文件大小，提高加载速度
4. SWC 原生支持 TypeScript，可以将 TypeScript 编译为 JavaScript
5. SWC 支持 React 和 JSX 语法，可以将 JSX 转换为标准的 JavaScript 代码。它还支持一些现代的 React 特性

## 案例
### 1. 语法转换：将新版本的 JavaScript 语法转换为旧版本的语法
转换前
```javascript
//语法
const a = (params = 2) => 1 + params;
const b = [1, 2, 3]
const c = [...b, 4, 5]
class Babel {

}
new Babel()
//API
const x = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10].filter((x) => x % 2 === 0)
const y = Object.assign({}, { name: 1 })
```

swc 转换代码

```javascript
import swc from '@swc/core'

const result = swc.transformFileSync('./test.js', {
  jsc: {
    target: "es5", //代码转换es5
    parser: {
      syntax: 'ecmascript'
    }
  }
})
console.log(result.code)
```
转换后结果

```javascript
//语法
function _array_like_to_array(arr, len) {
  if (len == null || len > arr.length) len = arr.length;
  for(var i = 0, arr2 = new Array(len); i < len; i++)arr2[i] = arr[i];
  return arr2;
}
function _array_without_holes(arr) {
  if (Array.isArray(arr)) return _array_like_to_array(arr);
}
function _class_call_check(instance, Constructor) {
  if (!(instance instanceof Constructor)) {
    throw new TypeError("Cannot call a class as a function");
  }
}
function _iterable_to_array(iter) {
  if (typeof Symbol !== "undefined" && iter[Symbol.iterator] != null || iter["@@iterator"] != null) return Array.from(iter);
}
function _non_iterable_spread() {
  throw new TypeError("Invalid attempt to spread non-iterable instance.\\nIn order to be iterable, non-array objects must have a [Symbol.iterator]() method.");
}
function _to_consumable_array(arr) {
  return _array_without_holes(arr) || _iterable_to_array(arr) || _unsupported_iterable_to_array(arr) || _non_iterable_spread();
}
function _unsupported_iterable_to_array(o, minLen) {
  if (!o) return;
  if (typeof o === "string") return _array_like_to_array(o, minLen);
  var n = Object.prototype.toString.call(o).slice(8, -1);
  if (n === "Object" && o.constructor) n = o.constructor.name;
  if (n === "Map" || n === "Set") return Array.from(n);
  if (n === "Arguments" || /^(?:Ui|I)nt(?:8|16|32)(?:Clamped)?Array$/.test(n)) return _array_like_to_array(o, minLen);
}
var a = function() {
  var params = arguments.length > 0 && arguments[0] !== void 0 ? arguments[0] : 2;
  return 1 + params;
};
var b = [
  1,
  2,
  3
];
var c = _to_consumable_array(b).concat([
  4,
  5
]);
var Babel = function Babel() {
  "use strict";
  _class_call_check(this, Babel);
};
new Babel();
//API
var x = [
  1,
  2,
  3,
  4,
  5,
  6,
  7,
  8,
  9,
  10
].filter(function(x) {
  return x % 2 === 0;
});
var y = Object.assign({}, {
  name: 1
});
```
`swc转换用时 default: 8.088ms`

`Babel转换用时 default: 417.59ms`

### 2. swc转换react jsx语法
test.jsx
```javascript
import react from 'react'
import { createRoot } from 'react-dom/client'

const App = () => {
  return <div>大伟</div>
}

createRoot(document.getElementById('root')).render(<App />)
```
转换代码
js
```javascript
import swc from '@swc/core'
console.time()
const result = swc.transformFileSync('./test.jsx', {
  jsc: {
    target: "es5", //代码转换es5
    parser: {
      syntax: 'ecmascript',
      jsx: true
    },
    transform:{
      react: {
        runtime: 'automatic'
      }
    }
  }
})
console.log(result.code)
console.timeEnd()
```
结果
```javascript
import { jsx as _jsx } from "react/jsx-runtime";
import react from 'react';
import { createRoot } from 'react-dom/client';
var App = function() {
  return /*#__PURE__*/ _jsx("div", {
    children: "大伟"
  });
};
createRoot(document.getElementById('root')).render(/*#__PURE__*/ _jsx(App, {}));
```
`swc转换用时 default: 4.251ms`

`Babel转换用时 default: 80.613ms`

### 3. swc简易打包

创建配置文件spack.config.js

编写以下代码执行 npx spack打包
```javascript
const { config } = require('@swc/core/spack')
const path = require('path')
module.exports = config({
  entry: {
    web: path.join(__dirname, './test.js') //入口
  },
  output: {
    path: path.join(__dirname, './dist'), //出口
    name: 'test.js'
  }
})
```
终端运行命令
```javascript
npm i @swc/cil -D
```