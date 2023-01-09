<!-- category: "antd"
labels: "antd"
createdAt: 2022-11-12T19:01:31.832+00:00 -->
最近需要对antd的Table展开行做一些样式处理，查看文档，发现有一个expandable中有个expandedRowClassName，就直接拿过来用，结果就一直没生效，展开行的类名始终加不上去，但从文档上看不出什么问题，最后经过排查才发现，非expandedRowRender渲染的展开行不会生效，也就是不自定义渲染就是不会生效，真是服了，这么重要的事文档上面是一个字都不提啊！而我又不需要用expandedRowRender，最后我解决方法还是用Table的rowClassName，给展开行数据加一个展开标识，rowClassName里检查到这个展开标识后就返回展开行需要的类名，否则就不返回，这样就解决了我的问题。但是u1s1，antd的文档真是不全呀，很多信息不完整。