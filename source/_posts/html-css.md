---
title: html+css
date: 2016-12-16 10:11:08
categories: programming
tags: tips
---

### html 和 css 看到的一些容易忽略的东西

#### 1.行高

1.1行高的表示

```css
body{line-height:normal;}

body{line-height:inherit;}

body{line-height:120%;}

body{line-height:25px;} /*或em*/

body{line-height:1.2;}

body{font:100%/normal  arial;}

body{font:100%/120%  arial;}

body{font:100%/1.2  arial;}

body{font:100%/25px  arial;} 

/*行高为百分比，根据字体大小计算出实际行高，之后的子孙节点如果继承则按计算好的行高继承
行高为normal或者1.2数字时会根据font-size缩放
指南规定段落行高至少1.5倍*/
```

1.2box的类型

<!-- more -->

```html
<p>1<em>11</em>1111
  1111111111
</p>

<!--
p标签为containing box
文本和em为inline box，文本为匿名inline box 其高度会以行高为标准
inline box 在containing box 里面一个接一个组成了line box 一行一个 其高度取决于最大的那个inline box
content box 是围绕文字高度取决于font-size的box

-->
```

#### **2.[visual formatting model（视觉格式化模型）](https://www.w3.org/TR/CSS21/visuren.html#)**

> 记录着如何将doument tree转换为所见的media。
>
> 在视觉格式化模型中，文档树中的每一个元素会根据盒模型生成0个或多个盒子，这些盒子的布局由以下决定：
>
> - 盒子的尺寸和类型
> - 定位方案（[positioning scheme](https://www.w3.org/TR/CSS21/visuren.html#positioning-scheme)）：包括：标准文档流（normal flow）,浮动，绝对定位
> - 和文档树中元素之间的关系
> - 额外信息（如视口尺寸，图片的本身尺寸等）

##### 2.1.1block level element

> - display属性为block, list-item,table
>
> - 每一个块级元素生成一个principal块级盒子 （Block-level boxes 即BFC(block formatting context)的box）,如display为list-item的块级元素还会生成additional box
>
> - 除了表盒，所有的块级盒都是块容器(**Except for table boxes,** which are described in a later chapter, and replaced elements, **a block-level box is also a block container box).** 
>
> - 块容器要么只包含块级盒要么只包含内联级盒（**A block container box** either contains only block-level boxes or establishes an inline formatting context and thus contains only inline-level boxes）. 
>
>   ```html
>   <DIV>
>     Some text    <!--被强制转换为块级盒-->
>     <P>More text
>   </DIV>
>   ```
>
> - 如果一个内联盒包含块级盒，则其会在块级盒周围被分解为多个匿名块级盒，原来的内联盒将不存在。When an inline box contains an in-flow block-level box, the inline box (and its inline ancestors within the same line box) are broken around the block-level box (and any block-level siblings that are consecutive or separated only by collapsible whitespace and/or out-of-flow elements), splitting the inline box into two boxes (even if either side is empty), one on each side of the block-level box(es). The line boxes before the break and after the break are enclosed in anonymous block boxes, and the block-level box becomes a sibling of those anonymous boxes. When such an inline box is affected by relative positioning, any resulting translation also affects the block-level box contained in the inline box.
>
>   ```html
>   <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
>   <HEAD>
>   <TITLE>Anonymous text interrupted by a block</TITLE>
>       <style type="text/css">
>           p    { display: inline;
>                   border:1px solid black}
>   span { display: block }
>       </style>
>   </HEAD>
>   <BODY>
>   <P>
>   This is anonymous text before the SPAN.
>   <SPAN>This is the content of SPAN.</SPAN>
>   This is anonymous text after the SPAN.
>   </P>
>   </BODY>
>   <!--The P element contains a chunk (C1) of anonymous text followed by a block-level element followed by another chunk (C2) of anonymous text. The resulting boxes would be a block box representing the BODY, containing an anonymous block box around C1, the SPAN block box, and another anonymous block box around C2.
>
>   The properties of anonymous boxes are inherited from the enclosing non-anonymous box (e.g., in the example just below the subsection heading "Anonymous block boxes", the one for DIV). Non-inherited properties have their initial value. For example, the font of the anonymous box is inherited from the DIV, but the margins will be 0.
>
>   Properties set on elements that cause anonymous block boxes to be generated still apply to the boxes and content of that element. For example, if a border had been set on the P element in the above example, the border would be drawn around C1 (open at the end of the line) and C2 (open at the start of the line).-->
>
>   <!--匿名的盒的属性继承于其上一个非匿名包含盒的属性-->
>
>   <!--被破坏的inline box 属性会留给 其分解的匿名盒-->
>   <!--对于匿名盒含有半分比的属性，会直接根据其最近的祖先非匿名盒的需要相乘的值和百分数相乘-->
>   ```
>
>   ![hehe](https://github.com/BaichuanWu/prictures/raw/master/front/format_model.png)
>
> - 不是所有的块容器都是块级盒（**Not all** block container boxes are block-level boxes: non-replaced inline blocks（内联的块容器？inline-block） and non-replaced table cells are block containers but not block-level boxes）.
>
> - Block-level boxes that are also block containers are called **block boxes**.

##### 2.1.2 Inline-level elements and inline boxes

> Inline-level elements are those elements of the source document that do not form new blocks of content; the content is distributed in lines (e.g., emphasized pieces of text within a paragraph, inline images, etc.). The following values of the 'display' property make an element inline-level: 'inline', 'inline-table', and 'inline-block'. Inline-level elements generate inline-level boxes, which are boxes that participate in an inline formatting context.
>
> An inline box is one that is both inline-level and whose contents participate in its containing inline formatting context. A non-replaced element with a 'display' value of 'inline' generates an inline box. Inline-level boxes that are not inline boxes (such as replaced inline-level elements, inline-block elements, and inline-table elements) are called atomic inline-level boxes because they participate in their inline formatting context as a single opaque box.

**2.2 position schemes**

> 1. 标准文档流(normal flow), In CSS 2.1, normal flow includes [block formatting](https://www.w3.org/TR/CSS21/visuren.html#block-formatting) of block-level boxes, [inline formatting](https://www.w3.org/TR/CSS21/visuren.html#inline-formatting) of inline-level boxes, and [relative positioning](https://www.w3.org/TR/CSS21/visuren.html#relative-positioning) of block-level and inline-level boxes.
> 2. 浮动(float),In the float model, a box is first laid out according to the normal flow, then taken out of the flow and shifted to the left or right as far as possible. Content may flow along the side of a float.
> 3. 绝对定位(absolute float),In the absolute positioning model, a box is removed from the normal flow entirely (it has no impact on later siblings) and assigned a position with respect to a containing block.

**2.2.1 normal flow**

> 1. BFC:浮动，绝对定位，不是块盒的块容器，over-flow不是visible的块盒会为他们的内容形成新的bfc;
>
>    块级盒的垂直magin会collapse (tip:需要注意的是，display:table 本身并不会创建BFC，但是它会产生匿名框(anonymous boxes)，而匿名框中的display:table-cell可以创建新的BFC，换句话说，触发块级格式化上下文的是匿名框，而不是display:table。所以通过display:table和display:table-cell创建的BFC效果是不一样的。)
>
>    通俗地来说：创建了 BFC的元素就是一个独立的盒子，里面的子元素不会在布局上影响外面的元素，反之亦然，同时BFC任然属于文档中的普通流。
>
>    ​





#### 3.media type

> 目前为止css有两种指明媒体依赖的方法
>
> - 通过@import 或者 @media
>
>   ```css
>   @import url("fancyfonts.css") screen;
>   @media print {
>     /* style sheet for print goes here */
>   }
>   ```
>
> - 通过link 标签的media属性
>
>   ```html
>   <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
>   <HTML>
>      <HEAD>
>         <TITLE>Link to a target medium</TITLE>
>         <LINK REL="stylesheet" TYPE="text/css" 
>   	 MEDIA="print, handheld" HREF="foo.css">
>      </HEAD>
>      <BODY>
>         <P>The body...
>      </BODY>
>   </HTML>
>   ```
>
> 不加@media默认给所有的media type适用

>  
>
>  | Media Types | Media Groups     |                             |             |                    |
>  | ----------- | ---------------- | --------------------------- | ----------- | ------------------ |
>  |             | continuous/paged | visual/audio/speech/tactile | grid/bitmap | interactive/static |
>  | braille     | continuous       | tactile                     | grid        | both               |
>  | embossed    | paged            | tactile                     | grid        | static             |
>  | handheld    | both             | visual, audio, speech       | both        | both               |
>  | print       | paged            | visual                      | bitmap      | static             |
>  | projection  | paged            | visual                      | bitmap      | interactive        |
>  | screen      | continuous       | visual, audio               | bitmap      | both               |
>  | speech      | continuous       | speech                      | N/A         | both               |
>  | tty         | continuous       | visual                      | grid        | both               |
>  | tv          | both             | visual, audio               | bitmap      | both               |

#### 4.css的运行模型

> 1. 从文档中解析出文档树
> 2. 确认目标的媒体类型
> 3. 获取文档所关联的目标媒体类型的样式表
> 4. 根据层叠和继承为每个目标元素添加能获取的属性注释，不同的媒体类型，属性的样式算法会不一样。
> 5. 根据带注释的文档树生成格式化结构（formating structure）,格式化结构和文档树会很不一样（ex:一些display为None的就不会生成）
> 6. 在目标媒体显示格式化结构 cavans 是目标媒体渲染的地方