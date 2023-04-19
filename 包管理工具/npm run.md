<!-- category: "包管理工具"
labels: "npm"
createdAt: 2023-04-10T20:11:21.107+00:00 -->

当我们使用 npm 来管理项目时，可以在 package.json 里的 script 字段里配置各种执行脚本命令，然后可以通过 `npm run xxx` 的命令来运行。
## 执行原理

例如：
```ts
  "scripts": {
    "build": "prisma generate && max build",
    "dev": "max dev",
    "format": "prettier --cache --write .",
    "postinstall": "max setup",
    "prepare": "husky install",
    "setup": "max setup",
    "start": "npm run dev"
  },
```
当执行 `npm run dev` 时，执行的其实是 dev 对应的 `max dev` 命令，那为啥不直接执行 `max dev` 呢？因为 @umijs/max 没有全局安装的话，系统是无法识别 max
命令的。而使用 `npm run script` 执行脚本的时候都会创建一个 shell，然后在 shell 中执行指定的脚本。

当我们下载完依赖后，打开 node_modules 目录，可以发现其中有一个特殊的 /.bin 目录，这个目录并不是任何 npm 包，而是各个模块可执行文件的软链接，真正指向的就是第三方依赖 package.json 里面的 bin 配置指向的那个文件。其余 node_modules 目录下的文件夹才是一个个下载下来的依赖模块。一般针对一个依赖模块，会有3个可执行文件（如果该模块配置了 bin 的话），没有后缀名的是对应 Unix 系的 shell 脚本，.cmd 为后缀名的是 windows bat 脚本，.ps1 为后缀名的则是 PowerShell 中可执行文件（可以跨平台），三者作用都是用 node 执行一个 js 文件。

好了，现在来大概捋一下流程：
1. 执行 `npm run xxx`,会先找到 package.json 的 scripts 对应的 xxx 命令配置，会创建一个 shell 执行对应的命令。
2. 这个 shell 会将当前项目的可执行依赖目录（即node_modules/.bin）添加到环境变量 path 中，并根据 xxx 找到对应模块 A 的可执行文件软链接。
3. 通过 模块 A 的 package.json 的 bin 配置找到真正的可执行文件，并运行该文件。

## 多任务执行顺序
一个脚本可以执行多个任务：
1. & 并行执行，多个任务是同时执行的:
  ```ts
    "dev":"node test.js & webpack"
  ```
2. && 串行顺序，多个任务按顺序执行，前面执行结束才可以执行后面:
  ```ts
    "dev":"node test.js & webpack"
  ```

## 顺序钩子
除了手动用 & 或 && 控制多任务执行顺序，npm 还自带两个顺序钩子，pre 和 post：
```ts
  "predev":"node test_one.js",
  "dev":"node test_two.js",
  "postdev":"node test_three.js"
```
当执行 `npm run dev` 的时候默认就会执行 `npm run predev && npm run dev && npm run postdev`,及自动按前后顺序串行执行。

## 获取当前运行的脚本名称
npm 提供一个 npm_lifecycle_event 变量，返回当前正在运行的脚本名称，可以配合顺序钩子使用：
```ts
  const target = process.env.npm_lifecycle_event;
  if(target === 'predev') console.log('the process is predev');  
  if(target === 'dev') console.log('the process is dev');
  if(target === 'postdev') console.log('this process is postdev');
```

## 简写命令
npm run 还可以简写，省略 run：
```ts
  npm start === npm run start
  npm stop === npm run stop
  npm test === npm run test
  npm restart === npm run stop && npm run restart && npm run start
```
## 获取 package.json 内部变量
例如 在执行文件内，可以通过 console.log(process.env.npm_package_version)，来获取 package.json 中的 version 字段，现在知道各种 -v 命令如何打印出来的吧。