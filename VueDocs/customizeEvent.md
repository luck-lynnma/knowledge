# 自定义事件

## 事件名

不同于组件和prop,事件名不存在任何自动化的大小写转换,但 **HTML是大小写不敏感的,所以建议使用kebab-case的事件名**

## .sync修饰符

有些情况下需要`双向数据绑定`,可以使用`.sync`这是vue 2.3新增的修饰符.

>注意:带有`.sync`修饰符的`v-bind`不能和表达式一起使用(例如:v-bind:title.sync='doc.title+"!"'是无效的,取而代之的是,你只能提供你想要绑定的`属性名`,类似`v-model`)
