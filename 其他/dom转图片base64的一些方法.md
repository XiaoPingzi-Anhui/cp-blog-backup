<!-- category: "其他"
labels: "css,echarts,html2canvas"
createdAt: 2022-05-27T10:44:43.062+00:00 -->
根据dom元素生成图片的需求经常会碰到，方法有很多，根据具体的需求选择好方法往往事半功倍，下面就介绍几种我遇到过的方法。
## 1.echarts
如果你是要操作echarts图转成图片的话，那么用echarts提供的getDataURL() api就可以完成了，详情请看[官网](https://echarts.apache.org/zh/api.html#echartsInstance)，大概用法如下：
```typescript
const img = new Image();
img.src = echartsInstance.getDataURL({
  // 导出的格式，可选 png, jpg, svg
  // 注意：png, jpg 只有在 canvas 渲染器的时候可使用，svg 只有在使用 svg 渲染器的时候可用
  type?: string,
  // 导出的图片分辨率比例，默认为 1。
  pixelRatio?: number,
  // 导出的图片背景色，默认使用 option 里的 backgroundColor
  backgroundColor?: string,
    // 忽略组件的列表，例如要忽略 toolbox 就是 ['toolbox']
  excludeComponents?: Array.<string>
});
```
如果大家在使用的时候发现生成的图片不全，可以延时执行getDataURL()方法，设个300ms，还有一种方法就是取消图表里面动画效果，就是在 option 中添加一个  animation: false 属性就可以解决这个问题，但是一般不太推荐这个方法。
## 2.canvas
如果不是echarts,那么一般的方法是通过canvas来做的，可以自己写，也可以使用现成的轮子，其基本原理就是把dom复制一份绘制到canvas中，绘制方法：canvas.drawImage(),再通过canvas.toDataURL()转成base64，一个简单的小栗子：
```typescript
let canvasList = document.getElementById('id');
let canvas = document.createElement('canvas');
let ctx = canvas.getContext('2d');
canvas.width = 400;
canvas.height = 400;
if (ctx) {
  ctx.rect(0, 0, canvas.width, canvas.height);
  ctx.fillStyle = 'white';
  ctx.fill();
  ctx.drawImage(canvas, 10, 10);
  const base64 = canvas.toDataURL('image/jpeg');      
}
```
## 3.html2canvas
大佬们写的轮子方便我这个菜狗直接用，html2canvas就是一个很好地库，详情可以去看[官方文档](http://html2canvas.hertzen.com/)

将某个dom转成图片下载到本地
```typescript
function downloadPNG(moduleName, id) {
  const realHtml = document.getElementById(id); // 需要截图的包裹的（原生的）DOM 对象
  const width = realHtml.offsetWidth; // 获取dom 宽度
  const height = realHtml.offsetHeight; // 获取dom 高度
  const canvas = document.createElement('canvas'); // 创建一个canvas节点
  const scale = 1; // 定义任意放大倍数 支持小数
  canvas.width = width * scale; // 定义canvas 宽度 * 缩放
  canvas.height = height * scale; // 定义canvas 高度 *缩放
  canvas.getContext('2d').scale(scale, scale); // 获取context,设置scale
  const opts = {
    taintTest: true, // 检测每张图片都已经加载完成
    scale: scale, // 添加的scale 参数
    useCORS: true, // 跨域设置
    canvas: canvas, // 自定义 canvas
    logging: true, // 日志开关
    width: width, // dom 原始宽度
    height: height, // dom 原始高度
  };
  html2canvas(realHtml, opts).then((canvas) => {
    try {
      canvas.toBlob(function (blob) {
        saveAs(blob, `${moduleName}.png`);
      }, 'image/png');
    } catch (e) {}
  });
}
```
## 4.dom-to-image
dom-to-image也是一个很不错的第三方库，但是我没有用过，看其npm介绍感觉使用还是比较简单，功能比较全面的，[文档](https://www.npmjs.com/package/dom-to-image)