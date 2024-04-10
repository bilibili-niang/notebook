---
tags:
  - scss
  - 前端
  - theme
date: 2024年4月3日14:45:48
---

```scss
/** 深浅色系 - light  **/:root[theme='light'],  
[theme='light'] {  
  --kacat-color-bg-50: #fdfdfd;  
  --kacat-color-bg-100: #fcfcfc;  
}


/** 深浅色系 - dark  **/:root[theme='dark'],  
[theme='dark'] {  
  --kacat-color-bg-50: #4a4a4a;  
  --kacat-color-bg-100: #3c3c3c; 
}
```
或者通过html标签的`primary-color`属性来修改主题色:
```scss
/** 主题色 - zinc  **/:root[primary-color='zinc'],  
[primary-color='zinc'] {  
  --kacat-color-primary: #71717a;  
  --kacat-color-primary-50: #fafafa;  
}
  
/** 主题色 - neutral  **/:root[primary-color='neutral'],  
[primary-color='neutral'] {  
  --kacat-color-primary: #737373;  
  --kacat-color-primary-50: #fafafa; 
}
```
关于`primary-color`标签,你可以在html标签身上看到
