---
layout: post
title: "略学习Lisp的quote和backquote"
date: 2013-03-18 23:59
comments: true
categories: lisp
---
略学习了lisp里面奇怪的符号集，解释这些符号上，人类的语言基本是苍白的。

[第一份参考](http://www.lispworks.com/documentation/HyperSpec/Body/02_df.htm)来自lispworks的文档，对[`'@,]这几种符号做了定义。

[第二份中文参考](http://wenku.it168.com/d_000648782.shtml) 被到处抄袭。对quote和backquote做了很好的中文说明，嵌套quote部分惨不忍睹。

[第三份参考](http://stackoverflow.com/questions/7549550/using-two-backquotes-and-commas-common-lisp) 对嵌套quote做了很好地解释。

{% codeblock 贴一些自己的学习代码 lang:lisp %}
CL-USER> (list 1 2)
(1 2)
CL-USER> '(1 2) 
(1 2) ;Quote与list相同
CL-USER> `(1 2)
(1 2) ;Backquote与list相同

CL-USER> (let ((x 1)) '(,x))
; Evaluation aborted on #<CCL::SIMPLE-READER-ERROR #xC78878E>. ;Quote不能和comma联用
CL-USER> (let ((x 1)) `(,x))
(1) ;Backquote可以和comma联用

CL-USER> (let ((x `(1 2))) `(,@x))
(1 2) ;BackQuote和comma-at-sign联用

CL-USER> (let ((x `(1 2))) `(,x))
((1 2)) ;用comma替代comma-at-sign，列表不会被展开
{% endcodeblock %}