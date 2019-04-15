# JavaScript
## 1. a标签

```html
 <!--页面内跳转,跳转到页面顶端-->
<a href="#" onclick="jump_top"/>  

<!--点击事件,最稳妥的方法,不会将 JavaScript 方法暴露在浏览器状态栏-->
<a href="javascript:void(0);" onclick="js_method()"/>

<!--和上边的作用一样-->
<a href="javascript:;" onclick="js_method()"/>
```

## 2. 建立路径

```properties
. ：代表目前所在的目录，相对路径。 如：<a href="./abc">文本</a> 或 <img src="./abc" />
.. ：代表上一层目录，相对路径。 如：<a href="../abc">文本</a> 或 <img src="../abc" />
../../ ：代表的是上一层目录的上一层目录，相对路径。 如：<img src="../../abc" />
/ ：代表根目录，绝对路径。 如：<a href="/abc">文本</a> 或 <img src="/abc" />
D:/abc/ ：代表根目录，绝对路径
```

## 3. 下拉框

```html
<!--可输入下拉框-->
<input type="text" list="itemlist" name="">
 
<datalist id="itemlist">    //datalist的id要与list属性名称一致
    <option>item1</option>
    <option>item2</option>
</datalist> 
```

## 属性

```properties
href : href是Hypertext Reference的缩写，表示超文本引用。用来建立当前元素和文档之间的链接。常用的有：link、a
src : src是source的缩写，src的内容是页面必不可少的一部分，是引入。src指向的内容会嵌入到文档中当前标签所在的位置。常用的有：img、script、iframe
```

