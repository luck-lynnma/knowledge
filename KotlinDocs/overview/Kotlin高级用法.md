# Kotlin String

## split操作实践

+ split + 正则

```
//Kotlin函数:split

inline fun CharSequenece.split(regex:regex,limit:Int=0):List<String>
```

`regex`:表示一个不可变的正则表达式
`limit`:非负的值,指定要返回的子字符串的最大数量.默认为0,表示无限制

Kotlin提供了扩展函数`toRegex()`将字符串转换为正则表达式

例子:

```
val str = "Kotlination.com = Be Kotlineer - Be Simple - Be Connective"

val separate1 = str.split("=|-".toRegex())

//运行结果
[Kotlination.com , Be Kotlineer , Be Simple , Be Connective]
```

+ split + 任意字符串
上面的例子等同于

```
//" = "或" - "作为分隔符
val separate2 = str.split(" = "," - ")
```
