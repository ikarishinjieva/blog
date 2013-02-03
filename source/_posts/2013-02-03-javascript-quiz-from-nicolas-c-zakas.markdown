---
layout: post
title: "Javascript quiz from Nicolas C.Zakas"
date: 2013-02-03 23:36
comments: true
categories: javascript
---

Nicolas C.Zakas写了五个js语言用例的分析。举一例：

{% codeblock lang:js %}
function b(x, y, a) {
    arguments[2] = 10;
    alert(a);
}
b(1, 2, 3);
{% endcodeblock %}

结果是10,。出乎意料。