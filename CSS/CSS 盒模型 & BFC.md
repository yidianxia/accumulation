## CSS 盒模型

**基本概念**

- 由 content、 padding、 border 和 margin 组成。
- 标准盒模型：width 和 height 指的是内容区域的宽度和高度，增加padding、border 和 margin 不会影响内容区域（content）的尺寸，但会增加元素的总尺寸。box-sizing: content-box;
- IE 盒模型： width 和 height 指的是内容区域 + border + padding 的宽度和高度。box-sizing: border-box;

**js 获取盒模型的宽高**

- 通过 DOM 节点的 style 样式获取，但只能获取行内样式

```
element.style.width/height
```

- 通用型。兼容Chrome 和 火狐

```
window.getComputedStyle(element).width/height
```

- IE 独有

```
element.currentStyle.width/height
```

- 获取一个元素的绝对位置。绝对位置是视窗 viewport 左上角的绝对位置。

```
element.getBoundingClientRect().width/height;
```

**margin 塌陷/ margin重叠**

标准文档流中，竖直方向的 margin 不叠加，只取较大的值作为 margin (水平方向的 margin 是可以叠加的，即水平方向没有塌陷现象)。

两个盒子是父子元素关系且 CSS 是这样的情况时 （margin 塌陷）：

```html
<div class="father">
    <div class="son"></div>
</div>
<style>
/* 给儿子设置margin-top为10像素 */
.son {
    height: 100px;
    margin-top: 10px;
}
</style>
```

此时父元素的 height 是100 而不是110，因为儿子和父亲在竖直方向上，共一个margin。

margin这个属性，本质上描述的是兄弟和兄弟之间的距离； 最好不要用这个marign表达父子之间的距离。

如果我们给父亲设置一个属性：`overflow: hidden`，就可以避免这个问题，此时父亲的高度是110px，这个用到的就是BFC。

## BFC

BFC（Block Formatting Context）：块级格式化上下文。

**BFC 渲染规则**

- 内部的盒会在垂直方向一个接一个排列
- BFC 内部的子元素，在垂直方向，**边距会发生重叠**。

- BFC在页面中是独立的容器，外面的元素不会影响里面的元素，反之亦然。
- BFC区域不与旁边的 float box 区域重叠。
- 计算BFC的高度时，浮动的子元素也参与计算。

**如何生成 BFC**

- overflow: 不为 visible，可以让属性是 hidden、auto。【最常用】
- float 的属性值不为 none。意思是，只要设置了浮动，当前元素就创建了 BFC。

- 只要 posiiton 的值不是 static 或者是 relative 即可，可以是`absolute`或`fixed`，也就生成了一个BFC。

- display 为 inline-block, table-cell, table-caption, flex, inline-flex。

**应用**

- 解决 margin 塌陷。当父元素和子元素发生 margin 塌陷给**子元素或父元素创建BFC**。
- 解决 margin 重叠。
- 清除浮动。父元素没有设置高度 而 子元素浮动时，设置成 BFC 可以让浮动元素参与计算高度。





































