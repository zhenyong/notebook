flex 杂记
---

[CSS Flexible Box Layout - CSS: Cascading Style Sheets | MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flexible_Box_Layout)

## container （默认 row）

align-content 多行，纵轴方向布局（对 flex-wrap:nowrap 无效）
space-between / space-around / space-evenly 区别: [圖解 Flexbox 基本屬性 | Summer。桑莫。夏天](https://cythilya.github.io/2017/04/04/flexbox-basics/)
在任意属性值后面跟上 safe 可以确保元素不会溢出容器

align-items 单行
align-self 当 align-items 是对容器所有元素在 cross axis 上的布局，align-self 则设置某个 item

xx-items 作用在容器内所有元素上
xx-content 作用在容器上，所有元素当作整体
xx-self 作用在具体单个元素
三类的区别参考里面的图 [css3 - The difference between justify-self, justify-items and justify-content in CSS Grid - Stack Overflow](https://stackoverflow.com/questions/48535585/the-difference-between-justify-self-justify-items-and-justify-content-in-css-gr?answertab=active#tab-top)


justify-content 主轴元素排列

## item

flex = flex-grow flex-shrink flex-basis

flex-basis 设置了绝对值，则优先于主轴方向的尺寸 （width / height），决定盒子模型 content-box 的尺寸
[前端 - flex设置成1和auto有什么区别 - SegmentFault 思否](https://segmentfault.com/q/1010000004080910)

flex-flow = flex-direction flex-wrap


