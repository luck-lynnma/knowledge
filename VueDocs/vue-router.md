# Vue-router
Vue-router应用于单页面富应用(SPA),将组件映射到路由,并告知vue-router在哪里渲染他们.

## 声明式导航

使用模块化机制编程步骤:
+ 导入Vue和Vue-router
+ 调用Vue.use(Vue-router)插入router组件
+ 定义router组件,也可以使用import进来
+ 定义路由(routes)  
  每一个应该映射一个组件,其中component可以是通过Vue.extend()创建的组件构造器,也可以是组件的配置对象
+ 创建router实例,然后传入'routes'配置
+ 创建和挂载根实例  
  通过`router`配置参数引入

通过注入路由器,可以使用`this.$router`访问路由器,也可以通过`this.$route`访问`当前`路由配置参数等信息

### 动态路由匹配

需要把通过某种模式匹配到的所有路由,全部映射到同一个组件,就要用到`动态路由匹配`,在`vue-router`的路由路径中使用`动态路径参数`,动态路径参数,以`:`开头,当匹配到一个路由时,参数值会被设置到`this.$route.params`,可以在每一个组件中使用.

例如:

|模式|匹配路径|$route.params|
|:---|:---|:---|
|/user/:username|/user/even/{username:'even'}|
|/user/:username/post/:post_id/|user/even/123|{username:'even',post_id:'123'}

### 相应路由参数变化
  在使用动态路径参数时,组件就会被复用,这意味着复用过程中组件生命周期钩子不会再调用,如果想要对路由参数变化做出响应时,可以使用`watch`监控`$route`对象

例子:

```js
...
template: '...',
watch:{
    '$route'(to,from){
        to.path;
        from.path
    }
}
```

或者可以使用`BefourRouterUpdate()`API

```js
...
template: '...',
befourRouterUpdate(to,from,next){
    //router改变
}
```

### 捕获所有路由或404路由

常规参数只会匹配被`/`分割的URL中字符,如果想要匹配任意路径需要使用通配符`*`

例子:

```js
{
    path:"/user-*"
},

//通常用来匹配404页面
{
    path:"*",
},

```

> 注意:使用`*`的路由,一定要放在最后.使用通配符,参数$route.param后面会自动添加一个'pathMatch'参数,需要通过$route.params.pathMatch.XXX获取参数名称

### 嵌套路由

+ 需要在VueRouter参数中使用`children`配置
+ 在被渲染的组件中包含嵌套的`<router-view>`

>注意:VueRouter的`children`配置的`path`不能以`/`开头,因为以`/`开头的嵌套路径会被当做根路径

## 编程式的导航
以上是使用`<router-link>`创建a标签来定义导航链接,我们还可以借助router的实例方法,编写代码来实现

### router.push(location,onComplete?,onAbort)

`onComplete`和`onAbort`这两个参数是回调,会在导航成功或终止时进行相应的回调

>在Vue实例内部,可以使用`$router`访问路由实例,因此也可以调用`this.$router.push()`

想要导航到不同的URL,则使用`router.push`会向history栈添加一个新的记录,所以,当用户点击后退按钮时,则会回到之前的URL.

其实当你点击`<router-link>`时.这个方法会在内部调用,所以,点击`<router-link to="...">`等同于调用`router.push(...)`

|声明事|编程式|
|:---|:---|
|< router-link :to="...">|router.push(...)|

该方法的参数可以是一个字符串路径,或者一个描述地址的对象,例如:

```js
//字符串
router.push('home')
//对象
router.push({path:'home'})
//命名路由
router.push({name:'user',params:{userId:'123'}})
//带查询参数,变成:/register?plan=private
router.push({path:'register',query:{plan:'private'}})
```

>注意:如果提供了`path`,`params`就会被忽略,上述例子中`query`并不属于这种情况.你可以提供路由name,或是手写完整带参数的`path`

例如:

```js

const userId = '123'
router.push({name:'user',params:{userId}}) //-> /user/123

router.push({path:'/user/${userId}'}) //-> /user/123
router.push({path:'/user',params:{userId}}) //-> /user(这里的params不生效)

```

同样规则也适用于`<router-link>`

>注意:如果目的地和当前路由相同,只有参数发生了改变(路由动态路径参数变化),需要调用`befourRouterUpdate()`

### router.replace(location,onComplete?,onAbort?)

跟`router.push`很像,唯一的区别就是,他不会像history添加新纪录,而是替换掉当前的history记录

### router.go(n)

n是一个整数,意思是在history记录中向前或者向后退多少步,类似`window.history.go(n)`

```js
router.go(1)//在浏览器记录中前进一步,等同于history.forward()
router.go(-1) //后退一步,相当于history.back()
```

## Router 构建选项

### routers

+ 类型:`Array<RouteConfig>`

`RouteConfig`定义:

```js
declare type RouteConfig = {
  path:string,
  component?:Component,
  name?:string , //命名路由
  components?:{[name:string]:Component} , //命名视图组件
  redirect?:string | Location | Function , //重定向
  props?:boolean | Object | Function,
  alias?:string | Array<string> ,   //别名
  children?:Array<RouteConfig>,  //嵌套路由
  beforeEnter?:(to:Route,from:Route,next:Function)=>ovid   //路由钩子
  meta?:any,    //元信息
}
```

### mode

+ 类型:`String`,
+ 默认值:`hash(浏览器环境)`|`abstract(nodejs环境)`
+ 可选值:`hash  |  history  |  abstract`

#### mode模式下打包配置路径
router默认mode为`hash`模式,也可以改为`history`模式
两种模式打包路径配置方式不同.

+ hash模式下,npm run build时,`assetsPublicPath:'/'`就要修改如下方式

```js
//config/index.js中
build: {
  ...
    // 静态资源部署后存放的公共路径
    assetsPublicPath: './',
}
```

否则找不到static下面的文件

+ history模式下,npm run build打包能成功找到静态资源,`需要后台服务配置作为支撑`,需要在Router构建选项中添加`base:'xxx'`配置,然后修改后,`assetsPublicPath='/'`

```js
export default new Router({
  mode: 'history', // 后端支持可开
  scrollBehavior: () => ({ y: 0 }),
  base: '/gateway', // 打包项目根目录
  routes: constantRouterMap
})

//config/index.js中
build: {
  ...
  // 静态资源部署后存放的公共路径
  assetsPublicPath: '/gateway/',
}
```

这种方式,打包后才会顺利找到静态资源.

#### 两种mode下发布

+ hash模式下,发布到服务器之后,因为router的url改变后,页面不会重新加载,所以不会出现刷新页面出现404

+ history模式下,发布到服务器上之后,刷新页面就会出现404,解决方式:

  + tomcat部署下,在tomcat的wabapps目录下,新建`gateway`目录,将打包的静态资源全部copy到gateway下
  + 新建`WEB-INF`目录,在此目录下新建`web.xml`文件,文件中添加如下内容即可解决问题:

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                        http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
      version="3.1" metadata-complete="true">
      <display-name>Router for Tomcat</display-name>
      <error-page>
          <error-code>404</error-code>
          <location>/index.html</location>
      </error-page>
  </web-app>
  ```

+ 或者使用nginx配置里添加vue-router跳转设置

```js
server {
         listen 80;

         server_name testwx.wangshibo.com;
         root /Data/app/xqsj_wx/dist;
         index index.html;
         access_log /var/log/testwx.log main;
          // 这里 开始
         location / {
             try_files $uri $uri/ @router;
             index index.html;
         }
        location @router {
            rewrite ^.*$ /index.html last;
        }
        //到这里结束
}
```

### base

+ 类型:`String`
+ 默认值:`/`

应用的基路径.例如:如果整个单页应用服务在`/app/`下,base就要设置为`/app/`


## Router钩子

vue-router中导航钩子按照定义位置不同(执行时机也不同)分为`全局钩子`,`路由钩子`,`组件钩子`.

+ 全局钩子:`router.beforeEach`,`router.beforeResolve`,`router.afterEach`
+ 路由钩子:`beforeEnter`,在路由配置项中定义
+ 组件级钩子:`beforeRouterEnter`,`beforeRouteUpdate`和`beforeRouteLeave`,在组件中定义

执行时机:
+ 首页进入routers配置项中页面:

  global beforeEach -> router beforeEnter -> component beforeRouteEnter -> global afterEach

+ routers配置项进去首页时:

  component beforeRouteLeave -> global beforeEach -> global beforeResolve -> global afterEach

+ beforeRouteUpdate是在 组件被复用时调用的,例如动态参数的路径`foo/:id`,在foo/1和foo/2页面之间跳转时,渲染的是同一个组件,因此组件实例会被复用


### 全局守卫

**"导航"** 表示路由正在发生改变.路由的`导航守卫`主要用来通过跳转或取消的方式守卫导航.

**全局的,单个路由独享的,或者组件级的**

`参数或查询的改变并不会触发进入/离开的导航守卫`,可以通过`监控$route`来实现或者使用`beforeRouteUpdate`的 **组件内守卫** 来实现

#### router.beforeEach

可以使用`router.beforeEach`注册一个`全局的前置守卫`.当一个导航触发时,全局前置守卫按照创建顺序调用.守卫是异步解析执行,此时所在导航的所有守卫resolve之前全部处于等待状态.

每个守卫接收三个参数:

+ `to:Route`:即将要进入的目标路由对象
+ `from:Route`:当前导航正要离开的路由
+ `next:Function`:一定要调用该方法来resolve这个钩子

**确保调用`next`方法,否则钩子就不会被resolved**

#### router.beforeResolve
可以使用`router.beforeResolve`注册一个全局解释守卫

所有组件内守卫和异步路由组件被解析后,router.beforeResolve才会被调用

#### router.afterEach

可以注册全局后置钩子,和守卫不同的是,这些钩子不会接收`next函数`,也不会改变导航本身.

### 路由独享守卫

可以在路由配置中直接定义`beforeEnter`守卫

```js
const router = new VueRouter({
  routes:[{
    path:'/foo',
    component:Foo,
    beforeEnter:(to,from,next)=>{

    }
  }]
})
```

### 组件内守卫

```js
const Foo = {
  template:'...',
  beforeRouterEnter(to,from,next){
    //在渲染该组件对应路由被confirm前被调用
    //不!能!获取组件实例`this`
    //因为当守卫执行前,组件实例还没被创建
    //但是支持向next传递一个回调来访问组件实例,在导航被确认的时候执行回调
    next(vm=>{
      //通过`vm`访问组件实例
    })
  },
  beforeRouterUpdate(to,from,next){
    //在当前路由改变,但是该组件复用时调用
    //路由动态参数路径时会用到
    //可以访问组件实例`this`
  },
  beforeRouteLeave(to,from,next){
    //导航离开该组件对应路由时调用
    //可以访问组件实例`this`
  }
}
```
