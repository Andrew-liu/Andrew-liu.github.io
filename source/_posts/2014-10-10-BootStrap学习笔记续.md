---
layout : page
date: 2014-10-10 23:31:39 +0800
title: BootStrap学习笔记续
tags: JavaScript
image:
  feature: abstract-5.jpg
commets: true
share: true
---

##3.1. 基本用法
###3.1.1. 列组合简单理解就是更改数字来合并列（原则：列总和数不能超12），有点类似于表格的colspan属性

```
<div class="container">
  <div class="row">
    <div class="col-md-4">.col-md-4</div>//4+8 = 12
    <div class="col-md-8">.col-md-8</div>
  </div>
  <div class="row">
    <div class="col-md-4">.col-md-4</div> // 4+4+4 = 12
    <div class="col-md-4">.col-md-4</div>
    <div class="col-md-4">.col-md-4</div>
  </div>
  <div class="row">
    <div class="col-md-3">.col-md-3</div>//3+6+3 = 12
    <div class="col-md-6">.col-md-6</div>
    <div class="col-md-3">.col-md-3</div>
 </div>
</div>
```


<!--more-->
```
```

###3.1.2. 列偏移
不希望相邻的两个列紧靠在一起，但又不想使用margin或者其他的技术手段来。这个时候就可以使用列偏移（offset）功能来实现。使用列偏移也非常简单，只需要在列元素上添加类名`col-md-offset-*`(其中星号代表要偏移的列组合数)，那么具有这个类名的列就会向右偏移。例如，你在列元素上添加“col-md-offset-4”，表示该列向右移动4个列的宽度。

> 要保证列与偏移列的总数不超过12，不然会致列断行显示

```
<div class="container">
<div class="row">
<div class="col-md-4">.col-md-4</div>
<div class="col-md-2 col-md-offset-4">列向右移动四列的间距</div>
<div class="col-md-2">.col-md-3</div>
</div>
<div class="row">
<div class="col-md-4">.col-md-4</div>
<div class="col-md-4 col-md-offset-4">列向右移动四列的间距</div>
</div>
</div>
```

###3.1.3. 列排序
列排序其实就是改变列的方向，就是改变左右浮动，并且设置浮动的距离。在Bootstrap框架的网格系统中是通过添加类`col-md-push-*`和`col-md-pull-*`(其中星号代表移动的列组合数)。

```
<div class="container">
  <div class="row">
    <div class="col-md-4 col-md-push-8">.col-md-4</div>
    <div class="col-md-8 col-md-pull-4">.col-md-8</div>
  </div>
```

###3.1.4. 列的嵌套
Bootstrap框架的网格系统还支持列的嵌套。可以在一个列中添加一个或者多个行（row）容器，然后在这个行容器中插入列（像前面介绍的一样使用列）

```
<div class="container">
    <div class="row">
        <div class="col-md-8">
        我的里面嵌套了一个网格
            <div class="row">
            <div class="col-md-6">col-md-6</div>
            <div class="col-md-6">col-md-6</div>
          </div>
        </div>
    <div class="col-md-4">col-md-4</div>
    </div>
    <div class="row">
        <div class="col-md-4">.col-md-4</div>
    <div class="col-md-8">
    我的里面嵌套了一个网格
        <div class="row">
          <div class="col-md-4">col-md-4</div>
          <div class="col-md-4">col-md-4</div>
          <div class="col-md-4">col-md-4</div>
        </div>
    </div>
    </div>
</div>
```

#4. 下拉菜单
> 因为Bootstrap的组件交互效果都是依赖于jQuery库写的插件，所以在使用bootstrap.min.js之前一定要先加载jquery.min.js才会生效果

```
<div class="dropdown">
  <button class="btn btn-default dropdown-toggle" type="button" id="dropdownMenu1" data-toggle="dropdown">
    下拉菜单
    <span class="caret"></span>
  </button>
  <ul class="dropdown-menu" role="menu" aria-labelledby="dropdownMenu1">
    <li role="presentation"><a role="menuitem" tabindex="-1" href="#">下拉菜单项</a></li>
    <li role="presentation"><a role="menuitem" tabindex="-1" href="#">下拉菜单项</a></li>
    <li role="presentation"><a role="menuitem" tabindex="-1" href="#">下拉菜单项</a></li>
    <li role="presentation"><a role="menuitem" tabindex="-1" href="#">下拉菜单项</a></li>
  </ul>
</div> 
```

##4.1. 使用方法
1. 使用一个名为`dropdown`的容器包裹了整个下拉菜单元素，示例中为:`<div class="dropdown"></div>`
2. 使用了一个`<button>`按钮做为父菜单，并且定义类名`dropdown-toggle`和自定义`data-toggle`属性，且值必须和最外容器类名一致，此示例为:`data-toggle="dropdown"`
3. 下拉菜单项使用一个ul列表，并且定义一个类名为`dropdown-menu`，此示例为:
`<ul class="dropdown-menu">`

##4.2. 下拉分隔线
下拉分隔线，假设下拉菜单有两个组，那么组与组之间可以通过添加一个空的`<li>`，并且给这个`<li>`添加类名`divider`来实现添加下拉分隔线的功能。

```
<li role="presentation"><a role="menuitem" tabindex="-1" href="#">下拉菜单项</a></li>
    <li role="presentation" class="divider"></li> <!--添加一个空个li实现下拉分割线-->
    <li role="presentation"><a role="menuitem" tabindex="-1" href="#">下拉菜单项</a></li>
```

##4.3. 菜单标题

```
<ul class="dropdown-menu" role="menu" aria-labelledby="dropdownMenu1">
<li role="presentation" class="dropdown-header">第一部分菜单头部</li>
<li role="presentation"><a role="menuitem" tabindex="-1" href="#">下拉菜单项</a></li>
```

##4.4. 对齐方式和菜单项状态
- Bootstrap框架中下拉菜单默认是左对齐，如果你想让下拉菜单相对于父容器右对齐时，可以在“dropdown-menu”上添加一个“pull-right”或者“dropdown-menu-right”类名

> `<ul class="dropdown-menu pull-right" role="menu" aria-labelledby="dropdownMenu1">`

- 下拉菜单项的默认的状态（不用设置）有悬浮状态（:hover）和焦点状态（:focus),下拉菜单项除了上面两种状态，还有当前状态（.active）和禁用状态（.disabled）。这两种状态使用方法只需要在对应的菜单项上添加对应的类名

```
 <li role="presentation" class="active"><a role="menuitem" tabindex="-1" href="#">下拉菜单项</a></li>
    ….
    <li role="presentation" class="disabled"><a role="menuitem" tabindex="-1" href="#">下拉菜单项</a></li>
```

##4.4. 按钮（按钮组）
> 按钮组和下拉菜单组件一样，需要依赖于button.js插件才能正常运行。

对于结构方面，非常的简单。使用一个名为“btn-group”的容器，把多个按钮放到这个容器中。

```
<div class="dropdown">
  <div class="btn-group">
      <button type="button" class="btn btn-default"><span class="glyphicon glyphicon-step-backward"></span></button>
      <button type="button" class="btn btn-default"><span class="glyphicon glyphicon-fast-backward"></span></button>
      <button type="button" class="btn btn-default"><span class="glyphicon glyphicon-backward"></span></button>
      <button type="button" class="btn btn-default"><span class="glyphicon glyphicon-play"></span></button>
      <button type="button" class="btn btn-default"><span class="glyphicon glyphicon-pause"></span></button>
  </div>
</div>
```

##4.5. 按钮（按钮工具栏）
Bootstrap框架按钮工具栏提供了这样的制作方法，你只需要将按钮组`btn-group`按组放在一个大的容器`btn-toolbar`中

按钮是通过btn-lg、btn-sm和btn-xs三个类名来调整padding、font-size、line-height和border-radius属性值来改变按钮大小。那么按钮组的大小，我们也可以通过类似的方法：
  ☑  .btn-group-lg:大按钮组
  ☑  .btn-group-sm:小按钮组
  ☑  .btn-group-xs:超小按钮组

```
<div class="btn-toolbar">
  <div class="btn-group btn-group-lg"> <!--大按钮组-->
    <button type="button" class="btn btn-default"><span class="glyphicon glyphicon-align-left"></span></button>
    <button type="button" class="btn btn-default"><span class="glyphicon glyphicon-align-center"></span></button>
    <button type="button" class="btn btn-default"><span class="glyphicon glyphicon-align-right"></span></button>
    <button type="button" class="btn btn-default"><span class="glyphicon glyphicon-align-justify"></span></button>
  </div>
  <div class="btn-group">
    <button type="button" class="btn btn-default"><span class="glyphicon glyphicon-indent-left"></span></button>
    <button type="button" class="btn btn-default"><span class="glyphicon glyphicon-indent-right"></span></button>
  </div>
  <div class="btn-group btn-group-sm">
    <button type="button" class="btn btn-default"><span class="glyphicon glyphicon-font"></span></button>
    <button type="button" class="btn btn-default"><span class="glyphicon glyphicon-bold"></span></button>
    <button type="button" class="btn btn-default"><span class="glyphicon glyphicon-italic"></span></button>
  </div>
  <div class="btn-group btn-group-xs">
    <button type="button" class="btn btn-default"><span class="glyphicon glyphicon-text-height"></span></button>
    <button type="button" class="btn btn-default"><span class="glyphicon glyphicon-text-width"></span></button>
  </div>
</div>
```

##4.6. 按钮（嵌套分组）
只需要把当初制作下拉菜单的“dropdown”的容器换成“btn-group”，并且和普通的按钮放在同一级。

```
<div class="btn-group">
  <button class="btn btn-default" type="button">首页</button>
  <button class="btn btn-default" type="button">产品展示</button>
  <button class="btn btn-default" type="button">案例分析</button>
  <button class="btn btn-default" type="button">联系我们</button>
  <div class="btn-group">
      <button class="btn btn-default dropdown-toggle" data-toggle="dropdown" type="button">关于我们<span class="caret"></span></button>
    <ul class="dropdown-menu">
        <li><a href="##">公司简介</a></li>
        <li><a href="##">企业文化</a></li>
        <li><a href="##">组织结构</a></li>
        <li><a href="##">客服服务</a></li>
    </ul>
  </div>
</div>
```

##4.7. 按钮（垂直分组）
只需要把水平分组的“btn-group”类名换成“btn-group-vertical”

```
<div class="btn-group-vertical">
<button class="btn btn-default" type="button">首页</button>
<button class="btn btn-default" type="button">产品展示</button>
<button class="btn btn-default" type="button">案例分析</button>
<button class="btn btn-default" type="button">联系我们</button>
<div class="btn-group">
   <button class="btn btn-default dropdown-toggle" data-toggle="dropdown" type="button">关于我们<span class="caret"></span></button>
   <ul class="dropdown-menu">
      <li><a href="##">公司简介</a></li>
      <li><a href="##">企业文化</a></li>
      <li><a href="##">组织结构</a></li>
      <li><a href="##">客服服务</a></li>
</ul>
</div>
</div>
```

##4.8. 按钮（等分按钮）
等分按钮也常被称为是`自适应分组按钮`，其实现方法也非常的简单，只需要在按钮组`btn-group`上追加一个`btn-group-justified`类名

> 在制作等分按钮组时，请尽量使用`<a>`标签元素来制作按钮，因为使用`<button>`标签元素时，使用display:table在部分浏览器下支持并不友好。

```
<div class="btn-wrap">
<div class="btn-group btn-group-justified">
  <a class="btnbtn-default" href="#">首页</a>
  <a class="btnbtn-default" href="#">产品展示</a>
  <a class="btnbtn-default" href="#">案例分析</a>
  <a class="btnbtn-default" href="#">联系我们</a>
</div>
</div>
```

##4.9. 按钮下拉菜单
按钮下拉菜单其实就是普通的下拉菜单，只不过把`<a>`标签元素换成了`<button>`标签元素。唯一不同的是外部容器`div.dropdown`换成了`div.btn-group`。

```
<div class="btn-group">
      <button class="btn btn-default dropdown-toggle" data-toggle="dropdown" type="button">按钮下拉菜单<span class="caret"></span></button>
      <ul class="dropdown-menu">
          <li><a href="##">按钮下拉菜单项</a></li>
          <li><a href="##">按钮下拉菜单项</a></li>
          <li><a href="##">按钮下拉菜单项</a></li>
          <li><a href="##">按钮下拉菜单项</a></li>
      </ul>
</div>
```

##4.10. 按钮的向下向上三角形
按钮的向下三角形，我们是通过在`<button>`标签中添加一个`<span>`标签元素，并且命名为`caret`:

```
<button class="btn btn-default dropdown-toggle" data-toggle="dropdown" type="button">按钮下拉菜单<span class="caret"></span></button>

<!--下拉菜单会向上弹起，这个时候三角方向需要朝上显示，实现方法：需要在“.btn-group”类上追加“dropup”类名-->
<div class="btn-group dropup">
  <button class="btn btn-default dropdown-toggle" data-toggle="dropdown" type="button">按钮下拉菜单<span class="caret"></span></button>
  <ul class="dropdown-menu">
        <li><a href="##">按钮下拉菜单项</a></li>
        <li><a href="##">按钮下拉菜单项</a></li>
        <li><a href="##">按钮下拉菜单项</a></li>
        <li><a href="##">按钮下拉菜单项</a></li>
  </ul>
</div>
```

#5. 导航
Bootstrap框架中制作导航条主要通过`.nav`样式。默认的`.nav`样式不提供默认的导航样式，必须附加另外一个样式才会有效，比如`nav-tabs`、`nav-pills`之类

```
<ul class="nav nav-tabs">
    <li><a href="##">Home</a></li>
    <li><a href="##">CSS3</a></li>
    <li><a href="##">Sass</a></li>
    <li><a href="##">jQuery</a></li>
    <li><a href="##">Responsive</a></li>
</ul>
```

##5.1. 导航（标签形tab导航）
一般情况之下，选项卡教会有一个当前选中项。在Bootstrap框架也相应提供了。假设我们想让`Home`项为当前选中项，只需要在其标签上添加类名`class="active"`即可：

```
<ul class="nav nav-tabs">
    <li class="active"><a href="##">Home</a></li>
    …
</ul>
```

##5.2. 导航（胶囊形(pills)导航）
当前项高亮显示，并带有圆角效果。其实现方法和“nav-tabs”类似，同样的结构，只需要把类名`nav-tabs`换成`nav-pills`

![图片](http://img.mukewang.com/53e86ee60001711e08160307.jpg)

```
<ul class="nav nav-pills">
      <li class="active"><a href="##">Home</a></li>
      <li><a href="##">CSS3</a></li>
      <li><a href="##">Sass</a></li>
      <li><a href="##">jQuery</a></li>
      <li class="disabled"><a href="##">Responsive</a></li>
</ul>
```

##5.3. 导航（垂直堆叠的导航）
制作垂直堆叠导航只需要在`nav-pills`的基础上添加一个`nav-stacked`

```
<ul class="nav nav-pills nav-stacked">
     <li class="active"><a href="##">Home</a></li>
     <li><a href="##">CSS3</a></li>
     <li><a href="##">Sass</a></li>
     <li class="nav-divider"></li><!--，下拉菜单组与组之间有一个分隔线。在垂直堆叠导航也具有这样的效果，只需要添加在导航项之间添加“<li class=”nav-divider”></li>”即可：-->
     <li class="disabled"><a href="##">Responsive</a></li>
</ul>
```

##5.4. 自适应导航（使用）
自适应导航指的是导航占据容器全部宽度，而且菜单项可以像表格的单元格一样自适应宽度。自适应导航和前面使用`btn-group-justified`制作的自适应按钮组是一样的。只不过在制作自适应导航时更换了另一个类名`nav-justified`。当然他需要和`nav-tabs`或者`nav-pills`配合在一起使用。

```
<ul class="nav nav-tabs nav-justified">
     <li class="active"><a href="##">Home</a></li>
     <li><a href="##">CSS3</a></li>
     <li><a href="##">Sass</a></li>
     <li><a href="##">jQuery</a></li>
     <li><a href="##">Responsive</a></li>
</ul>
```

##5.5. 导航加下拉菜单（二级导航）
Bootstrap框架中制作二级导航就更容易了。只需要将li当作父容器，使用类名`dropdown`，同时在li中嵌套另一个列表ul，使用前面介绍下拉菜单的方法就可以.

```
<ul class="nav nav-pills">
     <li class="active"><a href="##">首页</a></li>
     <li class="dropdown">
        <a href="##" class="dropdown-toggle" data-toggle="dropdown">教程<span class="caret"></span></a>
        <ul class="dropdown-menu">
            <li><a href="##">CSS3</a></li>
            …
       </ul>
     </li>
     <li><a href="##">关于我们</a></li>
</ul>
```

##5.6. 面包屑式导航
`Breadcrumb`主要是起的作用是告诉用户现在所处页面的位置（当前位置）

```
<ol class="breadcrumb">
  <li><a href="#">首页</a></li>
  <li><a href="#">我的书</a></li>
  <li class="active">《图解CSS3》</li>
</ol>
```


#6. 导航条
> 导航条(navbar)中有一个背景色、而且导航条可以是纯链接（类似导航），也可以是表单，还有就是表单和导航一起结合等多种形式

##6.1. 基础导航条
1. 首先在制作导航的列表(`<ul class=”nav”>`)基础上添加类名“navbar-nav”
2. 在列表外部添加一个容器（div），并且使用类名`navbar`和`navbar-default`
3. `navbar-inverse`与默认的导航条相比，使用方法并无区别，只是将navbar-deafult类名换成navbar-inverse。其变化只是导航条的背景色和文本做了修改
```
<div class="navbar navbar-default" role="navigation">
     <ul class="nav navbar-nav">
        <li class="active"><a href="##">网站首页</a></li>
        <li><a href="##">CSS</a></li>
        <li><a href="##">HMTL</a></li>
        <li><a href="##">Sass</a></li>
        <li><a href="##">Less</a></li>
     </ul>
</div>
```
##6.2. 为导航条添加标题、二级菜单及状态
通过`navbar-header`和`navbar-brand`来实现

```
<div class="navbar navbar-default" role="navigation">
  　<div class="navbar-header">
  　    <a href="##" class="navbar-brand">andrew</a>
  　</div>
    <ul class="nav navbar-nav">
        <li class="active"><a href="##">css</a></li>
        <li class="dropdown">
          <a href="##" data-toggle="dropdown" class="dropdown-toggle">java<span class="caret"></span></a>
          <ul class="dropdown-menu">
            <li><a href="##">CSS3</a></li>
            <li><a href="##">JavaScript</a></li>
            <li class="disabled"><a href="##">PHP</a></li>
          </ul>
       </li>
       <li><a href="##">名师介绍</a></li>
       <li><a href="##">成功案例</a></li>
       <li><a href="##">关于我们</a></li>
    </ul>
</div>
```

##6.3. 带表单的导航条
在Bootstrap框架中提供了一个`navbar-form`，使用方法很简单，在navbar容器中放置一个带有`navbar-form`类名的表单，

```
<div class="navbar navbar-default" role="navigation">
  　<div class="navbar-header">
  　    <a href="##" class="navbar-brand">慕课网</a>
  　</div>
    <ul class="nav navbar-nav">
       <li><a href="##">网站首页</a></li>
       
      <li class = "active"><a href="##">名师介绍</a></li>
      <li><a href="##">成功案例</a></li>
      <li class="dropdown">
          <a href="##" data-toggle="dropdown" class="dropdown-toggle">关于我们<span class="caret"></span></a>
          <ul class="dropdown-menu">
            <li><a href="##">CSS3</a></li>
            <li><a href="##">JavaScript</a></li>
            <li><a href="##">PHP</a></li>
          </ul>
      </li>
     </ul>
     <form action="##" class="navbar-form navbar-left" rol="search">
        <div class="form-group">
           <input type="text" class="form-control" placeholder="请输入关键词" />
        </div>
        <button type="submit" class="btn btn-default">搜索</button>
     </form>
</div>
```

##6.4. 固定导航条
Bootstrap框架提供了两种固定导航条的方式

1. `.navbar-fixed-top`：导航条固定在浏览器窗口顶部
2. `.navbar-fixed-bottom`：导航条固定在浏览器窗口底部

```
<!--只需要在制作导航条最外部容器navbar上追加对应的类名-->
<div class="navbar navbar-default navbar-fixed-top" role="navigation">
　…
</div>
<div class="content">我是内容</div>
<div class="navbar navbar-default navbar-fixed-bottom" role="navigation">
　…
</div>
```

##6.5. 响应式导航条
响应式设计也就随之而来。那么在一个响应式的Web页面中，对于响应式的导航条也就非常的重要。
###6.5.1. 使用方法
1. 保证在窄屏时需要折叠的内容必须包裹在带一个div内，并且为这个div加入`collapse、navbar-collapse`两个类名。最后为这个div添加一个class类名或者id名。
2. 保证在窄屏时要显示的图标样式（固定写法）：

```
<button class="navbar-toggle" type="button" data-toggle="collapse">
  <span class="sr-only">Toggle Navigation</span>
  <span class="icon-bar"></span>
  <span class="icon-bar"></span>
  <span class="icon-bar"></span>
</button>
```

3. 并为button添加data-target=`.类名/#id名`，究竞是类名还是id名呢？由需要折叠的div来决定

```
<!--需要折叠的div代码段-->
<!-- .navbar-toggle样式用于toggle收缩的内容，即nav-collapse collapse样式所在元素 -->
<div class="collapse navbar-collapse" id="example">
      <ul class="nav navbar-nav">
      …
      </ul>
</div>
<!--窄屏时显示的图标代码段-->
<button class="navbar-toggle" type="button" data-toggle="collapse" data-target="#example">
  ...
</button>
```

```
<div class="navbar navbar-default" role="navigation">
  <div class="navbar-header">
     　<!-- .navbar-toggle样式用于toggle收缩的内容，即nav-collapse collapse样式所在元素 -->
       <button class="navbar-toggle" type="button" data-toggle="collapse" data-target=".navbar-responsive-collapse">
         <span class="sr-only">Toggle Navigation</span>
         <span class="icon-bar"></span>
         <span class="icon-bar"></span>
         <span class="icon-bar"></span>
       </button>
       <!-- 确保无论是宽屏还是窄屏，navbar-brand都显示 -->
       <a href="##" class="navbar-brand">慕课网</a>
  </div>
  <!-- 屏幕宽度小于768px时，div.navbar-responsive-collapse容器里的内容都会隐藏，显示icon-bar图标，当点击icon-bar图标时，再展开。屏幕大于768px时，默认显示。 -->
  <div class="collapse navbar-collapse navbar-responsive-collapse">
        <ul class="nav navbar-nav">
            <li class="active"><a href="##">网站首页</a></li>
            <li><a href="##">系列教程</a></li>
            <li><a href="##">名师介绍</a></li>
            <li><a href="##">成功案例</a></li>
            <li><a href="##">关于我们</a></li>
        </ul>
  </div>
</div>
```

##6.6. 分页导航（带页码的分页导航）
在Bootstrap框架中提供了两种分页导航：
- 带页码的分页导航
- 带翻页的分页导航
- 在Bootstrap框架中使用的是`ul>li>a`这样的结构，在ul标签上加入`pagination`方法：

```
<ul class="pagination">
   <li><a href="#">&laquo;</a></li>
   <li><a href="#">1</a></li>
   <li><a href="#">2</a></li>
   <li><a href="#">3</a></li>
   <li><a href="#">4</a></li>
   <li><a href="#">5</a></li>
   <li><a href="#">&raquo;</a></li>
</ul>
```

###6.6.1. 分页大小设置
1. 通过“pagination-lg”让分页导航变大
2. 通过“pagination-sm”让分页导航变小

```
<ul class="pagination pagination-lg">
  <li><a href="#">&laquo;第一页</a></li>
  <li><a href="#">11</a></li>
  <li><a href="#">12</a></li>
  <li class="active"><a href="#">13</a></li>
  <li><a href="#">14</a></li>
  <li><a href="#">15</a></li>
  <li class="disabled"><a href="#">最后一页&raquo;</a></li>
</ul> 
```

##6.6.2. 翻页分页导航
在实际使用中，翻页分页导航和带页码的分页导航类似，为ul标签加入`pager类`

```
<ul class="pager">
   <li><a href="#">&laquo;上一页</a></li>
   <li><a href="#">下一页&raquo;</a></li>
</ul>
<!--左右对齐-->
<ul class="pager">
  <li class="previous"><a href="#">&laquo;上一页</a></li>
  <li class="next"><a href="#">下一页&raquo;</a></li>
</ul> 
```

##6.7. 标签
Bootstrap框架中特意label效果提取出来成为一个标签组件，并且以`.label`样式来实现高亮显示。

颜色样式设置：
和按钮元素button类似，label样式也提供了多种颜色：
1. label-deafult:默认标签，深灰色
2. label-primary：主要标签，深蓝色
3. label-success:成功标签，绿色
4. label-in:信息标签，浅蓝
5. label-warning：警告标签，橙色
6. label-danger：错误标签，红色

```
<span class="label label-default">默认标签</span>
<span class="label label-primary">主要标签</span>
<span class="label label-success">成功标签</span>
<span class="label label-info">信息标签</span>
<span class="label label-warning">警告标签</span>
<span class="label label-danger">错误标签</span>
```

##6.8. 徽章
比如你登录你的twitter后，如果你信息没有看，系统会告诉你有多少信息未读，如下图所示：
![徽章](http://img.mukewang.com/53f5aac500010a7f04370079.jpg)

> 在Bootstrap框架中，把这种效果称作为徽章效果，使用“badge”样式来实现,可以像标签一样，使用span标签来制作，然后为他加入`badge`类

```
<a href="#">Inbox <span class="badge">42</span></a>
```
