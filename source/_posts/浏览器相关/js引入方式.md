---
title: js引入方式
date: 2022-09-19 00:30:00
categories:
  - 浏览器
tags:
  - html
  - js
  - 浏览器
---

# script 标签内部编写代码

//此处省略

# 行内引入

```js
<a href="javascript:console.log('xxx')"></a>

<button onclick="console.log('xxx')"></button>
```

# script 外部资源引入

- async: 当我们在 script 标记添加 async 属性以后,浏览器遇到这个 script 标记时会继续解析 DOM,同时脚本也不会被 CSSOM 阻止,即不会阻止 CRP。
- defer: 与 async 的区别在于,脚本需要等到文档解析后（ DOMContentLoaded 事件前）执行,而 async 允许脚本在文档解析时位于后台运行（两者下载的过程不会阻塞 DOM,但执行会）。
