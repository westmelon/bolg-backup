---

title: Markdown从入门到睡着

date: 2017-08-23 10:43:51

tags: markdown

comments: true

---

总感觉这个主题对markdown的支持不太友好。。。

<!-- more -->

# Markdown从入门到睡着

markdown 是一种标记语言，可以使用普通的编辑器编写，通过简单的标记语法，它可以使普通文本内容具有一定的格式。

今天就来入个门吧~

先说一些必要的知识点吧

Markdown中，以下字符支持使用反斜线转义：\ 反斜线  ` 反引号  * 星号  _ 下划线  {} 大括号  [] 中括号  () 小括号  # 井号  + 加号  - 减号（连字符）  . 句点  ! 感叹号

### markdown 本身也支持 html标签

对Markdown语法无法支持的格式，你可以直接用HTML。你不需要事先声明或者使用什么定界符来告诉
Markdown要写HTML了，你直接写就是了,如链接a, 图片img等。

### markdown也要区分块级元素和行内元素

>唯一的限制是那些块级HTML元素 ` 如 div,table, pre, p等等 ` 必须使用(至少1行)空行
与相邻内开(这句话啥意思。。。？f。。我的理解是像`` ``` `` 这种会生成块级元素标签的markdown语法标记要与内容用一个回车隔开！！！）， 并且块元素的开始和结束标签之前不要留有空格或TAB。

>注意一点，不要在块级HTML元素内使用Markdown格式化命令，Markdown不会处理它们。比如你不要在一个
HTML块中使用 * emphasis * 这样的Markdown格式化命令. 但是, 不同于这些块级HTML元素，在HTML行内元素内的
Markdown语法标记会被正确处理。

### 基本技巧


#### 代码

可以使用`` `Tab键上面的那个键` ``来高亮某些关键字
同样可以使用`` ``` ``加在一段代码的前后，使一段代码高亮，听说还可以指定一种语言？
大概像这样：

<pre>
<span>```javascript</span><code>
$(document).ready(function () {
    alert('hello world');
});
</code><span>``` </span>
</pre>


效果：

```javascript
$(document).ready(function () {
    alert('hello world');
});
``` 

使用<code>&nbsp;&nbsp;&nbsp;&nbsp;</code>四个空格可以达到与`` ``` ``相同的效果：


<pre>
<i>    </i><code>$(document).ready(function () {</code>
<i>    </i><code>    alert('hello world');</code>
<i>    </i><code>});</code>
</pre>

还可以使用`<pre></pre>`标签达到相同的效果：

``` 
<pre>
$(document).ready(function () {
    alert('hello world');
});
</pre>
``` 

效果是一样的

不过这种方式是因为markdown本身支持html标签，而且`` ``` ``本身在翻译成html时就会变成`<pre></pre>`

#### 标题


文章内容较多时，可以用标题分段：

##### 1.类Setext形式标题
<pre>标题1
<span>======</span>
标题2
<span>------</span></pre>

说明：
Ⅰ.标题前后需要添加空行，也就是回车符啦；
Ⅱ.任何数量的 = 和 - 都可以有效果（瞎说，一个就不行）；
Ⅲ.相比类atx形式：
类 Setext形式和类atx形式的一阶和二阶标题下面会多出一条分隔线；
类atx形式的标题前后不需要添加空行。

##### 2.类atx形式标题

类 Atx 形式则是在行首插入 1 到 6 个 # ，对应到标题 1 到 6 阶。

大概像这样

###### 啥
##### 你说啥
#### 再大声点
### 这里信号不好
## 红红火火恍恍忽忽
# 哇，豆豆怎么这么帅

源码：

<pre>
###### 啥
##### 你说啥
#### 再大声点
### 这里信号不好
## 红红火火恍恍忽忽
# 哇，豆豆怎么这么帅
</pre>

当然你觉得高兴的话在后面补上同等数量的`#`也是可以的

<pre>
###### 啥 ######
##### 你说啥 #####
#### 再大声点 ####
### 这里信号不好 ###
## 红红火火恍恍忽忽 ##
# 哇，豆豆怎么这么帅 #
</pre>


#### 粗斜体


<pre><span>*</span><i>斜体文本</i><span>*</span>    <span>_</span><i>斜体文本</i><span>_</span>
<span>**</span><b>粗体文本</b><span>**</span>    <span>__</span><b>粗体文本</b><span>__</span>
<span>***</span><i><b>粗斜体文本</b></i><span>***</span>    <span>___</span><i><b>粗斜体文本</b></i><span>___</span></pre>

#### 链接

常用链接方法

[秋名山](https://baike.baidu.com/item/%E7%A7%8B%E5%90%8D%E5%B1%B1/82423)

<https://baike.baidu.com/item/%E7%A7%8B%E5%90%8D%E5%B1%B1/82423>

食用方法

<pre>文字链接 <span>[链接名称](http://链接网址)</span>
网址链接 <span>&lt;http://链接网址&gt;</span></pre>

高级链接技巧

<pre>这个链接用 1 作为网址变量 <span>[Google][1]</span>.
这个链接用 yahoo 作为网址变量 <span>[Yahoo!][yahoo]</span>.
然后在文档的结尾为变量赋值（网址）

<i>  </i><span>[1]:</span><i> </i>http://www.google.com/
<i>  </i><span>[yahoo]:</span><i> </i>http://www.yahoo.com/</pre>

#### 图片

跟链接的方法区别在于前面加了个感叹号 `!`，非常好记!

<pre><span>![图片名称](http://图片网址)</span></pre>

当然，你也可以像网址那样对图片网址使用变量

<pre>这个链接用 1 作为网址变量 [Google][1].
然后在文档的结尾位变量赋值（网址）

<i> </i>[1]:<i> </i>http://www.google.com/logo.png</pre>

![彩虹](http://wx3.sinaimg.cn/mw690/d2d25673gy1fisi5lmj2hj207c07cdfw.jpg)

#### 列表

普通无序列表

<pre><span>-</span><i> </i>列表文本前使用 [减号+空格]
<span>+</span><i> </i>列表文本前使用 [加号+空格]
<span>*</span><i> </i>列表文本前使用 [星号+空格]</pre>

- 韩寒会画画
+ 后悔画韩红
* 红红火火恍恍忽忽

普通有序列表

<pre><span>1.</span><i> </i>列表前使用 [数字+空格]
<span>2.</span><i> </i>我们会自动帮你添加数字
<span>7.</span><i> </i>不用担心数字不对，显示的时候我们会自动把这行的 7 纠正为 3</pre>

1. 列表前使用 [数字+空格]
2. 我们会自动帮你添加数字
7. 不用担心数字不对，显示的时候我们会自动把这行的 7 纠正为 3

列表嵌套

<pre><span>1.</span><i> </i>列出所有元素：
<i>    </i><span>-</span><i> </i>无序列表元素 A
<i>        </i><span>1.</span><i> </i>元素 A 的有序子列表
<i>    </i><span>-</span><i> </i>前面加四个空格
<span>2.</span><i> </i>列表里的多段换行：
<i>    </i>前面必须加四个空格，
<i>    </i>这样换行，整体的格式不会乱
<span>3.</span><i> </i>列表里引用：

<i>    </i><span>&gt;</span> 前面空一行
<i>    </i><span>&gt;</span> 仍然需要在 <span>&gt; </span> 前面加四个空格

<span>4.</span><i> </i>列表里代码段：

<i>    </i><span>```</span>
<i>    </i>前面四个空格，之后按代码语法 <span>```</span> 书写
<i>    </i><span>```</span>

<i>        </i>或者直接空八个，引入代码块
</pre>


1. 列出所有元素：
    - 无序列表元素 A
        1. 元素 A 的有序子列表
    - 前面加四个空格
2. 列表里的多段换行：
    前面必须加四个空格，
    这样换行，整体的格式不会乱
3. 列表里引用：

    > 前面空一行
    > 仍然需要在 >  前面加四个空格

4. 列表里代码段：

    ``` 
    前面四个空格，之后按代码语法 ``` 书写
    ``` 

        或者直接空八个，引入代码块


#### 引用

普通引用

<pre><span>&gt;</span> 引用文本前使用 [大于号+空格]
<span>&gt;</span> 折行可以不加，新起一行都要加上哦</pre>

> “也许每一个男子全都有过这样的两个女人，至少两个。娶了红玫瑰，久而久之，红的变了墙上的一抹蚊子血，白的还是"床前明月光"；娶了白玫瑰，白的便是衣服上沾的一粒饭黏子，红的却是心口上一颗朱砂痣”。

引用里嵌套引用

<pre><span>&gt;</span><i> </i>最外层引用
<span>&gt;</span><i> </i><span>&gt;</span><i> </i>多一个 <span>&gt;</span><i> </i>嵌套一层引用
<span>&gt;</span><i> </i><span>&gt;</span><i> </i><span>&gt;</span><i> </i>可以嵌套很多层</pre>

> 我在哪？
> > 我是谁？
> > > 我到哪里去？


#### 换行

如果另起一行，只需在当前行结尾加 2 个空格
就象这样?

如果是要起一个新段落，只需要空出一行即可。

#### 分隔符

如果你有写分割线的习惯，可以新起一行输入三个减号-。当前后都有段落时，请空出一行：

<pre>前面的段落

<span>---</span>

后面的段落</pre>


### 稍微高级一丢丢的技巧

#### 行内 HTML 元素

目前只支持部分段内 HTML 元素效果，包括 ` <kdb> <b> <i> <em> <sup> <sub> <br> ` ，如

键位显示

<pre class="">使用 <span>&lt;kbd&gt;Ctrl&lt;/kbd&gt;</span>+<span>&lt;kbd&gt;Alt&lt;/kbd&gt;</span>+<span>&lt;kbd&gt;Del&lt;/kbd&gt;</span> 重启电脑
</pre>

使用 <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>Del</kbd> 重启电脑

尴尬，好像没有样式。。。

代码块

上面说过了

粗斜体

<pre><span>&lt;b&gt;</span> Markdown 在此处同样适用，如 <span>*</span>加粗<span>*</span> <span>&lt;/b&gt;</span></pre>

<b> Markdown 在此处同样适用，如 *加粗* </b>

符号转义

如果你的描述中需要用到 markdown 的符号，比如 `_` `#` `*` 等，但又不想它被转义，这时候可以在这些符号前加反斜杠，如 `\_` `\#` `\*` 进行避免。

<pre><span>\_</span>不想这里的文本变斜体<span>\_</span>
<span>\*\*</span>不想这里的文本被加粗<span>\*\*</span>
</pre>

\_不想这里的文本变斜体\_
\*\*不想这里的文本被加粗\*\*

公式

当你需要在编辑器中插入数学公式时，可以使用两个美元符 $$ 包裹 TeX 或 LaTeX 格式的数学公式来实现。提交后，问答和文章页会根据需要加载 Mathjax 对数学公式进行渲染。如：

<pre><span>$$</span><i> </i>x = {-b \pm \sqrt{b^2-4ac} \over 2a}.<i> </i><span>$$</span>

<span>$$</span>
x \href{why-equal.html}{=} y^2 + 1
<span>$$</span>
</pre>



同时也支持 HTML 属性，如：

<pre><span>$$</span><i> </i>(x+1)^2 = \class{hidden}{(x+1)(x+1)}<i> </i><span>$$</span>

<span>$$</span>
(x+1)^2 = \cssId{step1}{\style{visibility:hidden}{(x+1)(x+1)}}
<span>$$</span></pre>
