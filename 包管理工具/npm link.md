<!-- category: "包管理工具"
labels: "npx"
createdAt: 2023-02-20T11:20:51.107+00:00 -->

当我们需求开发自己的 npm 包时，肯定是需要在本地进行调试测试的，比如我要开发一个包 packageA,然后我在 projectA 中引入 packageA 使用，进行调试，问题是如何在 projectA 中
引用 packageA？我们项目中常用的是直接从远端的仓库 install 安装，但是本地调试如果这样做的话，packageA 修改了之后，还必须打包发布到远端仓库，再在 projectA 中更新 packageA,
才能调试更改内容，太麻烦！所以就有了 `npm link`。也就是说 projectA 引用的 packageA 不从远端 install，而是直接引用本地的 packageA。

## 链接步骤
链接主要分为两步（下文只介绍 window 下的情况）：

### 将 packageA link 到全局
在 packageA 项目的根目录下执行 `npm link` 命令，将 packageA 软链接到 node 全局安装目录下(可以使用 `npm root -g` 命令查看全局安装目录),这个命令执行是要等待一会的。等成功执行完了，就相当于你全局安装了 packageA，可以在其他项目使用，你可以在 node 的 node_modules 文件夹找到，但你全局安装的 packageA 其实只是一个软链接，链接到你本地真实的 packageA 路径。如果你修改了 packageA 的内容，那么在全局 node 的 node_modules 下，packageA 也会修改。

### 在 projectA 中使用
在 projectA 的根目录下执行 `npm link packageAName` ,这个命令执行很快。注意，这个命令中的 packageAName 是你 packageA 项目的 package.json 中的 name 配置哦。现在在 projectA 中的node_models 里就会添加一个 packageA 的软连接。就说明链接模块成功了。就可以愉快进行本地调试啦！

## 取消链接
若要在 projectA 中取消链接，直接在 projectA 的根目录下执行 `npm unlink packageAName`，若要取消 projectA 的全局链接，就在 projectA 的根目录下执行 `npm unlink`。

## 注意事项
如果按照上面的步骤 启动 projectA 报错，找不到 projectA，那么请检查你的 projectA 入口文件是否正确，是否没有打包？

## 更好的方案（npx link）
偶然发现另一个方案，另一个小工具[link](https://www.npmjs.com/package/link),可以解决一些 `npm link` 的问题。