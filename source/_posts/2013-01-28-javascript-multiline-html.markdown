---
layout: post
title: "Javascript中写多行HTML"
date: 2013-01-28 23:00
comments: true
categories: javascript
---
需要写多行HTML string到javascript中，怎么排版代码都很难看，最后发现这个利用注释凶残的方法。(不过此方法对firefox无效，firefox中function#toString会吃掉注释)

{% codeblock lang:js %}
Function.prototype.getMultiLine = function() {  
	var lines = new String(this);  
	lines = lines.substring(lines.indexOf("/*") + 3, lines.lastIndexOf("*/"));  
	return lines;  
}  
  
var ffff = function() {  
	/* 
	张三去倒水
 
	 天哪！ 
	 */  
 }  
   
document.write(ffff.getMultiLine());
{% endcodeblock %}
一些参考：

[代码抄自这里](http://www.cnblogs.com/starlet/archive/2010/05/24/1742572.html)