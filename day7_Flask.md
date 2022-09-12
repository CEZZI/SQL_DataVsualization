# day7 Flask和JavaScript概述

## 1. Pycharm克隆项目

在pycharm的欢迎页中Get from VCS（从版本控制中获取项目）

- `URL(Universal Recourse Locator)`：统一资源控制符 ---> 能够唯一标识一个资源

## 2. 创建虚拟环境

- 生成依赖项清单
  在终端里输入`pip freeze >requirements.txt`,建立一个requirements.txt

- `pip install -r requirements.txt` 可以直接下载依赖项的包

- 在pycharm中选择√，勾选需要纳入版本控制的文件，相当于执行了`git add`操作

- 在pycharm中`push`后，才能在网站上看到

## 3. Flask与渲染页面

以下是比较老旧的做法

### main.py

- 利用`render_template('html文件名',所需变量名=selected_fruits)` 作为返回值来渲染html页面

```python
"""
Have a nice day！
main- falsk 初体验

"""
import random

from flask import Flask, render_template

app = Flask(__name__)

# 定义一个借口
@app.route('/')
def say_hello():
    fruits_list = [
        '苹果', '香蕉', '山竹', '榴莲', '杨梅', '草莓', '蓝莓', '石榴', '番茄',
        '杨桃', '哈密瓜', '西瓜', '葡萄', '百香果', '樱桃', '车厘子', '荔枝'
    ]
    fruits_count = random.randrange(3, 6)
    selected_fruits = random.sample(fruits_list, fruits_count)

    return render_template('index.html',selected_fruits=selected_fruits)
    # 渲染母版页面;输出名必须和页面需求的变量 一模一样



if __name__ == '__main__':
    app.run(debug=True)
    # 项目跑起来

```

### index.html

- 代码以`<html>`开始，到`</html>`结束
- 需要写的内容放在`<body>和</body>`之间

```html
<!DOCTYPE html>
<html lang="en">
<head>
        <meta charset="UTF-8">
        <title>首页</title>
    </head>
    <body>
    <h1>今日推荐的水果</h1>
    <hr>
    <ul>
        {% for fruit in selected_fruits %}
        <li>{{ fruit }}</li>
        {% endfor %}

    </ul>>
    </body>
</html>
```

## 4. 通过CSS渲染页面

### 1. 前端页面的构成

前端页面 = Tag + CSS + JavaScript

- Tag - 承载内容 - content
- CSS - 层叠样式表 - 页面显示 - display
  - 内嵌样式表（不推荐）
  - 外部样式表（单独的文件，使用link引入）
- JS - 交互行为 - behavior

### 2.通过选择器控制页面的样式

- 通配符选择器

  可以匹配页码中给所有的标签/元素
- 标签选择器
  - `h1{...}`
- ID选择器
  - `#fruits{...}` 匹配ID等于fruits的标签
- 父子选择器
  - `#fruits>li{...}`
- `text-algin`: 文本的位置
- `margin_top`: 和顶端的距离
- `width&hight`：宽度和高度
- `font-size`：字体大小

```html
<!DOCTYPE html>
<html lang="en">
<head>
        <meta charset="UTF-8">
        <title>首页</title>
        <style>
            /*通配符选择器*/
            *{
                margin: 0;
                padding: 0;
            }
            /* 标签选择器（元素选择器）*/
            h1{
                text-align: center;
                color:pink;
                margin-top: 20px;
            }
            /*ID选择器*/
            #fruits{
                width: 320px ;
                margin:0 auto;
            }
            /*父子选择器*/
            #fruits>li{
                text-align:center;
                list-style: none;
                width: 320px;
                height: 40px;
                background-color: bisque;
                color: rgb(202, 100, 100);
                margin-top: 2px;+
                line-height: 40px;
                font-size: 22;
            }
        </style>
    </head>
    <body>
    <h1>今日推荐的水果</h1>
    <hr>
    <ul id="fruits">
        {% for fruit in selected_fruits %}
        <li>{{ fruit }}</li>
        {% endfor %}

    </ul>>
    </body>
</html>
```

## 5. JavaScript概述

### 5.1 浏览器中的JavaScript有三个要素

- ECMAScript - ES -语法规范（关键字、运算符、分支、循环、对象、函数...）
- BOM - Browser Object Model -浏览器对象模型 - window
  - location / history / screen / navigator
  - alert() / prompt() / confirm() / open() /close()
- DOM - Document Object Model - 文件对象模型 - document
  - querySelector
  - createElement - 创建新标签
  - appendChild() / insertBefore() - 添加标签内容
  - removeChild() - 删除标签
  - textContent / innerHTML - 修改标签内容
  - style - 修改标签样式

### 5.2 JavaScript的简介

#### 教程： <https://www.runoob.com/js/js-intro.html>

- 变量命名: `let 变量名`
- 函数: `function 函数名() {...}`
- 按钮: `<button onclick="sayHello()">按钮名</button>`
- 弹窗: `window.alert(内容)`
  - 输入：`window.prompt('内容')`

- 相应事件 ---> 按钮 ---> 输入---> 弹窗

#### 驼峰命名法（Camel Notation）

- 命名变量和函数：第一个单词全小写，从第二个单词开始每个单词的首字母大写
- 命名类：每个单词的首字母大写

1. 直接写入html输出流

```HTML
document.write("<h1>这是一个标题</h1>");
document.write("<p>这是一个段落。</p>");
```

2. 对事件的反映

```HTML
<button type="button" onclick="alert('欢迎!')">点我!</button>
```

3. 改变html的内容

document对象的querySelector和querySelectAll可以通过CSS选择器获取页面元素，前者获取第一个元素，后者获取元素的列表

```HTML
x=document.getElementById("demo");  //查找元素
x.innerHTML="Hello JavaScript";    //改变内容
```

#### 例子

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>

    <button onclick="sayHello()">确定</button>


    <script>
        // 驼峰命名法（Camel Notation）：第一个单词全小写，从第二个单词开始每个单词的首字母大写
        // 命名变量和函数：第一个单词全小写，从第二个单词开始每个单词的首字母大写
        // 命名类：每个单词的首字母大写
        function sayHello(){
            let name = window.prompt('请输入你的名字：')
            window.alert(name + '你好漂亮'+'!')
            let h1 = document.querySelector('h1')
            h1.textContent = 'Goodbye,my friend'
            h1.style.color = 'purple'
            h1.style.fontSize = '2cm'

        }

        for (let i =0; i<4; ++i){
            if(i % 2==0) {
                document.write('<h1 style="color: pink">Good Luck to You</h1>')  
            } else {
                document.write('<h1 style="color: blue">Have a nice day</h1>')
            }                   
        }

        //输出一个乘法口诀表
        document.write('<table>')
        for (let i = 0; i<10;++i){
            for(let j=0;j<=i;++j)
            {
                let content = i + '*' +j +'=' + i*j
                document.write('<td>' + content + '</td>')
            }
        document.write('</tr>')
    }

        document.write('</table>')
    </Script>
</body>
</html>
```

利用原生的JavaScript页面渲染太麻烦了，利用框架写JavaScript才方便

## 5.3 vue初体验

v-for

- JS里面花括号是创建对象的语法
  - JSON ---> JavaScript Object Notation
  - let person ={name:'cao',age:22,eat:function()}

- splice()
- 双向绑定： 在文本康
