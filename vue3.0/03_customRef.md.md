#### customRef

- 作用:实现一个自定义的ref,并对其依赖项跟踪和更新触发进行显示控制

像是下面的代码一样:
```vue
<template>
  <div class="lim">
    <div class="btns">
      <el-input type="text" v-model="keyWord">
      </el-input>
    </div>
    {{ keyWord }}
  </div>
</template>

<script setup>
// 使用vue内置提供的ref
// let keyWord = ref('hello')
import { customRef } from 'vue'

const myRef = (val) => {
  let timer
  // 接收一个函数,里面的函数必须返回一个对象
  return customRef((track, trigger) => {
    return {
      get: function () {
        console.log('获取了customRef')
        // 追踪改变,在return之前调用一下
        track()
        return val
      },
      set: function (newVal) {
        console.log('有人set了')
        clearTimeout(timer)
        // 实现防抖
        timer = setTimeout(() => {
          // 1.改数据
          val = newVal
          // 通知vue去重新解析模板
          // 2.通知解析模板
          trigger()
        }, 500)
      }
    }
  })
}

// 使用程序员自定义的ref
let keyWord = myRef('hello')
</script>
```
![[Pasted image 20230706233258.png]]
你可以通过自定义一个ref来实现某些功能,例如像这里的就是实现了防抖

但要注意其中的`customRef`接收的两个参数,
需要分别在get,set中分别在return之前进行调用一下



