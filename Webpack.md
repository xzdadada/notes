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
