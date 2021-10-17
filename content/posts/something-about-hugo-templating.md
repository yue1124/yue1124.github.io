+++
title = "Something About Hugo Templating"
date = 2021-10-16T04:58:56+08:00
draft = false
showInfo = true
showTOC = true
author = "bigyue"
tags = ["hugo", "template"]
categories = ["建站"]
description = "本文主要初步了解了Hugo中template的基本运行方式，大部分都是官方文档的直译，且可能存在错误。"
+++

Hugo 使用Go的`html/template`以及`text/template`标准库作为`templating`的基础。

所以如果要更深入地了解Hugo中的Templating，可以将标准库再深入看看。

## 基本语法

Go Templates基本应用再HTML文件中，使用`{{ }}`包裹，Go Template会解析对应地变量或者执行相应的方法。

**变量**

已存在的变量 `{{ .Title }}`

>  在Hugo中，会给每个template传递`Page`对象作为template中变量的默认作用域，所以上述的`{{ .Title }}`实际上访问的是`Page`对象中的`Title`参数。这个`Page`对象通常也称为`Context`上下文对象，所谓的`{{ . }}`其实指向的是上下文对象。

自定义的变量`{{ $address }}`

> 声明自定义的变量时赋值 `{{ $address := "123 st" }}`
>
> 为已声明的自定义变量再赋值 `{{ $address = "1 st" }}`

**方法**

`{{ FUNCTION ARG1 ARG2 .. }}` eg: `{{ add 1 2 }}` 将执行`add`方法，并将1，2作为参数

Hugo提供了一些基础的方法，当然也会教会我们如何扩展方法，不过使用扩展的方法可能需要重新编译Hugo。

## Includes方式

在模板中include其他模板是非常常见的功能。

在Hugo中提供`partial`方法和`template`方法来包含其他模板。

`{{ partial "hearder.html" . }}`

> 该方法将会包含`layouts/partials/header.html`文件
>
> 不要忘了最后的一个点 能够将上下文信息传递给所包含的模板

`{{ template "_internal/<TEMPLATE>.<EXTENSION>" }}`

> `template`是老版本的方法，现在这个方法只用来引入`internal templates`，这个东西之后再了解吧。

## 逻辑处理

### 迭代

使用`range`方法对`map，array，or slice`对象进行迭代，在模板中是大量使用的。

通过下面的例子学习一下用法吧

```html
{{ range $array }}
	{{ . }} <!-- 点表示变量array中的元素 -->
{{ end }}

{{ range $elem_val := $array }} <!-- 也可以用自定义变量的方式增加可读性 -->
	{{ $elem_val }}
{{ end }}

{{ range $elem_index, $elem_val := $array }} <!-- 对于数组，可以是 index, value -->
	{{ $elem_index }} -- {{ $elem_val }}
{{ end }}

{{ range $elem_key, $elem_val := $map }} <!-- 对于键值对，则是 key, value -->
	{{ $elem_key }} -- {{ $elem_val }}
{{ end }}

{{ range $array }}
	{{ . }}
{{ else }}
	<!-- range中的else是对所迭代元素是否为空的判断，只有当array是空的时候，才会渲染这里 -->
{{ end }}
```

### 条件判断

`if, else, with, or, and, not`能够提供条件判断的能力，其中`if, with`需要和`range`一样，使用`{{ end }}`表明结尾。

在条件判断中，以下情形被认为是`false`

- 布尔值`false`
- `0`
- 长度为0的`array, slice, map, string`

#### **`with`**

`with`是表达某物存在的意思，比如`with xx`，表示`xx`存在的话则...。这一点和`if xx`效果一样，只不过`with`还能检测变量是否存在。

```html
{{ with .Params.title }} <!-- 只有当title变量存在且不为false时，以下内容才会被渲染 -->
	<span>{{ . }}</span> <!-- 和range 一样 . 会被绑定成对应的title变量 -->
{{ else }}
	<!-- with中支持使用else对其他情况进行处理 -->
{{ end }}
```

#### **`if`**

`if .. else`以及`else if`都是支持的，这一点很好懂的，这里就不赘述了。

## 管道Pipe

所谓管道，其实是指`Action`的堆叠。简言之，当我们需要按顺序执行一连串动作时，就可以使用管道。

之所以称为管道，是因为前者的输出可以直接作为后者的输入，信息就像在管道中流动一样。

`{{ shuffle (seq 1 5) }}`比如这个没有使用管道的版本中，执行两个方法`seq`和`shuffle`，其中`seq`的输出作为`shuffle`的输入，因此可以用管道的方式这么写

`{{ (seq 1 5) | shuffle }}` 管道中每个`action`用`|`隔开，当有更多按顺序执行的方法时，使用管道方式将更具有可读性，简洁性。

比如

```html
{{ if or (or (isset .Params "title") (isset .Params "caption")) (isset .Params "attr") }}
{{ end }}
<!-- 以上能被重写成以下的样子 -->
{{ if isset .Params "capthion" | or isset .Param "title" | or isset .Params "attr" }}
{{ end }}
```

## 上下文

我们已经了解到`{{ . }}`实际上代表的是上下文，也知道在`range`，`with`中会将上下文绑定到局部变量中。此时则可以使用`{{ $. }}`表示全局的上下文变量。

> 所以在包含其他模板时，最后的`.`其实就是传递上下文变量给其他模板。

## 空白与注释

```html
<div>
    {{ .Title }}
</div>
<!-- 以上内容可能被渲染成如下 -->
<div>
    Hello, World!
</div>
<!-- 通过使用连字符 - 删除两边的空白 -->
<div>
    {{- .Title -}}
</div>
<!-- 渲染结果如下, 空白被删除了 -->
<div>Hello, World!</div>
```

使用`{{/* 这是一个注释 */}}`，可以在模板中写注释，注释最后将被忽略

HTML本身的注释语句最终也会被模板所删除，但如果HTML的注释语句中包含Go Template语句，也是会被执行的。 