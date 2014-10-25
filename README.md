BaiduSuggestion
===============
# JavaScript实现百度搜索suggestion功能

---

百度搜索suggestion功能增强了用户体验，现在很多带有搜索功能的网站都会有类似搜索suggestion功能，用以增强用户体验。
暑假的时候试着做了一个百度搜索suggestion功能的demo练习，记录一下，以便以后查看。

本文包括以下几个主目录：

> * <i class="icon-chevron-sign-left"></i> 需求分析
> * <i class="icon-adjust"></i> 技术分析
> * <i class="icon-pencil"></i> 开发工具
> * <i class="icon-reorder"></i> 具体的实现过程


----------

## 一、需求分析
![百度suggestion的效果图][1]


### ① 基本需求：

 1. 在输入框输入关键字后，出现下拉框，下拉框中为suggestion的内容。根据不同的关键字，显示不同的suggestion。
 2. 下拉框支持鼠标点选，鼠标移到suggestion的选择项时，要将对应的字段自动填入百度输入框
 3. 要对suggestion下拉框进行动态处理，百度输入框失去焦点时，suggestion下拉框要自动消失。
 4. 下拉框支持上下键切换，键盘的上下键可对下拉框中的suggestion选项进行选择，当前选项应该自动填入百度输入框。

### ② 扩展需求：

 1. suggestion数据来时百度（jsonp跨域请求js）
 2. 点击“百度一下”按钮可进行百度搜索。
 3. 按回车键可对单前百度输入框中的字段进行搜索。


----------

## 二、技术分析

 1. 根据suggestion字段内容，动态生成对应的suggestion下拉框，这里需要用到Dom节点的操作。
 2. 鼠标点选，这里需要触发鼠标事件。
 3. 键盘上下键选择，这里需要触发键盘事件
 4. 动态改变CSS样式
 5. 向百度后台发jsonp请求，跨域请求js。


----------

## 三、开发工具

 1. Sublime Text
 2. FireFox浏览器

----------

## 四、具体的实现过程
### ① 雏形阶段（suggestion数据来自本地）

 1. 构建基本页面
 *关键代码片段：*
```html
<form id="form" name="myForm">  
    <div id="span_search">  
        <input type="text" id="input_search" autocomplete="off" maxlength="100" name="value_search" onkeyup="keyEvent(event);">  
        <div id="suggestion_div">   
            <ul id="suggestions_ul"></ul>   
        </div>   
    </div>  
    <input id="input_button" type="submit" value="百度一下" >  
</form>  
```
 2. 动态添加下拉框
*1）声明并初始化全局变量（动态添加下拉框的几个很重要的变量）*
```javascript
var inputField = document.forms["myForm"].input_search;  
var suggestionDiv = document.getElementById("suggestion_div");  
var suggestionsUl = document.getElementById("suggestions_ul");   
var suggestionLi = null; 
```
*2）初始化数据源*
```javascript
var suggestion = ["abcde","华南理工大学","abcd","华南师范大学","abcdefg","华南农业大学","abcdefgh","广东工业大学","广东外语外贸大学","计算机科学与技术","广东财经大学","计算机学院","广州医科大学","计算机","广州大学"];//模拟的数据源  
suggestion.sort(); // 按字母排序，使显示结果更友好  
```
*3）动态生成下拉列表*
```javascript
function setSuggestions(the_suggestions) {// 动态生成suggestion下拉框  
    clearSuggestions(); // 每输入一个字符就先清除原先的提示  
    suggestionDiv.className = "show";  
    for (var i = 0; i < the_suggestions.length; i++) {// 将匹配的提示结果逐一显示给用户  
        suggestionLi = document.createElement("li");  
        suggestionsUl.appendChild(suggestionLi);  
        suggestionLi.appendChild(document.createTextNode(the_suggestions[i]));  
        mouseEvent();// 对每个<li>触发鼠标事件  
    }  
}  
```
*4）鼠标事件*
```javascript
function mouseEvent(){  
    suggestionLi.onmouseover = function() {  
        this.className = "mouseOver"; // 鼠标经过时高亮  
        inputField.value = this.firstChild.nodeValue;// 同时将当前的选中值赋值给百度输入框  
    }  
    suggestionLi.onmouseout = function() {  
        this.className = "mouseOut"; // 离开时恢复原样  
    }  
    suggestionLi.onclick = function() {// 用户点击某个匹配项时清除提示框  
        clearSuggestions();   
    }  
}  
```
*5）键盘事件*
##### 这个键盘上下键切换是我在写雏形版时写得相对比较久的一个功能，刚开始想到利用一个标记变量来进行切换，后来发现自己越写越复杂，就只好百度了，后来发现用索引值更好，用索引值就可以标记每一个不同的`<li>`
##### 向上切换的关键代码部分：
```javascript
suggestionNodes.eq(selectedSuggestion).css("background-color", "white");//先将上面一个<li>变成白色  
selectedSuggestion++; // 每向下移一位，selectedSuggestion就+1  
if (selectedSuggestion == suggestionNodes.length) {  
    selectedSuggestion = 0;//如果索引值变成suggestionNodes.length，则将索引值指向第一个元素  
}  
//console.log(selectedSuggestion);  
suggestionNodes.eq(selectedSuggestion).css("background-color", "#ebebeb");//再将下面一个<li>变成灰色  
```
##### 向下切换的关键代码部分：
```javascript
suggestionNodes.eq(selectedSuggestion).css("background-color", "white");//先将下面一个<li>变成白色  
selectedSuggestion--;// 每向上移一位，selectedSuggestion就-1  
if (selectedSuggestion == -1) {  
    selectedSuggestion = suggestionNodes.length - 1;//如果索引值变成-1，则将索引值指向最后一个元素  
}  
//console.log(selectedSuggestion);  
suggestionNodes.eq(selectedSuggestion).css("background-color", "#ebebeb");//再将上面一个<li>变成灰色    
```
*6）动态显示下拉框*
##### 关键代码片段：
```javascript
var aResult = []; // 用于存放匹配结果  
if (inputField.value.length > 0) {  
    for (var i = 0; i < suggestion.length; i++){ // 从数据源中找匹配的数据  
        if (suggestion[i].indexOf(inputField.value) == 0){  
            aResult.push(suggestion[i]); // 压入结果  
        }   
    }  
    if (aResult.length > 0) // 如果有匹配的数据则显示出来  
        setSuggestions(aResult);  
}else  
clearSuggestions(); // 此步用作用户输入（Backspace健）将输入框清除完时的处理  
```
##### 以上为雏形阶段的实现
 3. 雏形阶段的最终结果
![雏形阶段的最终结果][2]
 4. 雏形阶段遇到的问题及解决方法
1）未作用户按（Backspace健）将输入框清除完时的处理，导致用户删除了百度输入框的内容时，suggestion下拉提示框依然存在，影响用户体验。于是在百度输入框没有内容时，作清除suggestion下拉提示框的处理。
2）未作百度输出框失去焦点时的处理，导致用户点击页面其他部分时，suggestion下拉提示框依然存在，影响用户体验。于是在百度输入框失去焦点时，清除suggestion下拉提示框。
3）键盘对suggestion下拉列表的上下切换功能的问题，刚开始想到利用一个标记变量来进行切换，后来发现自己越写越复杂，后来利用索引值实现。

### ②成熟阶段（发jsonp，跨域请求js）

1. 自己的一些想法
在做完雏形之前，我的想法如下：
输入字段进行搜索时，触发onkeyup事件异步向后台传值，从后台中搜索到相应值后，后台向前端传入值（当时想到利用JSON），前端解析JSON后在搜索框下动态生成下拉框。后来想还是想在本地模拟个数据做个雏形（PS；因为前端接到后台传来的JSON对象也是要解析才能用的，在suggestion这个功能中，需要将所有的suggestion数据封装在一个数组里面，所以，我就直接模拟了一个数组，而没有模拟JSON数据，模拟JSON数据的话，还要解析，对于雏形来说就不用了）。
 2. 实现发jsonp，跨域请求js
有了开始时的想法，做完雏形后，就开始想是否可以向百度后台发请求，实现suggestion。于是就开始看百度搜索时到底向后发送了什么请求。打开Firebug，看到输入“百”后，貌似向后台发了两条请求
![此处输入图片的描述][3]
打开请求看了好久，也没看出点什么，未果，然后就打开“百度”，搜索了一番，发现了有用jsonp可以利用关键字获取百度suggestion数据的博文，认真的看了一番之后，发现了关键在向百度后台提交关键字的URL，在哪传值和如何获取返回的suggestion数据。然后试着直接访问http://suggestion.baidu.com/su
![此处输入图片的描述][4]
下载后打开，发现了文件中只有一段行代码
```javascript
window.baidu.sug({q:"",p:false,s:[]}); 
```
数组里面就是我们需要的suggestion数据。于是，发jsonp，跨域请求js，就可以获取到suggestion数据。
```javascript
function fillUrls() {  
    var strdomin = $.trim($("#input_search").val());  
    var qsData = { 'wd': strdomin, 'p': '3', 'cb': 'findSuggestions', 't': new Date().getMilliseconds().toString() };  
  $.ajax({// 发jsonp（跨域请求js）  
        async: false,  
        url: "http://suggestion.baidu.com/su",  
        type: "GET",  
        dataType: 'jsonp',  
        jsonp: 'jsoncallback',  
        data: qsData,  
        timeout: 500,  
        success: function (json) {  
        },  
        error: function (xhr) {  
            alert(xhr);  
        }  
    });  
}  
```
用一个变量去接收suggestion数组里面的数据
```javascript
var urls = strurls.s; 
```
最终效果如下
![此处输入图片的描述][5]
用console.dir(strurls)将接收到的数据打印在控制台上也看到了返回的suggestion数据。
### ③完成阶段（向百度后台提交搜索关键字进行搜索）
 1. 实现前的一些想法
在完成上面的成熟版之后，我想既然可以向后台提交关键字获取suggestion，应该也可以向后台提交关键字进行搜索的。（PS：好像这个并没有用到什么技术，但是觉得很有意思，就顺便做了，而且在实现的过程中也出现了一些问题，顺便也总结一下。）
 2. 实现过程
回想之前我们做项目时可以通过<form>表单利用action向后台提交数据的。我打开Firebug，查看百度搜索时的传值情况，发现了百度搜索的action地址是http://www.baidu.com/s

![此处输入图片的描述][6]

于是就试着去写一下，在<form>标签里加上了一句action="http://www.baidu.com/s"，但是，提交了之后却变成百度首页，并没有搜索。于是，我又打开Firebug，查看百度搜索的传值情况

![此处输入图片的描述][7]

发现有两个参数有我要搜索的字段“百度云”，分别是“bs”和“wd”
于是我就将输入框的mane的值改成以上两个值去试。发现“wd”就是关键字对应的字段，再查看了一下地址栏果然也没错，有传值。

![此处输入图片的描述][8]

然后我再试着直接去修改百度搜索地址栏的值

![此处输入图片的描述][9]

发现搜索果然出现了，原来“wd”就是百度后台接受搜索关键字的字段。
  [1]: http://img.blog.csdn.net/20140914104219853?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGltX3RzYW5n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast
  [2]: http://img.blog.csdn.net/20140914111514255?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGltX3RzYW5n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast
  [3]: http://img.blog.csdn.net/20140914111950781?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGltX3RzYW5n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast
  [4]: http://img.blog.csdn.net/20140914112145878?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGltX3RzYW5n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast
  [5]: http://img.blog.csdn.net/20140914120227406?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGltX3RzYW5n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast
  [6]: http://img.blog.csdn.net/20140914120732312?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGltX3RzYW5n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast
  [7]: http://img.blog.csdn.net/20140914121429891?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGltX3RzYW5n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast
  [8]: http://img.blog.csdn.net/20140914121537517?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGltX3RzYW5n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast
  [9]: http://img.blog.csdn.net/20140914121819367?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGltX3RzYW5n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast
