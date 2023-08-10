#### 关于jeecg

##### 尝试渲染一个弹窗
```javascript
 add (that,row) {
      let value='';
      that.$confirm({
        icon: 'add',
        width: 800,
        title: '选择跳转',
        content: h => h('div', {},
            [
          h('a-radio-group',{
          	direction:'vertical'
          },[
          h('a-radio', {
                on: {
                  click: function (event) {
                    value=1
                    
                  }
                },
                attrs: {
                  value:'1'
                },
              }, `预警单`),
 					h('a-radio', {
                on: {
                  click: function (event) {
                    value=2
                    
                  }
                },
                attrs: {
                  value:'2'
                },
              }, `指令单`)
          ])]),
        onOk (close) {
        console.log('value:')
        console.log(value)
          if(value==1){
           that.$jump('/spx/event/add/instruction',{ 
            path: '/spx/event/add/instruction',
            query: {
              billType: 'YUJING',
            } 
          })
          }else if(value==2){
            that.$jump('/spx/event/add/instruction',{ 
            path: '/spx/event/add/instruction',
            query: {
              billType: 'ZHILING',
            } 
          })
          }
          close()
      
        },
        onCancel () {
        }
      })
    }
```
run:
![[Pasted image 20230605095234.png]]
##### 关于实现跳转

```javascript
that.$jump('/spx/event/add/instruction',{ 
            path: '/spx/event/add/instruction',
            query: {
              billType: 'YUJING',
            } 
          })
```

##### 单选框添加默认选择

```javascript
h('a-radio', {
                on: {
                  click: function (event) {
                    value=1
                  }
                },
                value:'1',
                attrs: {
                },
              }, `预警单`),
 					h('a-radio', {
                on: {
                  click: function (event) {
                    value=2
                  }
                },
                attrs: {
                  value:'2'
                },
              }, `指令单`)
          ])
```
第一个是默认选中的,第二个没有

#### 新增操作按钮

例如想要新增下面这个按钮:
![[Pasted image 20230609163341.png]]
在线开发>online报表配置

右键>自定义按钮
![[Pasted image 20230609163629.png]]
新增,
![[Pasted image 20230609163651.png]]
要注意这里的按钮位置,button是位置在表单头部的几个按钮,link为每一行的最后一个操作item中的选择

#### 配置报表的搜索
如图,要配这个:
![[Pasted image 20230612091122.png]]

编辑:
![[Pasted image 20230612091158.png]]
对需要查询的字段勾选
![[Pasted image 20230612091233.png]]
扩展参数这里可以设置搜索的其他条件
![[Pasted image 20230612091311.png]]
这里就是隐藏了标签

#### 关于报表中弹出框报表的配饰:

如果你在按钮的js增强中看到像是下面的代码:
```shell
that.onlineCode= code .........
```
那么可以使用下面的方法,它跳转的是子视图
例如想要修改下面这个弹窗的input为下拉框:
点击这个:
![[Pasted image 20230616164223.png]]
把下面这个改为下拉框:
![[Pasted image 20230616164251.png]]
进入online报表开发,对需要改的报表右键,
![[Pasted image 20230616164338.png]]

![[Pasted image 20230616164432.png]]
进入后可以看到他的子表单:
![[Pasted image 20230616164503.png]]
修改这里即可:
![[Pasted image 20230616164802.png]]
但要注意,配置下拉框,那么它的选项配置需要通过字典来获取,用后端给的字典填一下:
![[Pasted image 20230616164927.png]]

#### 添加一级菜单:

菜单管理>新增:
![[Pasted image 20230627164130.png]]
必填的填好:
![[Pasted image 20230627164147.png]]
系统管理>角色管理
![[Pasted image 20230627164241.png]]
更多>授权
![[Pasted image 20230627164302.png]]
默认新建的菜单是不显示的,想要显示的菜单打√
![[Pasted image 20230627164357.png]]

