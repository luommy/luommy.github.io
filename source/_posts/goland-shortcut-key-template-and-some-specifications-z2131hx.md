---
title: Goland 快捷键、模板与一些规范
date: '2023-08-23 22:42:24'
updated: '2023-09-22 00:32:42'
excerpt: 记录下一些不值一提技巧....
tags:
  - goland技巧
permalink: /post/goland-shortcut-key-template-and-some-specifications-z2131hx.html
comments: true
toc: true
---

# Goland 快捷键、模板与一些规范

题记： 快捷键的使用可以大大提高效率，良好的注释对项目后续的开发维护工作也是十分必要。本文档旨在明确项目开发过程中go代码的注释规范，并提供基于goland的注释模板设置指导。便于开发人员快速配置环境，高效、合规开展工作。

> 规范部分随缘整理.....

## 我的配置

1. Go文件采用goland自带的go文件和代码模板

    ```
    /**
     * @Description //TODO
     * @Author luommy
     * @Date ${DATE} ${TIME}
     **/

    package ${GO_PACKAGE_NAME}

    func main() {

    }
    ```

​![image](http://127.0.0.1:55135/assets/image-20230922001754-vgrw09w.png)

‍

2. 方法、结构体、接口注释采用插件实现

‍

## 自定义快捷键与注释模板

​自定义快捷键：`CTRL+J`​

​![image](http://127.0.0.1:55135/assets/image-20230815101928-bc1rjz0.png)​

### 添加新建类的注释模板

路径：File -> Settings -> File and Code Templates

​![image](http://127.0.0.1:55135/assets/image-20230921234433-29gnid1.png)​

添加如下信息：

```
package ${GO_PACKAGE_NAME}
/**
 * @Description //TODO
 * @Author luommy
 * @Date ${DATE} ${TIME}
 **/
func main() {

}
```

### 添加方法注释模板

‍

**Live Templates**

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202309251650170.png)

## go templates 内置函数

‍

---

### 在模板变量中使用的预定义函数

有很多都用不到的，感觉并不好用，没啥能用到的场景

|Item|Description|
| --------| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|​`lowercaseAndDash(String)`​​|将驼峰字符串转换为小写，并插入连接符作为分隔符。例如,`lowercaseAndDash(MyExampleName)`​​返回`my-example-name`​​.|
|​`snakeCase(String)`​​|将驼峰字符串转化为蛇形字符串，例如,`snakeCase(fooBar)`​​返回`foo_bar`​​.|
|​`spaceSeparated(String)`​​|将字符串转化为小写并插入空格作为分隔符，例如,`spaceSeparated(fooBar)`​​returns`foo bar`​​.|
|​`underscoresToCamelCase(String)`​​|将蛇形字符串转化为驼峰字符串. 例如,`underscoresToCamelCase(foo_bar)`​​返回`fooBar`​​.|
|​`underscoresToSpaces(sParameterWithSpaces)`​​|将字符串中的下划线替换为空格. 例如,`underscoresToSpaces(foo_bar)`​​返回`foo bar`​​.|
|​`camelCase(String)`​​|将字符串转化为驼峰法. 例如,`camelCase(my-text-file)`​​,`camelCase(my text file)`​​, 和`camelCase(my_text_file)`​​均返回`myTextFile`​​.|
|​`capitalize(String)`​​|将参数的第一个字母大写。|
|​`capitalizeAndUnderscore(sCamelCaseName)`​​|根据驼峰法分割参数，将各个部分转化为大写，并插入下划线。例如,`capitalizeAndUnderscore(FooBar)`​​返回`FOO_BAR`​​.|
|​`classNameComplete()`​​|这个表达式代替了在变量位置完成类名。|
|​`clipboard()`​​|返回系统剪贴板的内容。|
|​`complete()`​​|引用代码完成。|
|​`completeSmart()`​​|调用变量位置的智能类型完成。|
|​`concat(expressions...)`​​|返回传递给函数的所有字符串作为参数的级联。|
|​`date(sDate)`​​|以指定的格式返回当前系统日期。<br />如果没有参数，则以默认的系统格式返回当前日期。<br /> ![]([https://upload-images.jianshu.io/upload_images/2224826-8ea59fac927794ed.png?imageMogr2/auto-orient/strip](https://upload-images.jianshu.io/upload_images/2224826-8ea59fac927794ed.png?imageMogr2/auto-orient/strip)|
|​`decapitalize(sName)`​​|用相应的小写字母替换参数的第一个字母。|
|​`enum(sCompletionString1,sCompletionString2,...)`​​|返回在模板扩展时建议完成的逗号分隔字符串的列表。|
|​`escapeString(sEscapeString)`​​|Escapes the string specified as the parameter.|
|​`expectedType()`​​|Returns the expected type of the expression into which the template expands. Makes sense if the template expands in the right part of an assignment, after`return`​​, etc.|
|​`fileName()`​​|返回当前文件（包括扩展名）|
|​`fileNameWithoutExtension()`​​|返回当前文件（不包括扩展名）|
|​`firstWord(sFirstWord)`​​|返回作为参数传递的字符串的第一个单词。|
|​`lineNumber()`​​|返回当前行数。|
|​`substringBefore(String,Delimiter)`​​|删除指定分隔符之后的扩展名，只返回文件名。这对测试文件名是有帮助的，例如,`substringBefore($FileName$,".")`​​returns`component-test`​​in`component-test.js`​​).|
|​`time(sSystemTime)`​​|以指定的格式返回当前系统时间。|
|​`timestamp()`​​|返回当前系统时间戳|
|​`user()`​​|返回当前用户|

‍

‍

## 中英互译插件

目前觉得挺好用的

支持自定义快捷键：setting-keymap-plugins

习惯设置Alt+T 一键互译

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202309251650861.png)​

## 注释插件：Goanno

插件市场安装，通过tools进行配置——Goanno Setting

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202309251650233.png)​​​

### 方法、接口、结构体注释模板配置

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202309251650305.png)​

配置内容如下：

```
Normal Method 配置内容：
/**
   @Title ${function_name} 
   @Description ${todo} 
   @Author luommy ${date} 
   @Param ${params} 
   @Return ${return_types} 
**/

interface配置内容
// ${interface_name} 

Interface Method 配置内容
// @Title ${function_name} 
// @Description ${todo} 
// @Author luommy ${date} 
// @Param ${params} 
// @Return ${return_types} 

Struct配置内容
// ${struct_name} 

Struct Field  不做配置

```

配置完成点击submit

验证注释 在方法、结构体、接口上 

**win**使用快捷键: `ctrl +alt +/`​

**mac**使用快捷键:`control + commond + /`​  

可自动生成注释 如下截图:

![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202309220034329.png)​

## AI 插件 CodeGeeX 

[官网](https://codegeex.cn/)

几个重点的功能：

1. 多语言代码之间翻译，无缝转换
2. 注释生成代码/自动补全代码
3. AI问答
