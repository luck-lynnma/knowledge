# Shell教程一

## Shell变量

### Shell变量定义
+ 显示方式`param_name=value`
+ 使用赋值语句给变量赋值 **for file in `ls /etc` 或者 for file in $(ls /etc)**  
> 将目录`/etc`下文件名循环命名给file

### Shell变量使用
使用一个定义后的变量，只要在变量名前面`$`即可，`{}`加不加都行，`{}`主要用来帮助解析器识别变量边界

### 只读变量

使用`readonly`定义只读变量，只读变量的值不能被改变
` readonly param_name=value `

### 删除变量
使用`unset`命令可以删除变量
` unset variable_name `删除后不能再次被使用，** unset不能删除readonly变量 **

## Shell字符串
shell字符串可以使用`''`、也可以使用`""`、甚至可以`不用引号`
>单双引号区别：1、反引号——命令替换——将一个命令标准输出插在一个命令行中任何位置；
2、单引号、双引号——字符串界定符——单引号保持字符串原样输出；双引号字符串字面值
