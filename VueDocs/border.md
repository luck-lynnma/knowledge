# 处理边界情况

## 尽量避免使用 $root获取vue根实例,推荐使用Vuex管理应用状态

## 避免使用 $parent属性从子组件访问父组件
因为`$parent`无法很好的扩展到更深层次的嵌套组件上,而尽量使用`依赖注入`

## 使用$refs访问子组件实例或元素

为需要访问的子组件使用`ref="name"`定义一个ID引用,然后再使用`this.$refs.name`访问子组件

>注意:`$refs`只会在组件渲染完成之后生效,且不是响应式的,仅仅是直接操作子组件的`逃生舱`-----应该避免在模板或计算属性中访问`$refs`

## 依赖注入
`provide`和`inject`
`provide`选项允许指定提供给后代的方法/数据,例如:

```js
provide(){
  return{
    reload:this.reload
  }
},
methods:{
    reload (){
       this.isRouterAlive = false;
       this.$nextTick(function(){
         this.isRouterAlive = true
       })
    }
},
```

在后代中使用`inject['getMap']`接收想要添加在该实例上的属性.

```js
inject:['reload'],
methods:{
  functionName(){
    this.reload();//直接使用,刷新当前页面
  }
},
```

这种方法,可以保证任意后代中都可以访问provide方法

>注意:依赖注入,耦合性增加,使重构变得更加困难
