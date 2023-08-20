#### 关于

模块联邦sdk化，免构建、热更新、工具链无关的微模块方案

[文档](https://tnfe.github.io/hel/)

### 开发并发布到npm

开发远程 vue 组件
####克隆模板库

克隆远程 vue 组件模板项目为`rvc-xxx`（名字请按实际需要修改，此处仅做示例）

```
https://github.com/hel-eco/hel-tpl-remote-vue-comp
```

改 package.json

将`name`和`appGroupName`改为 hel 模块组名

```
  "name": " rvc-xxx",
  "appGroupName": " rvc-xxx",
```

开发组件

在`src/components`目录下新增组件，并`src/components/index`文件里导出即可

> 例子

像是这里已经发布的一个版本(已打包):
[link](https://unpkg.com/browse/ice-test-remote@1.5.2/)
我这里只修改了:
`src/components/index.js`:
```javascript
/*  
|--------------------------------------------------------------------------  
|  
| 这些组件暴露给使用方  
|  
|--------------------------------------------------------------------------  
*/  
import App from './App.vue';  
import HelloWorld from './HelloWorld.vue';  
import Button from '@/components/button.vue'  
import add from './tools'  
  
export default {  
  App,  
  HelloWorld,  
  Button,  
  add  
};
```

`src/components/button.vue`:
```vue
<template>  
  <div class="button">  
    <slot></slot>  </div></template>  
  
<script>  
export default {  
  name: "button"  
}  
</script>  
  
<style scoped>  
.button {  
  border-radius: 4px;  
  padding: 0 3px;  
  margin: 3px 6px;  
}  
</style>
```
`src/components/tools/index.js`:
```javascript
export default {  
    add: function (a, b) {  
        return a + b  
    }  
}
```

然后:
```shell
npm build
npm publish
```
即可,但是推送可能会有一定延迟
上面的例子是可以被调用的,这里写了一个button,一个add的方法

> 注意

在build之前改版本号
build之后的生成物是个版本号有关系的,如果版本号冲突,是无法推送的
### 调用(vue中)

我这里使用的例子:
[link](https://codesandbox.io/s/demo-load-remote-vue-comp-st0295)
会在上面的代码基础上进行增加并调用自己的远程库:

这里我在需要展示的vue文件中:
```vue
<template>  
  <div class="hello">  
    <h1>{{ msg }}</h1>  
    <p>      你好  
    </p>  
    <h5>下面这个是远程组件</h5>  
    <RemoteComp name="loaded in codesandbox"/>  
    <h5>下面这个是我自定义的组件</h5>  
    <remoteTest/>    <h5>下面这个是我的第二个组件</h5>  
    <remoteTestApp></remoteTestApp>    <h5>上面这个是远程组件</h5>  
    <button @click="remoteClick">点击加载</button>  
    <br>    <input type="text" v-model="a">+  
    <input type="text" v-model="b">  
    ={{ c }}  
    <button @click="add">add</button>  
  </div></template>  
  
<script>  
// https://github.com/hel-eco/hel-tpl-remote-vue-comp  
import comps from "hel-tpl-remote-vue-comps"  
  
import dtlib from 'ice-test-remote'  
  
console.log(dtlib)  
export default {  
  name: "HelloWorld",  
  components: {  
    RemoteComp: comps.App,  
    remoteTest: dtlib.HelloWorld,  
    remoteTestApp: dtlib.App  
  },  
  props: {  
    msg: String,  
  },  data () {  
    return {  
      a: 0,  
      b: 0,  
      c: 0  
    }  
  },  methods: {  
    async remoteClick () {  
      const helMicro = await import('hel-micro')  
      const lib = await helMicro.preFetchLib('hel-tpl-remote-lib', { versionId: '2.0.1' })  
      console.log(lib.num.random(22))  
    },    async add () {  
      const helMicro = await import('hel-micro')  
      const { add } = await helMicro.preFetchLib('ice-test-remote', { versionId: '1.5.2' })  
      // 调用自定义方法  
      this.c = add.add(parseInt(this.a), parseInt(this.b))  
    }  },  mounted () {  
  
  }}  
</script>  
  
<!-- Add "scoped" attribute to limit CSS to this component only -->  
<style scoped>  
h3 {  
  margin: 40px 0 0;  
}  
  
ul {  
  list-style-type: none;  
  padding: 0;  
}  
  
li {  
  display: inline-block;  
  margin: 0 10px;  
}  
  
a {  
  color: #42b983;  
}  
</style>
```

这里调用`function`目前是点击时触发加载,第一次加载时会比较慢
这里的组件:`dtlib`,`comps`是本地已经安装的
这里是在使用到的组件中单独引入
![[Pasted image 20230816150504.png]]
``