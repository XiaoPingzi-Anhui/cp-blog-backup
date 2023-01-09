<!-- category: "antd"
labels: "antd"
createdAt: 2022-11-03T19:15:11.832+00:00 -->
近期有个需求，是表格折叠行的，需求是要在指定的几个列，点击一些图标进行展开收起操作。其实就是antd的树形结构，但是antd的table树形结构，其展开收起的图标是固定的，且默认在第一列里。4版本开始，可以通过expandable配置展开收起的相关配置。自定义图标可以通过expandIcon配置，但是这个图标在哪一列显示，我一直没弄懂怎么配，看文档是通过childrenColumnName属性，但我加了之后直接报错了，也没有找到相关的示例。
最后我是通过expandedRowKeys实现的，expandedRowKeys可以控制行的展开收起状态，其key对应的是数据行里的key属性。然后通过expandIcon: () => {}配置让默认的展开图标不显示。那自定义图标？不就随便自定义了，只要绑定操作expandedRowKeys的方法，任何dom不都可以做展开收起图标了！这样十分灵活，自由度非常高！