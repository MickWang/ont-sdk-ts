<p align="center">
  <img 
    src="https://ont.io/static/img/firstpagelogo.b81628b.jpg"
    width="125px"
    alt="Ontology logo">
</p>

<h1 align="center">ont-sdk-ts</h1>

## Overview

This is Ontology Wallet's TS(JS) SDK for Ontology BlockChain. Visit the [docs](./docs/howtouse.md).

## Installation

从NPM 系统下载安装。

````
npm install 'Ont-sdk-ts'
````

从github下载

```
git clone ''
```

### 编译

从github下载的源码，进入项目根目录，运行:

````
npm run build
````

你会得到编译后的代码在lib文件夹下。

### 测试

项目的测试代码在根目录```/test``` 文件夹下。运行：

```
npm run test
```

你会看到jest测试结果。

### 引用

#### Import

模块通过Ont导出。

```
import {Wallet} from 'Ont'
```

#### Require

Sdk 使用ES6的模块解决方案，```require``` 需要明确声明你所需的模块。

````
var Ont = require 'Ont'
var wallet = Ont.Wallet()
````

### Web

将编译后得到的lib文件夹下```browser.js```文件引入到页面中：

````
<script src="./lib/browser.js"></script>
````

在代码中使用需要在Ont的全局命名空间下。

```
var wallet = Ont.Wallet()
```


## Contribution

`ont-sdk-ts` always encourages community code contribution. Before contributing please read the [contributor guidelines](.github/CONTRIBUTING.md) and search the issue tracker as your issue may have already been discussed or fixed. To contribute, fork `ont-sdk-ts`, commit your changes and submit a pull request.

By contributing to `ont-sdk-ts`, you agree that your contributions will be licensed under its MIT license.

## License

* Open-source [MIT](LICENSE.md).
* Main author is [@lllwvlvwlll](https://github.com/lllwvlvwlll).
