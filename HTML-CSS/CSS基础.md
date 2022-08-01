# CSS基础

> # 基础选择器



* ### 选择器用于选择html里的元素修改样式**



##  **标签选择器**

1. 利用标签名字选择

2. 会选择整个html的标签

   ​     **代码演示：**

```css
        /* 所有p标签设置颜色：白色 */
        p {
           color: #ffffff;
        }
```



## 类选择器

1. 在标签上使用”class“属性设置类

2. 使用".类名"的方式调用设置样式

   **代码演示：**

```css
        /* 所有class="box"的标签设置颜色：白色 */
        .box {
           	  color: #ffffff;
        }

		<div class="box">盒子</div>
```



## **id选择器**

1. 在标签上使用“id”设置id值

2. 使用“#id名”调用设置背景

3. id属性只能在html出现一次

   **代码演示：**

```css
        /* 使id="box"的标签设置颜色：白色 */
        .box {
           	  color: #ffffff;
        }

		<div id="box">盒子</div>
```



## **通配符选择器**

1. 使用“*”号设定所有标签的样式

   **代码演示：**

```css
        /* 使所有的标签设置颜色：白色 */
        * {
           color: #ffffff;
        }
```



----



> # 文本样式



- ### 设置文本的风格、缩进、对齐等样式



## 首行缩进

1. css中首行缩进使用“text-indent”设置

2. 使用px或者em进行缩进

3. 只能给一行的标签使用

   **代码演示：**

```css
        /* p标签首行缩进2个字符 */
        p {
           text-indent: 2em;
        }
```



## 对齐方式

1. css中设置对齐方式使用”text-align“

2. 使用“center”、”left“、”right“、设置

3. 只能给独占一行的标签使用

   **代码演示：**

```css
        /* p标签居中 */
        p {
           text-align: center;
        }
```



## 文本装饰

1. 使用"text-decoration"设置文本装饰

2. 默认为“none”

   |    属性值    |     说明     |
   | :----------: | :----------: |
   |     none     | 默认：无装饰 |
   |  underline   |    下划线    |
   |   overline   |    上划线    |
   | line-through |    删除线    |

   **代码演示：**

   ```css
           /* 删除a标签的下划线 */
           a {
              text-decoration: none;
           }
   ```




## 行高

1. 使用“line-height”设置文本的行高
2. 定义文字、上、下共有的高度



----



> # 字体样式



- ### 设置文字大小、粗细、字体等样式

  

## 字体大小

1. 使用”font-size“设定字体的px大小

   **代码演示：**

```css
        /* 设置p标签文字为15px */
        p {
           font-size: 15px;
        }
```



## 字体粗细

1. 使用”font-weight“设定字体粗细数值
2. 设置400为正常，700为加粗


      **代码演示：**

```css
        /* 设置p标签文字加粗 */
        p {
           font-weight: 700;
        }
```



## 字体风格

1. 使用”font-style“设定字体风格
2. 默认值为“normal”，“italic”为斜体


      **代码演示：**

```css
        /* 设置p标签文字变成斜体 */
        p {
           font-style: italic;
        }
```



## 设置字体

1. 使用”font-family“设定文字字体
2. 多个字体用空格隔开
3. 字体从左往右解析，英文字体尽量写左边。
4. 多个单词字体放进双引号

   **代码演示：**


```css
        /* 设置p标签文字变成斜体 */
        p {
           font-family: "Microsoft yahei" 黑体;
        }
```



## font复合写法

1. 可使用”font“复合写字体样式

2. 按照“style”，"weight"，"size/line-height"，"font-family"顺序写

3. "size/line-height"，"font-family"两个属性必须写

   **代码演示：**


```css
        /* 设置p标签文字变成斜体、加粗、文字大小16像素、行高30px、字体为微软雅黑和黑体 */
        p {
           font: italic 700 16px/30px "Microsoft yahei" 黑体;
        }
```



















----



