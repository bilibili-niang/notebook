---
tags:
  - 前端
  - webpack
---

1.什么是异步加载模块?  
同步加载: import $ from 'jquery';  
例如: 在index.js中导入了10个模块, 那么只要index.js被执行, 就会一次性将10个模块加载进来   
异步加载: import('jquery').then(({default: $ }) => {使用模块代码});  
例如: 在index.js中导入了10个模块, 那么哪怕index.js被执行, 也要看是否满足加载条件才去加载  

特点:  
对于异步加载的模块无需配置, webpack会自动分割  
https://webpack.js.org/guides/lazy-loading/  