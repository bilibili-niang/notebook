[文章地址](https://www.bilibili.com/read/cv25519544/)

tsx事件冒泡
```tsx
import {withModifiers} from 'vue'

const onClick = () => {
	console.log("click")
}

const Button = {
	render() {
   	return <div onClick={ withModifiers(onClick, ['stop', 'prevent']) }>text</div>
   }
}
```
```shell
function withModifiers(fn: Function, modifiers: string[]): Function

用于向事件处理函数,添加内置v-on修饰符
```