[TOC]

# Webpack

## 流程

1. 初始化阶段：
   1. 初始化参数：从配置文件、配置对象、Shell 参数中读取，与默认配置进行结合，得出最终的参数
   2. 创建编译器对象：利用参数创建 `Compiler` 对象
   3. 初始化编译环境：注入内置插件、注册模块工厂、初始化 RuleSet 集合、加载配置的插件等
   4. 开始编译：执行 `compiler` 对象的 `run` 方法
   5. 确定入口：根据配置中的 `entry` 找出入口文件的集合，调用 `compilition.addEntry` 将入口文件转换为 `dependence` 对象
2. 构建阶段：
   1. 编译模块（make）：根据 `entry` 的 `dependence` 创建 `module` 对象，调用 `loader` 将模块转换成标准 JS 内容，通过 JS 解释器将内容转换为 AST 对象，然后从中找出模块的依赖模块，递归执行，获得每个模块的内容以及模块的依赖关系图 `ModuleGraph`
3. 生成阶段：
   1. 输出资源（seal）：根据入口和模块之间的依赖关系，组成一个个 `chunk`，再把每个 `chunk` 转换为单独的文件进行输出
   2. 写入文件系统：确定好输出内容后，将内容写入文件系统

## 组成部分

loader：用于对特定的文件类型进行处理，

由于 webpack 只能处理 js 以及 JSON 格式的文件，对于像 css 或图片格式的文件，需要对应的 loader 来处理，才能打包到最后的产物中

plugin：用于丰富 webpack 的能力

webpack 运行的生命周期中，会广播出各种事件，Plugin 负责监听这些事件，然后通过 webpack 的相关 API 来更改输出结果



## 构建阶段

1. 根据文件类型创建 module 子类
2. 调用 loader，转译 module 的内容
3. 调用 acorn 将 js 文本解析为 AST
4. 遍历 AST，触发 hooks

### 模块

一个 Module 类的基本属性如下

```js
class Module extends DependenciesBlock {
	constructor(type, context = null, layer = null) {
		super();
		this.type = type;
		this.context = context;
		this.layer = layer;
		this.needId = true;
		...
	}
}
```

### 如何记录依赖关系

Module 类继承 DependenciesBlock 类，在 DependenciesBlock 中，存在 dependencies 属性，用于记录本模块所有的依赖 Dependency

```js
class DependenciesBlock{
    constructor() {
		/** @type {Dependency[]} */
		this.dependencies = [];
		/** @type {AsyncDependenciesBlock[]} */
		this.blocks = [];
		/** @type {DependenciesBlock | undefined} */
		this.parent = undefined;
	}
    addDependency(dependency) {
        this.dependencies.push(dependency);
    }
}
```

## 特性

### webpack tree shaking

#### 原理

由于 ES2016 是通过 import 以及 export 静态导入导出，所以可以通过这两个语句对于模块做静态的分析，从字面量即可推断出模块值被使用的情况，从而构建对应的依赖关系图

#### 使用方法

1. 使用 ESM 规范编写模块代码

2. 标记功能：配置 `optimization.usedExports` 为 `true`

3. 代码优化功能：

   - 配置 `mode = production`
   - 配置 `optimization.minimize = true`
   - 提供 `optimization.minimizer` 数组

#### Make 阶段

生成 Module，根据 AST 中的 import 以及 export 语句，创建对应的 Dependency 对象

通过 https://astexplorer.net/ 可以查看 ast 结果

对于如下的代码

```js
import a from './a.js'
import {funcB} from './b.js'

export function square(x) {
    return x * x;
}

export default x = 10
```

其通过 acorn 解析出的 AST 为如下形式

```json
{
  "type": "Program",
  "body": [
    {
      "type": "ImportDeclaration",
       ...
      "specifiers": [
        {
          "type": "ImportDefaultSpecifier",
          ...
        }
      ],
      "source": {
        "type": "Literal",
        ...
        "value": "./a.js",
        "raw": "'./a.js'"
      }
    },
    {
      "type": "ImportDeclaration",
	  ...
      "specifiers": [
        {
          "type": "ImportSpecifier",
          ...
        }
      ],
      "source": {
        "type": "Literal",
        "value": "./b.js",
        "raw": "'./b.js'"
      }
    },
    {
      "type": "ExportNamedDeclaration",
      ...
    },
    {
      "type": "ExportDefaultDeclaration",
      ...
    }
  ],
  "sourceType": "module"
}
```

对于不同的 import 以及 export 方式，标记为不同的 type，在 javascriptParser 中，进行对应处理

根据不同的 export 类型，会创建不同的 `HarmonyExport...Dependency`，然后加入Module 的 `dependencies` 中

对于 import 语句，会解析其属性，然后创建 sideEffectDep

FlagDependencyExportsPlugin 插件：遍历 dependencies 数组，将 Dependency 转换为 ExportInfo，存储在 ModuleGraphModule 中

#### Seal 阶段

从 entry 开始遍历 ModuleGraph，对于 module 的 exportsInfo，通过 `compilation.getDependencyReferencedExports(dep, runtime)` 获取被引用的导出值

调用 `exportInfo.setUsedConditionally` 标记导出值的使用情况

将所有被使用的导出值，记录在 `exportInfo._usedInRuntime` 中

#### 打包阶段

1. 通过 exportsInfo，对于被引用的导出以及未被引用的导出，分别创建对应的 HarmonyExportInitFragment 对象，加入 initFragments 数组

   1. HarmonyExportInitFragment 通过 exportMap 记录被引用的导出值（`Map<exportName, variableName>`），通过 unusedExports 记录未被引用的导出值（`exportName`）

2. 遍历 initFragments 数组，生成最终结果

   ```js
   getContent({ runtimeTemplate, runtimeRequirements }) {
       ...
   
       const unusedPart =
           this.unusedExports.size > 1
               ? `/* unused harmony exports ${joinIterableWithComma(
                       this.unusedExports
                   )} */\n`
               : this.unusedExports.size > 0
                   ? `/* unused harmony export ${first(this.unusedExports)} */\n`
                   : "";
       const definitions = [];
       const orderedExportMap = Array.from(this.exportMap).sort(([a], [b]) =>
           a < b ? -1 : 1
       );
       for (const [key, value] of orderedExportMap) {
           definitions.push(
               `\n/* harmony export */   ${propertyName(
                   key
               )}: ${runtimeTemplate.returningFunction(value)}`
           );
       }
       const definePart =
           this.exportMap.size > 0
               ? `/* harmony export */ ${RuntimeGlobals.definePropertyGetters}(${
                       this.exportsArgument
                   }, {${definitions.join(",")}\n/* harmony export */ });\n`
               : "";
       return `${definePart}${unusedPart}`;
   }
   ```

3. 删除不可能被执行到的代码（由带有 DCE 功能的插件完成）

webpack 编译完之后，会将模块中未使用的导出通过 `unused harmony [default] exports ...` 进行罗列，并且只保留定义语句，不会在 chunk 中生成对应的 export，此类代码被称为 Dead Code

### 热更新

#### 使用方法

#### 原理

监听文件更改，重新编译，webpack 传输

1. 更改文件，进行保存
2. 触发 webpack 相关事件，对文件重新编译打包，并生成唯一的 hash 值
3. 生成补丁文件 `manifest.json` 和 `chunk.js` 模块
4. 通过 webpack 通知浏览器，获取 manifest 文件，进行 hash 值比对
5. 若 hash 更改，则获取 `chunk.js`，然后重新 `render`，实现模块替换

### Proxy 代理

webpack 的 devserver 会开启一个本地的同源代理服务器，当浏览器发送请求的时候，通过配置，将需要代理的请求通过代理服务器进行代理

#### 原理

跨域策略是浏览器与服务器之间的限制，服务器与服务器之间没有同源策略限制

## 产物分析

webpack 打包后的文件结构大致如下

```js
var __webpack_modules__ = ({
    "./src/math.js":
        ((__unused_webpack___webpack_module__, __webpack_exports__, __webpack_require__) => {
            __webpack_require__.d(__webpack_exports__, {
                cube: () => (/* binding */ cube)
            });
            function cube(x) {
                return x * x * x;
            }
        })
});
// The module cache
var __webpack_module_cache__ = {};
```

`__webpack_modules__`：记录所有的 webpack 模块，以 文件名-函数 的方式记录

`__webpack_module_cache__`：用于存储被引用过的模块

为什么使用函数？

利用函数的块级作用域，可以使得模块之间互不干扰

```js
// The require function
function __webpack_require__(moduleId) {
    // Check if module is in cache
    var cachedModule = __webpack_module_cache__[moduleId];
    if (cachedModule !== undefined) {
        return cachedModule.exports;

    }
    // Create a new module (and put it into the cache)
    var module = __webpack_module_cache__[moduleId] = {
        // no module.id needed
        // no module.loaded needed
        exports: {}

    };

    // Execute the module function
    __webpack_modules__[moduleId](module, module.exports, __webpack_require__);

    // Return the exports of the module
    return module.exports;

}

/* webpack/runtime/define property getters */
(() => {
    // define getter functions for harmony exports
    __webpack_require__.d = (exports, definition) => {
        for (var key in definition) {
            if (__webpack_require__.o(definition, key) && !__webpack_require__.o(exports, key)) {
                Object.defineProperty(exports, key, { enumerable: true, get: definition[key] });

            }

        }

    };

})();

/* webpack/runtime/hasOwnProperty shorthand */
(() => {
    __webpack_require__.o = (obj, prop) => (Object.prototype.hasOwnProperty.call(obj, prop))

})();
```

`__webpack_require__`：模块的引用逻辑，可以类比 CommonJS 的 require``

`__webpack_require__.d`：工具函数，通过 Object.defineProperty 将模块的导出值绑定入模块的 exports 对象

__`webpack_require__.o`：工具函数：通过 `hasOwnProperty` 判断属性是否应该被添加

```js
/*!**********************!*\
  !*** ./src/index.js ***!
  \**********************/
var _math_js__WEBPACK_IMPORTED_MODULE_2__ = __webpack_require__(/*! ./math.js */ "./src/math.js");

function component() {
    const element = document.createElement('pre');

    element.innerHTML = [
        '你好 webpack！',
        '5 的立方等于 ' + (0, _math_js__WEBPACK_IMPORTED_MODULE_2__.cube)(5)
    ].join('\n\n');

    return element;
}

document.body.appendChild(component());

```

入口文件的逻辑，通过 `__webpack_require__` 引用模块
