- 作用:实现祖孙件通信
- 套路:父组件有一个provide选项来提供数据,子组件有一个inject选项来开始使用这些数据
- 具体写法:

祖组件中:
```vue
<template>
  <div class="lim">
    <h2>祖组件</h2>
    <child></child>
    名字:{{ car.name }}
    <br>
    价格:{{ car.price }}
    <br>
    <el-button @click="car.price++">价格增加</el-button>

  </div>
</template>

<script setup>
import Child from './child.vue'
import { provide, reactive } from 'vue'

const car = reactive({
  name: '车',
  price: '40'
})

// 给自己的后代组件传入数据
provide('car', car)
</script>
```

child:
```vue
<template>
  <div class="lim">
    <h2>子组件</h2>
    <el-text>子组件中获取的数据:</el-text>
    {{ car }}
    <br>
    子组件这里获取到了,而且还是响应式的
  </div>
</template>
<script setup>
import { inject } from 'vue'

let car = inject('car')
</script>
```
run:

![](http://blog.icestone.top/upload_b5727962b431c2186d871322230fca0c.png)

当然,如果存在子组件的子组件,也是可以直接获取到其上一级上一级的组件
child:
```vue
<template>
  <div class="lim">
    <childchild></childchild>
    <h2>子组件</h2>
    <el-text>子组件中获取的数据:</el-text>
    {{ car }}
    <br>
    子组件这里获取到了,而且还是响应式的
  </div>
</template>
<script setup>
import { inject } from 'vue'
import Childchild from './childchild.vue'

let car = inject('car')
</script>
```
childchild:
```vue
<template>
<div class="lim">
  <h2>孙组件</h2>
  <code>
    孙组件中接收的值:
    car:{{ car }}
  </code>
</div>
</template>
<script setup>
import { inject } from 'vue'

let car = inject('car')
</script>
```
![](http://blog.icestone.top/upload_951f49b8dd2754d1b8215beb93fa70a0.png)