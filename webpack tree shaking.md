# webpack tree shaking

## 基础

### 原理

由于 ES2016 是通过 import 以及 export 静态导入导出，所以可以通过这两个语句对于模块做静态的分析，从字面量即可推断出模块值被使用的情况，从而构建对应的依赖关系图

### 使用方法

1. 使用 ESM 规范编写模块代码

2. 标记功能：配置 `optimization.usedExports` 为 `true`

3. 代码优化功能：

   - 配置 `mode = production`
   - 配置 `optimization.minimize = true`
   - 提供 `optimization.minimizer` 数组


## 基本步骤

### Make 阶段

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

![image-20240725184947251](C:\Users\zhengdaxu\AppData\Roaming\Typora\typora-user-images\image-20240725184947251.png)

根据不同的 export 类型，会创建不同的 `HarmonyExport...Dependency`，然后加入Module 的 `dependencies` 中

对于 import 语句，会解析其属性，然后创建 sideEffectDep

![image-20240725192951611](C:\Users\zhengdaxu\AppData\Roaming\Typora\typora-user-images\image-20240725192951611.png)

FlagDependencyExportsPlugin 插件：遍历 dependencies 数组，将 Dependency 转换为 ExportInfo，存储在 ModuleGraphModule 中

### Seal 阶段：

从 entry 开始遍历 ModuleGraph，对于 module 的 exportsInfo，通过 `compilation.getDependencyReferencedExports(dep, runtime)` 获取被引用的导出值

调用 `exportInfo.setUsedConditionally` 标记导出值的使用情况

将所有被使用的导出值，记录在 `exportInfo._usedInRuntime` 中

### 打包阶段：

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

![image-20240724201317994](C:\Users\zhengdaxu\AppData\Roaming\Typora\typora-user-images\image-20240724201317994.png)

webpack 编译完之后，会将模块中未使用的导出通过 `unused harmony [default] exports ...` 进行罗列，并且只保留定义语句，不会在 chunk 中生成对应的 export，此类代码被称为 Dead Code
