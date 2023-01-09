<!-- category: "优秀的工具(库)"
labels: "node,nvm"
createdAt: 2022-07-09T09:16:12.525+00:00 -->
在开发中，做不同的项目可能需要使用到不同的node版本，刚开始切换版本时只能将旧版本卸载然后安装目标版本，这样真的很麻烦。一个小伙伴给我推荐了nvm工具，真香！这里记录一些常用的命令供查阅。详情移步[官方文档](http://nvm.uihtm.com/)
```typescript
nvm arch：显示node是运行在32位还是64位。
nvm install <version> [arch] ：安装node， version是特定版本也可以是最新稳定版本latest。可选参数arch指定安装32位还是64位版本，默认是系统位数。可以添加--insecure绕过远程服务器的SSL。
nvm list [available] ：显示已安装的列表。可选参数available，显示可安装的所有版本。list可简化为ls。
nvm on ：开启node.js版本管理。
nvm off ：关闭node.js版本管理。
nvm proxy [url] ：设置下载代理。不加可选参数url，显示当前代理。将url设置为none则移除代理。
nvm node_mirror [url] ：设置node镜像。默认是https://nodejs.org/dist/。如果不写url，则使用默认url。设置后可至安装目录settings.txt文件查看，也可直接在该文件操作。
nvm npm_mirror [url] ：设置npm镜像。https://github.com/npm/cli/archive/。如果不写url，则使用默认url。设置后可至安装目录settings.txt文件查看，也可直接在该文件操作。
nvm uninstall <version> ：卸载指定版本node。
nvm use [version] [arch] ：使用制定版本node。可指定32/64位。
nvm root [path] ：设置存储不同版本node的目录。如果未设置，默认使用当前目录。
nvm version ：显示nvm版本。version可简化为v。
```
其实还有一个主流的node版本管理工具叫n，nvm与n的异同点可以看看简书上的这篇[文章](https://www.jianshu.com/p/faa3e32055ce)。对于我目前来说，nvm已经满足我的需求了，也就没有研究n了。