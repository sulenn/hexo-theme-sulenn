---
title: drizzle 和 react 学习
cover: ../../../../img/contact-bg.jpg
author: sulenn
date: 2019-01-30 10:45:59
tags: 
    - ethereum
    - drizzle
    - react
    - solidity
    - html
    - javascript
categories: blockchain
---

# GETTING STARTED WITH DRIZZLE AND REACT

本文翻译至：[https://truffleframework.com/tutorials/getting-started-with-drizzle-and-react](https://truffleframework.com/tutorials/getting-started-with-drizzle-and-react) 版本为：2018-08-28

Drizzle 是 Truffle Suite 的最新成员，也是我们的第一个前端开发工具。 Drizzle 的核心是将合约数据和交易数据等内容从区块链同步至 Redux store。 `drizzle` 基础库顶部有更高级别的抽象; 用于React兼容性 （[drizzle-react](https://truffleframework.com/docs/drizzle/react/react-integration)） 和一组即用型 React 组件 （[drizzle-react-components](https://truffleframework.com/docs/drizzle/react/react-components)） 的工具。

今天我们专注于初级应用，带你从头开始用 React 和 Drizzle 设置 Truffle 项目。通过这种方式，我们可以更好地了解 Drizzle 在应用中如何使用。有了这些知识，您可以利用您选择的任何前端框架充分利用 Drizzle，或者放心使用更高级别的React抽象

这将是一个非常初级的教程，专注于设置和获取存储在合约中的简单字符串。它适用于具有 Truffle 基础知识的人，此外还需要对 JavaScript 和 React.js 有一定的了解，但是不太熟悉 Drizzle 。

> 注意：了解 Truffle 基础，可以参考 [Pet Shop](https://truffleframework.com/tutorials/pet-shop) 教程

本教程将覆盖：

1. 设置开发环境

2. 从头开始创建 Truffle 项目

3. 编写智能合约

4. 编译和移植智能合约

5. 测试智能合约

6. 创建 React.js 项目

7. 设置前端客户端

8. 用 Drizzle 连接 React 应用程序

9. 写一个从 Drizzle 中读取的组件

10. 写一个写入智能合约的组件

## 1. 设置开发环境

开始之前有一些技术要求。请安装以下内容：

- [Node.js v8+ LTS and npm](https://nodejs.org/en/) (comes with Node)

### 1.1 Truffle

安装如上内容后，安装 Truffle：

```shell
npm install -g truffle
```

验证 Truffle 是否已正确安装，在终端上输入 `truffle version` 。如果发现错误，请检查是否已将npm模块添加到路径中。

### 1.2 Ganache-CLI

我们还将使用 Ganache-CLI ，这是一个用于以太坊开发的个人区块链，可用于部署合约、开发应用程序和运行测试。 可以使用以下命令全局安装：

```shell
npm install -g ganache-cli
```

这里如果出现非全局安装问题，可使用 `ln -s` 将 ganache-cli 链接至 `/usr/local/bin/` 。详细使用方法请 google

### 1.3 Create-React-App

最后，由于这是一个 React.js 教程，我们将使用 [Create-React-App](https://github.com/facebook/create-react-app/) 创建我们的React项目

如果你拥有 NPM 5.2 或更高版本，则无需执行任何操作。你可以通过运行 `npm --version` 来检查 `NPM` 版本。如果没有，则需要使用以下命令全局安装该工具：

```shell
npm install -g create-react-app
```

## 2. 创建 Truffle 项目

1. Truffle 在当前目录中初始化，因此首先在你选择的开发文件夹中创建一个目录，然后移动至目录

    ```shell
    mkdir drizzle-react-tutorial
    cd drizzle-react-tutorial
    ```

2. 运行如下命令生成空 Truffle 项目

    ```shell
    truffle init
    ```

简单看一下生成的目录结构

### 2.1 目录结构

默认的目录结构包含如下：

- `contracts/` : 包含我们智能合约的 [Solidity](https://solidity.readthedocs.io/) 源文件。 这里有一个名为 `Migrations.sol` 的重要合约，我们稍后会讨论。

- `migrations/` ：`Truffle` 使用移植系统来处理智能合约部署。 移植是一种特殊的智能合约，可以跟踪合约变化。

- `test/` ：包含智能合约 JavaScript 和 Solidity 测试

- `truffle-config.js` ：Truffle 配置文件

## 3. 编写智能合约

我们将添加一个名为 MyStringStore 的简单智能合约。

1. 在 `contracts/` 目录中创建名为 `MyStringStore.sol` 的新文件。

2. 文件中添加如下内容：

    ```solidity
    pragma solidity ^0.5.0;

    contract MyStringStore {
    string public myString = "Hello World";

    function set(string memory x) public {
        myString = x;
    }
    }
    ```

由于这不是Solidity教程，所以您需要了解的是：

- 我们创建了一个名为 `myString` 的公共字符串变量，并将其初始化为 “Hello World” 。 这会自动创建一个 getter方法（因为它是一个公共变量），所以我们不必自己编写一个。

- 我们创建了一个 setter 方法，将 `myString` 变量设置为传入的任何字符串。

**注意：**

这里需要在 set 函数传入参数 x 中添加 memory 属性，不然可能会在 4.1 编译部分出现如下错误（应该是 solidity 版本的问题）：

![3](http://ww1.sinaimg.cn/large/006alGmrgy1fzojk8nckuj30k405maaq.jpg)

### 3.1 运行测试链

在我们继续操作之前，让我们首先使用 Ganache-CLI 启动我们的测试区块链。

打开一个新终端并运行以下命令：

```shell
ganache-cli -b 3
```

这将生成一个新的区块链，默认情况下会侦听127.0.0.1:8545，并将每3秒进行一次挖矿。 如果我们没有指定这个，Ganache会立即挖矿，我们将无法模拟真正的区块链开采所需的延迟。

保持终端窗口打开，您可以观察稍后与之交互时发生的情况

### 3.2 指定网络

我们需要让我们的 Truffle 项目知道如何连接到这个区块链。 要做到这一点，我们需要将以下内容放在 `truffle-config.js中` ：

```js
module.exports = {
  networks: {
    development: {
      host: "localhost",
      port: 8545,
      network_id: "*" // Match any network id
    }
  }
};
```

## 4. 编译和移植智能合约

现在我们准备编译和移植合约

### 4.1 编译

1. 在终端中，确保您位于项目目录的根目录中并输入：

    ```shell
    truffle compile
    ```

> 如果您使用的是 Windows 并且在运行此命令时遇到问题，请参阅文档 [resolving naming conflicts on Windows](https://truffleframework.com/docs/advanced/configuration#resolving-naming-conflicts-on-windows)

您应该看到类似于以下输出内容：

```shell
Compiling ./contracts/Migrations.sol...
Compiling ./contracts/MyStringStore.sol...
Writing artifacts to ./build/contracts
```

### 4.2 移植

现在我们已经成功编译了合约，是时候将它们移植到区块链了！

> 注意：阅读更多关于移植 [Truffle documentation](https://truffleframework.com/docs/getting_started/migrations)

创建移植脚本

1. 在 `migrations/` 目录中创建名为 `2_deploy_contracts.js` 的新文件

2. 文件中添加如下内容：

    ```js
    const MyStringStore = artifacts.require("MyStringStore");

    module.exports = function(deployer) {
    deployer.deploy(MyStringStore);
    };
    ```

3. 返回终端，移植合约到区块链

    ```shell
    truffle migrate
    ```

你可以看到如下类似输出：

```shell
Using network 'development'.

Running migration: 1_initial_migration.js
  Deploying Migrations...
  ... 0xcc1a5aea7c0a8257ba3ae366b83af2d257d73a5772e84393b0576065bf24aedf
  Migrations: 0x8cdaf0cd259887258bc13a92c0a6da92698644c0
Saving successful migration to network...
  ... 0xd7bc86d31bee32fa3988f1c1eabce403a1b5d570340a3a9cdba53a472ee8c956
Saving artifacts...
Running migration: 2_deploy_contracts.js
  Deploying MyStringStore...
  ... 0x43b6a6888c90c38568d4f9ea494b9e2a22f55e506a8197938fb1bb6e5eaa5d34
  MyStringStore: 0x345ca3e014aaf5dca488057592ee47305d9b3e10
Saving successful migration to network...
  ... 0xf36163615f41ef7ed8f4a8f192149a0bf633fe1a2398ce001bf44c43dc7bdda0
Saving artifacts...
```

您可以按顺序查看正在执行的移植，然后是每个已部署合同的区块链地址（您的地址将有所不同）。

## 5. 测试智能合约

在我们继续之前，我们应该编写几个测试来确保合约按预期工作。

1. 在 `test/` 目录中创建一个名为 `MyStringStore.js` 的新文件。

2. 将以下内容添加到 `MyStringStore.js` 文件：

    ```js
    const MyStringStore = artifacts.require("./MyStringStore.sol");

    contract("MyStringStore", accounts => {
    it("should store the string 'Hey there!'", async () => {
        const myStringStore = await MyStringStore.deployed();

        // Set myString to "Hey there!"
        await myStringStore.set("Hey there!", { from: accounts[0] });

        // Get myString from public variable getter
        const storedString = await myStringStore.myString.call();

        assert.equal(storedString, "Hey there!", "The string was not stored");
    });
    });
    ```

### 5.1 运行测试

1. 返回终端，运行测试：

    ```shell
    truffle test
    ```
2. 如果所有测试都通过，您将在控制台看到与此类似的输出：

    ```shell
    Using network 'development'.

    Contract: MyStringStore
        ✓ should store the value 'Hey there!' (3085ms)

    1 passing (3s)
    ```

真棒！ 现在我们知道合同确实有效。

## 6. 创建 React.js 项目

现在我们完成了智能合约，我们可以用 React.js 编写我们的前端客户端！ 为此，只需运行此命令（如果您有NPM 5.2或更高版本）：

```shell
npx create-react-app client
```

如果您有较旧版本的 NPM ，请确保按照 [Setting up the development environment](https://truffleframework.com/tutorials/getting-started-with-drizzle-and-react#create-react-app) 部分中的说明全局安装 Create-React-App ，然后运行以下命令：

```shell
create-react-app client
```

这应该在您的 Truffle 项目中创建一个 `client` 目录，并生成一个裸的  React.js 项目，让你开始构建您的前端。

## 7. 设置前端客户端

现在我们在 `client` 目录中有一个前端客户端，使用命令 `cd client` 切换到该目录，然后继续执行以下步骤进行设置。

### 7.1 链接编译文件

由于 Create-React-App 默认不允许从 `src` 文件夹外部导入文件，因此我们需要将我们 `build` 文件夹中的 contracts 放入 `src` 中。 我们可以在每次编译合约时复制和粘贴一次，但更好的方法是创建符号链接。

如果你之前没有创建过符号链接，请将其视为文件系统中的一个 magical portal 

切换到 `src` 目录，然后创建符号链接文件夹：

```shell
// For MacOS and Linux

cd src
ln -s ../../build/contracts contracts

// For Windows 7, 8 and 10
// Using a Command Prompt as Admin

cd src
mklink /D contracts ..\..\build\contracts
```

实际上，这应该在 `src` 中创建看起来像 `contract` 文件夹的内容，但它实际上指向我们的 Truffle 项目的 `build/contracts` 文件夹中的文件。

### 7.2 安装 Drizzle

这是最有趣的部分，我们安装 Drizzle。 使用命令 `cd ..` 更改回 `client` 目录，然后运行以下命令：

```shell
npm install drizzle
```

这就是依赖关系！ 请注意，我们不需要自己安装 Web3.js 或 Truffle-Contract 。 Drizzle 包含我们与智能合约交互所需的一切。

## 8. 用 Drizzle 连接 React 应用程序

在我们进一步讨论之前，让我们通过在 `client` 目录中运行如下命令来启动我们的 React 应用程序：

```shell
npm start
```

这将在 `localhost：3000` 下启动前端服务，因此请在浏览器中打开它

> 注意：如果已经安装了 MetaMask，请确保使用隐身窗口（或暂时禁用 MetaMask ）。 否则，应用程序将尝试使用 MetaMask 中指定的网络，而不是 localhost：8545 下的开发网络。

如果加载的默认 Create-React-App 页面没有任何问题，你可以继续。
