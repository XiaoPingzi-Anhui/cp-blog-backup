<!-- category: "浏览器"
labels: "浏览器"
createdAt: 2023-04-14T10:29:08.623+00:00 -->

 2023年2月14日情人节，IE 终于死了，前端们的头发终于可以少掉点了，但是！Safari 怎么还在啊！你能不能跟上 IE 的脚步，别再折磨人了，下面是我平时开发过程中，Safari 适配踩的一些坑，做个总结：
### css：
1. 元素没有滚动条？是因为高度不对？高度没继承成功？查查是不是某个父级没有手动指定高度，给加个 height:100% 或者其他的值，safari 元素不手动指定高度，子元素是继承不了的；
2. position: sticky 在 safari 不生效？试试加上一条 position: -webkit-sticky; 而且滚动高度超过其父元素高度，不管父元素是否可见，都不再粘滞？
3. 给具有 border-radius 属性的元素加阴影，但是阴影在 safari 中没有 圆角？试试给元素添加一个可创建层叠上下文的 CSS 属性。比如 transform: scale(1)、isolation: isolate、position: relative; z-index: 0;
4. overflow: overlay 的滚动条，在 chrome 是不占位置的，但是在 safari 中，是占位置的，表现与 auto 相同，火狐浏览器则不支持 overlay，这也是为什么很多地方写overflow: auto; overflow: overlay; 两条属性；
5. 尽量别用新属性，例如 justify-content 不支持start/end,可用 flex-(start/end) 或者left/right 代替；

### js：
1. 不支持正则的零宽断言，用了就会报错："Invalid regular expression:invalid group specifier name",考虑换其他方法实现吧
2. 不支持 YYYY-MM-DD HH:mm:ss 日期格式，如果你在Safari 执行 new Date('2023-04-14 12:00:00')，结果会是 Invalid Date,说到时间格式，推荐 DayJs,最好用的时间库