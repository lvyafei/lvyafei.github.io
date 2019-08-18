---
layout: lay_post
title: "CSS布局(layout)三大器-display-position-float"
date: 2017-04-09
categories: CSS布局
tags: WEB技术
author: lvyafei
---

## 0.概述

CSS布局常用方式有display(显示),position(定位),float(浮动)三种手段，每种手段对应的使用场景也不一样。
<!-- more -->

## 1.display属性

display的常用值有:inline,block,flex,grid,table。还有不显示none,以及交叉组合:inline-block,inline-flex,inline-grid,inline-table。

**inline**:行内元素

**block**:块元素

**flex**:Flexible Box弹性布局

**grid**:网格布局

**table**:表格布局

```html

<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
	<title>test</title>
</head>
<body>

<h2>1.块元素(block-element)</h2>
<span>a:块级元素会独占一行,其宽度自动填满其父元素宽度。<br/></span>
<span>b:块级元素可以设置 width,height属性,块级元素即使设置了宽度,仍然是独占一行的。<br/></span>
<span>c:块级元素可以设置margin和padding。<br/></span>
<span>d:block元素可以包含block元素和inline元素；但inline元素只能包含inline元素。要注意的是这个是个大概的说法，每个特定的元素能包含的元素也是特定的，所以具体到个别元素上，这条规律是不适用的。比如 P 元素，只能包含inline元素，而不能包含block元素。<br/></span>

<h4>1.1 div</h4>
<div style="height:80px;width:50px;border:1px solid red"></div>
<div style="height:80px;width:120px;border:1px solid blue"></div>

<h4>1.2 p</h4>
<p style="height: 20px;width:200px;background-color:red">
	<span>这是p标签中的span标签</span>
</p>

<h4>1.3 form</h4>
<form style="border:1px solid blue">
	<span>form中的span标签</span>
</form>

<h4>1.4 块元素汇总</h4>
<span>
* address - 地址<br/>
* blockquote - 块引用<br/>
* center - 举中对齐块<br/>
* dir - 目录列表<br/>
* div - 常用块级容易，也是css layout的主要标签<br/>
* dl - 定义列表<br/>
* fieldset - form控制组<br/>
* form - 交互表单<br/>
* h1 - 大标题<br/>
* h2 - 副标题<br/>
* h3 - 3级标题<br/>
* h4 - 4级标题<br/>
* h5 - 5级标题<br/>
* h6 - 6级标题<br/>
* hr - 水平分隔线<br/>
* isindex - input prompt<br/>
* menu - 菜单列表<br/>
* noframes - frames可选内容，（对于不支持frame的浏览器显示此区块内容<br/>
* noscript - ）可选脚本内容（对于不支持script的浏览器显示此内容）<br/>
* ol - 排序表单<br/>
* p - 段落<br/>
* pre - 格式化文本<br/>
* table - 表格<br/>
* ul - 非排序列表<br/>
</span>

<h2>2.行内元素(inline-element)</h2>
<span>a:行内元素不会独占一行,相邻的行内元素会排列在同一行里,直到一行排不下,才会换行,其宽度随元素的内容而变化。<br/></span>
<span>b:行内元素设置width,height无效。<br/></span>
<span>c:行内元素的水平方向的padding-left,padding-right,margin-left,margin-right 都产生边距效果，但是竖直方向的padding-top,padding-bottom,margin-top,margin-bottom都不会产生边距效果。<br/></span>

<h4>2.1 input</h4>
<input type="button" value="button1" />
<input type="button" value="button2" />

<h4>2.2 label</h4>
<label>label1</label>
<label>label2</label>
<label>label3</label>

<h4>2.3 select</h4>
<select>
	<option>sele1</option>
	<option>sele2</option>
	<option selected="true">sele3</option>
	<option>sele4</option>
	<option>sele5</option>
</select>
<select>
	<option>水果1</option>
	<option selected="true">水果2</option>
	<option>水果3</option>
	<option>水果4</option>
	<option>水果5</option>
</select>

<h4>2.4 行内元素汇总</h4>
<span>
* a - 锚点<br/>
* abbr - 缩写<br/>
* acronym - 首字<br/>
* b - 粗体(不推荐)<br/>
* bdo - bidi override<br/>
* big - 大字体<br/>
* br - 换行<br/>
* cite - 引用<br/>
* code - 计算机代码(在引用源码的时候需要)<br/>
* dfn - 定义字段<br/>
* em - 强调<br/>
* font - 字体设定(不推荐)<br/>
* i - 斜体<br/>
* img - 图片<br/>
* input - 输入框<br/>
* kbd - 定义键盘文本<br/>
* label - 表格标签<br/>
* q - 短引用<br/>
* s - 中划线(不推荐)<br/>
* samp - 定义范例计算机代码<br/>
* select - 项目选择<br/>
* small - 小字体文本<br/>
* span - 常用内联容器，定义文本内区块<br/>
* strike - 中划线<br/>
* strong - 粗体强调<br/>
* sub - 下标<br/>
* sup - 上标<br/>
* textarea - 多行文本输入框<br/>
* tt - 电传文本<br/>
* u - 下划线<br/>
* var - 定义变量<br/>
</span>

<h2>3.inlinke-block</h2>
<span>简单来说就是将对象呈现为inline对象，但是对象的内容作为block对象呈现。之后的内联对象会被排列在同一行内。比如我们可以给一个link（a元素）inline-block属性值，使其既具有block的宽度高度特性又具有inline的同行特性</span>

<h2>4.可变元素</h2>
<span>4.1可变元素为根据上下文语境决定该元素为块元素或者内联元素。<br/></span>
<span>
<br/>
* applet - java applet<br/>
* button - 按钮<br/>
* del - 删除文本<br/>
* iframe - inline frame<br/>
* ins - 插入的文本<br/>
* map - 图片区块(map)<br/>
* object - object对象<br/>
* script - 客户端脚本<br/>
</span>

<h2>5.布局设置display</h2>
<span>一般来说，可以通过display:inline和display:block的设置，改变元素的布局级别。display可以设置的参数不止以上三种，只是比较常用而已。</span>

</body>
</html>

```

## 2.position属性

position属性的值有:absolute(绝对布局),relative(相对布局),fixed(固定布局),static(默认布局),inherit(继承父标签position属性值)

和position搭配使用的属性有:left,right,top,bottom和层次属性z-index.

## 3.float属性

float属性的值有:left,right,none(默认值,不浮动),inherit(继承父标签float属性值)

