# CSS快速复习





## 三种重要的定位属性

### position

| 值                                                           | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [absolute](https://www.runoob.com/css/css-positioning.html#position-absolute) | 生成绝对定位的元素，相对于 static 定位以外的第一个父元素进行定位。元素的位置通过 "left", "top", "right" 以及 "bottom" 属性进行规定。 |
| [fixed](https://www.runoob.com/css/css-positioning.html#position-fixed) | 生成固定定位的元素，相对于浏览器窗口进行定位。元素的位置通过 "left", "top", "right" 以及 "bottom" 属性进行规定。 |
| [relative](https://www.runoob.com/css/css-positioning.html#position-relative) | 生成相对定位的元素，相对于其正常位置进行定位。因此，"left:20" 会向元素的 LEFT 位置添加 20 像素。 |
| [static](https://www.runoob.com/css/css-positioning.html#position-static) | 默认值。没有定位，元素出现在正常的流中（忽略 top, bottom, left, right 或者 z-index 声明）。 |
| [sticky](https://www.runoob.com/css/css-positioning.html#position-sticky) | 粘性定位，该定位基于用户滚动的位置。它的行为就像 position:relative; 而当页面滚动超出目标区域时，它的表现就像 position:fixed;，它会固定在目标位置。**注意:** Internet Explorer, Edge 15 及更早 IE 版本不支持 sticky 定位。 Safari 需要使用 -webkit- prefix (查看以下实例)。 |
| inherit                                                      | 规定应该从父元素继承 position 属性的值。                     |
| initial                                                      | 设置该属性为默认值，详情查看 [CSS initial 关键字](https://www.runoob.com/cssref/css-initial.html)。 |









## uniapp 之中的CSS

1. 不支持`*`全局选择器

2. 仅仅支持类选择器

3. APP端不支持`rpx`单位

4. 支持的CSS选择器：

    1. 类选择器
    2. id选择器
    3. 组件选择器

    其他不建议使用





