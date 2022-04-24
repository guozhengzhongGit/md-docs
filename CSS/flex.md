# flex 原理

> 不管在实际工作还是面试场景中 Flex 作为 CSS3 新特性之一出场率都很高，总结一下

## Flex 容器

设置了 display: flex 或者 inline-flex 的元素称为容器，容器内部的元素称为项目，容器有主轴和交叉轴，作用于容器的有六个属性:

1. flex-direction：决定主轴方向row和column，起点在左还是右

2. flex-wrap：主轴排列不下项目时是否换行显示，默认不换行，会缩小项目占据的主轴空间尺寸去适配

3. flex-flow：是前面两个的简写，一般不用，还是推荐分开写

4. justify-content 项目在主轴上的对齐方式

5. align-items 项目在交叉轴上的对齐方式，默认 stretch，铺满交叉轴的空间，可以很轻松实现多列等高

6. align-content 存在多跟轴线时的对齐方式

## Flex 项目

- 项目上也可设置六个属性：

1. order：定义项目在容器中的排列顺序，数字越小越靠前

2. flex-grow：若存在剩余空间，各项目将按这个比例分配剩余空间放大。默认值为 0，即即使有剩余空间，也不放大

3. flex-shrink：缩小比例。若主轴空间不足，则按照这个比例进行缩小，默认值为 1，负值无效

4. flex-basis：在分配剩余空间之前项目占据的主轴空间，浏览器根据这个值判断主轴是否存在剩余空间。默认值为 auto，即项目本身的大小

5. flex：flex-grow和flex-shrink以及flex-basis的简写(**重点**)

- 三个属性都取默认值就是flex的默认值即 0 1 auto

- auto 代表 1 1 auto

- none 代表 0 0 auto

- 非负数字，是 flex-grow 的值即 flex: 1 代表 1 1 0%

- 0 代表 0 1 0%

- flex 取值为长度或百分比，则为 flex-basis 的值，即 flex: 0% 代表 1 1 0%

- flex 取值为两个非负数字，则分别为 flex-grow 和 flex-shrink 的值即flex: 2 3 代表 2 3 0%

- flex 取值为一个非负数字和长度或百分比，则为 flex-grow 和 flex-basis 的值，即 flex: 1 0% 代表 1 1 0%

6. align-self 单独定义自己的对齐方式

## flex 计算公式

### 子元素溢出时

1. 计算溢出量

2. 根据各项目自身宽度乘以缩小比例求和计算总权重

3. 计算需要缩小的尺寸。溢出量乘以缩小比例乘以自身宽度再除以总权重

4. 使用自身宽度减去需要缩小的尺寸就是元素最终的尺寸

```css
.container {

display: flex;

width: 600px;

}

.left {

flex: 1 2 500px;

background: red;

}

.right {

flex: 2 1 400px;

background: green;

}

1. 溢出量为 500 + 400 - 600 = 300

2. 总权重为 2 * 500 + 1 * 400 = 1400

3. 左侧项目需要缩小的尺寸为 2 * 500 * 300 / 1400 = 214.28；右侧项目需要缩小的尺寸为 1 * 400 * 300 / 1400 = 85.72

4. 左侧项目最终尺寸为 500 - 214.28 = 285.72；右侧项目最终尺寸为 400 - 85.72 = 314.28
```

### 子元素放大时

1. 计算主轴剩余空间

2. 根据放大系数和剩余空间计算各项目可以放大的尺寸

3. 原尺寸加上放大尺寸

```css
.container {

display: flex;

width: 600px;

}

.left {

flex: 1 2 300px;

background: red;

}

.right {

flex: 2 1 200px;

background: green;

}

1. 主轴剩余空间为 600 - 300 - 200 = 100

2. left 可放大的空间为 100 * 1 / (2 + 1) = 33.33；right 可放大的空间为 100 * 2 / (2 + 1) = 66.67；

3. left 最终尺寸为 300 + 33.33 = 333.33；right 最终尺寸为 200 + 66.67 = 266.67
```

[效果在这里](https://codepen.io/bryance/pen/wvMKaGe)

