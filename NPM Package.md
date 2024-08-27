# NPM Package

## 主要文件

### Package.json 

作用：记录对于项目的各种配置，包括项目信息、项目依赖、脚本、文件等等

属性：

- 信息描述：
  - 必要属性：name，version
  - author、homepage, repository…
- 文件&目录：
  - main, bin, files
- 依赖配置：
  - dependencies, devDependencies…
- 脚本配置：
  - script, config

#### 依赖版本

version 为 `x.y.z` 的格式，x 为主版本号 major，y 为次版本号 minor，z 为补丁版本 patch

1. 固定主版本号：`^x.y.z`，允许自动更新次版本、补丁版本，即 $x.y.z <= v < x+1.0.0$
2. 固定次版本号：`~x.y.z`，允许自动更新补丁版本，即 $x.y.z <= v < x.y+1.0$
3. 完全固定版本号：`x.y.z`

`"workspace:*"`：表明当前引用的包为工作区内部的包，在 npm 安装依赖的时候，直接使用工作区的包，而不是从注册表中进行下载

### Package-lock.json

内容：记录 node_modules 中所有包的信息，包括精确的版本-version、下载地址-resolved、依赖关系-requires、哈希值-integrity等等

作用：

1. 更加具体地描述依赖包的各种信息，保证每次安装依赖都完全相同
   1. npm 的 dependencies 由于存在修饰符，对依赖包的版本不完全固定，可能导致自动升级包的次版本或补丁版本，造成版本问题
2. 提升下载速度，不需要再查找包的信息

---



## Package 发布

### 带有 scope 作用域的发布

scope 为作用域，以 `@` 开头的包均视为带有作用域的包

scope 需要在 npm 建立 organization，名字即为 scope 的名字

在项目根目录创建 `.npmrc`

```
@scopeName:registry=https://registry.npmjs.org/
```

使用 npx lerna init 初始化项目，在 `package.json` 中设置 

```json
"workspaces": [
    "packages/*"
],
```

之后创建 packages 文件夹，在里面再新建包对应的文件夹，并配置包的名称

文件结构如下

```
- packages/
	- core/
		- src/
			- index.ts
		- package.json
		- rollup.config.js
		- tsconfig.json
- .npmrc
- lerna.json
- package.json
```

发布：

1. `lerna run build`
2. `lerna publish `

