<!-- category: "包管理工具"
labels: "bin,npm"
createdAt: 2023-03-22T19:51:21.107+00:00 -->

一些 npm 包经常还带有命令行操作，大家用的不少，例如 `webpack -v` 等等，那么这到底如何实现的呢？今天自己实现一个最最简单的demo，定义自己的命令。

## 创建npm项目
找个地方新建个文件夹，然后执行 `npm init -y`,初始化一个最简单的 npm 项目

## 加 bin 配置
在 package.json 中，添加 bin 可执行命令。之后你的文件应该如下：
```ts
{
  "name": "command-test", // 你自己的项目名称
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "bin": {
    "CTest": "./bin/index.js" // 命令及对应的执行文件，注意，命令不要和系统的命令重复了，系统的命令优先级比较高！！！，最好也不要和其他库重复
  },
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}

```

## 增加可执行文件
创建执行文件 bin/index.js，内容如下：
```ts
#!/usr/bin/env node
process.argv[2] === "-v" && console.log("1.0.0");
process.argv[2] === "-help" && console.log("SOS");
```
`#!/usr/bin/env node` 这句话是告诉你的电脑，当执行这个文件的时候，使用 nodeJS 环境来执行,process.argv 可以用来获取命令行携带的参数，有一些库可以方便获取参数，这里就不介绍了。

## 链接到全局环境
如果你不准备发包，那么就执行 `npm link`,执行成功后就可以在命令行执行 `CTest -v`、`CTest -help` 看看了。如果你打开你的 node 安装目录，也会发现 CTest 相关的可执行文件。如果你将这个包发到了 npm 仓库，别人安装后也可以执行你的命令了。