---
layout: post
title: "有意思的javascript笔误"
date: 2013-05-28 22:33
comments: true
categories:  javascript
---
前两天被拉去查一个很怪的错，描述是“一个js文件压缩前和压缩后执行结果不一样”

查了很久锁定以下代码

{% codeblock lang:javascript %}
	s = s + +("...")
{% endcodeblock %}

一看就是笔误了，压缩后为

{% codeblock lang:javascript %}
	s=s++("...")
{% endcodeblock %}

空格被压缩后显然会抛语法错。但没压缩能正常运行就有点意思了，做了以下尝试

{% codeblock lang:javascript %}
+('...')
> NaN
-('...')
> NaN
'1' + +('...')
> "1NaN"
+(NaN)
> NaN
{% endcodeblock %}

那个多出来的加号，被解释成取数字正值，就像减号在数字前是取数字的负值一样。