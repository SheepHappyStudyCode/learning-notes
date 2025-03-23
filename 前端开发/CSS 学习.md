# CSS 学习

## 选择器的优先级

> 选择器的**特异度**越高，优先级越高

id > class > 标签



## 继承

某些属性会自动继承其父元素的计算值，除非显式指定一个值

例如文字的 `font-size`属性可以从父元素继承

继承关键字：`inherit`

初始值关键字：`initial`



## 布局相关技术

1. **常规流布局**（Normal Flow Layout）：正常布局流（normal flow）是指在不对页面进行任何布局控制时，**浏览器默认的 HTML 布局方式**。常规流布局包括**块级元素**（Block-level Elements）和**行内元素**（Inline Elements）的布局方式。

2. 现代布局技术：**弹性盒子（FlexBox）和栅格布局（Grid）**

3. 浮动：早期用来在文字里插入图片，现在很少用，被 FlexBox 和 Grid 布局取代。
4. 绝对定位：对于一些特殊的元素要靠设置 position 属性定位，尽量少用。



### 盒子模型

![image-20250127145707238](https://gitee.com/SheepHappyStudyCode/blog-pic/raw/master/20250127145714351.png)

1. 一般设置宽高是设置 content 内容的宽高，而不是整个盒子的宽高。

2. 如果想设置整个盒子的宽高，可以加上`box-sizing: border-box`属性（推荐，更符合人的直觉）



### 块级元素和行内元素

> 这两个都是常规流的概念

1. 块级元素（Block）：会占据父元素的整个宽度，一行只能放一个。

​	常见的块级元素：`<div>`、`<p>`、`<h1>`、`<section>`、`<article>`



2. 行内元素（Inline）：宽度由内容决定，一行能放多个。

​	常见的行内元素：`<span>`、`<a>`、`<strong>`、`<em>`



### Flex Box

是一种**一维**的布局模型。它给 flexbox 的子元素之间提供了强大的空间分布和对齐能力。



### Grid 布局

Flex 只能在一维上进行布局

Grid 可以在二维上进行布局

可以划分为一个一个网格，再布局



### 绝对定位

position 属性

1. static：默认属性，遵循常规流的布局（由上到下，由左到右）
2. relative：在 static 的基础上增加一些偏移量
3. absolute：在首屏绝对定位
4. fixed：在屏幕绝对定位
5. sticky：在显示范围内固定定位，超过显示范围后会黏在一个地方