<!-- category: "git"
labels: "git"
createdAt: 2022-12-05T16:31:11.832+00:00 -->
今天更新了一个许久未更新的仓库，运行却发现一些报错，经排查发现是某一个文件夹的名称被人修改过了，原先文件夹名称全是小写，现在远端改成了首字母大写，但是我本地明明pull过了，但还是小写，才导致了运行报错，很奇怪呀，这样很不友好，难道是git的一个bug?不应该有这么大的bug。一查才发现，原来git是否对大小写敏感是可配置的，可以通过`git config --get core.ignorecase`查看，true的话是不敏感，可以通过`git config core.ignorecase false`配置成敏感，这样问题就解决啦！