<!-- category: "包管理工具"
labels: "npx"
createdAt: 2023-02-20T11:20:51.107+00:00 -->

使用脚手架创建项目模板时，经常能用到 npx 命令，今天突然很好奇，这东西到底干嘛的，就大概看了一下。
npx 是 npm5.2.0 版本新增的一个工具包， 最大的特性就是，npx 安装的包，会下载到一个临时目录，用完即删，不会占用本地资源。
以 cra 脚手架创建一个 react 项目为例，常规的做法是先安装 create-react-app，然后才能使用 create-react-app 执行命令进行项目创建，即：
```ts
npm i -g create-react-app
create-react-app my-react-app
```
分为两步，当然啦，第一步不用每次都安装 cra，只用执行一次就一直可用。如果我们使用 npx ，就可以省略安装 create-react-app 这一步。
```ts
npx create-react-app my-react-app
```
一步到位，这也是各个文档推荐的写法。npx 具体的步骤为：npx 会在当前目录下的 ./node_modules/.bin 里去查找是否有可执行的命令，没有找到的话再从全局里查找是否有安装对应的模块，全局也没有的话就会自动下载对应的模块，如上面的 create-react-app，npx 会将 create-react-app 下载到一个临时目录，用完即删，不会占用本地资源。

npx 也有一些命令参数：
1. --no-install
--no-install 告诉npx不要自动下载，也就意味着如果本地没有该模块则无法执行后续的命令。
2. --ignore-existing
--ignore-existing 告诉npx忽略本地已经存在的模块，每次都去执行下载操作，也就是每次都会下载安装临时模块并在用完后删除。
3. -p
-p 用于指定npx所要安装的模块，它可以指定某一个版本进行安装，还可以用于同时安装多个模块
4. -c
-c 告诉npx所有命令都用npx解释。