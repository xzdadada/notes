# NPM Package

## 版本管理

在 package.json 的 dependencies 和 devDependencies 中，以 `packageName: version` 的键值对形式指定依赖项和对应版本



### 关于 version：

version 为 `x.y.z` 的格式，x 为主版本号 major，y 为次版本号 minor，z 为补丁版本 patch

1. 固定主版本号：`^x.y.z`，允许自动更新次版本、补丁版本，即 $x.y.z <= v < x+1.0.0$
2. 固定次版本号：`~x.y.z`，允许自动更新补丁版本，即 $x.y.z <= v < x.y+1.0$
3. 完全固定版本号：`x.y.z`

`"workspace:*"`：表明当前引用的包为工作区内部的包，在 npm 安装依赖的时候，直接使用工作区的包，而不是从注册表中进行下载