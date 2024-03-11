# Arthas的一些特殊用法文档说明



#### 查看第一个参数：

```
$ watch com.taobao.container.Test test "params[0]"
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 34 ms.
@ArrayList[
    @Pojo[com.taobao.container.Test$Pojo@75a1cd57],

    @Pojo[com.taobao.container.Test$Pojo@3d012ddd],

    @Pojo[com.taobao.container.Test$Pojo@6f2b958e],

    @Pojo[com.taobao.container.Test$Pojo@1eb44e46],

    @Pojo[com.taobao.container.Test$Pojo@6504e3b2],
```

#### 查看第一个参数的size：

```
$ watch com.taobao.container.Test test "params[0].size()"
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 22 ms.
@Integer[40]
```

#### 将结果按name属性投影：

```
$ watch com.taobao.container.Test test "params[0].{ #this.name }"
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 25 ms.
@ArrayList[
    @String[name 0],

    @String[name 1],
```

#### 按条件过滤：

```
$ watch com.taobao.container.Test test "params[0].{? #this.name == null }" -x 2
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 27 ms.
@ArrayList[
    @Pojo[
        name=null,
        age=@Integer[32],
        hobby=null,
    ],
]
@ArrayList[
    @Pojo[
        name=null,
        age=@Integer[31],
        hobby=null,
    ],
]

$ watch com.taobao.container.Test test "params[0].{? #this.name != null }" -x 2
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 24 ms.
@ArrayList[
    @Pojo[
        name=@String[name 1],
        age=@Integer[3],
        hobby=null,
    ],
```

#### 过滤后统计：

```
$ watch com.taobao.container.Test test "params[0].{? #this.age > 10 }.size()" -x 2
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 29 ms.
@Integer[31]
@Integer[31]
```

#### 子表达式求值：

```
$ watch com.taobao.container.Test test "params[0].{? #this.age > 10 }.size().(#this > 20 ? #this - 10 : #this + 10)" -x 2
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 76 ms.
@Integer[21]
@Integer[21]
```

#### 选择第一个满足条件：

```
$ watch com.taobao.container.Test test "params[0].{^ #this.name != null}" -x 2
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 58 ms.
@ArrayList[
    @Pojo[
        name=@String[name 0],
        age=@Integer[2],
        hobby=null,
    ],
]

$ watch com.taobao.container.Test test "params[0].{^ #this.name != null}.size()" -x 2
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 57 ms.
@Integer[1]
```

#### 选择最后一个满足条件：

```
$ watch com.taobao.container.Test test "params[0].{$ #this.name != null}" -x 2
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 62 ms.
@ArrayList[
    @Pojo[
        name=@String[name 39],
        age=@Integer[41],
        hobby=null,
    ],
]
```

#### 访问静态变量

- 在watch命令中访问如下，但是会受到classloader的限制，不推荐使用

```
$ watch com.taobao.container.Test test "@com.taobao.container.Test@m"
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 36 ms.
@HashMap[
    @String[a]:@String[aaa],

    @String[b]:@String[bbb],
]
```

- 使用新版本中的getstatic命令，通过-c指定classloader，可以查看任意static变量，同时支持ognl表达式处理

```
$ getstatic com.alibaba.arthas.Test n 'entrySet().iterator.{? #this.key.name()=="STOP"}'
field: n
@ArrayList[
    @Node[STOP=bbb],
]
Affect(row-cnt:1) cost in 68 ms.

$ getstatic com.alibaba.arthas.Test m 'entrySet().iterator.{? #this.key=="a"}'
field: m
@ArrayList[
    @Node[a=aaa],
]
```



#### 调用静态方法

```
$ watch com.taobao.container.Test test "@java.lang.Thread@currentThread()"
```

#### 调用静态方法再调用非静态方法

```
$ watch com.taobao.container.Test test "@java.lang.Thread@currentThread().getContextClassLoader()"
```



#### 匹配线程&正则多个类多个方法

```
trace -E 'io\.netty\.channel\.nio\.NioEventLoop|io\.netty\.util\.concurrent\.SingleThreadEventExecutor'  'select|processSelectedKeys|runAllTasks' '@Thread@currentThread().getName().contains("IO-HTTP-WORKER-IOPool")&&#cost>500'
```

### 按条件过滤：

```
$ watch com.taobao.container.Test test "{params}" "params[0].{? #this.name == null }.size()>0" -x 2
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 27 ms.
@ArrayList[
    @Pojo[
        name=null,
        age=@Integer[32],
        hobby=null,
    ],
]
@ArrayList[
    @Pojo[
        name=null,
        age=@Integer[31],
        hobby=null,
    ],
]
```

实际用下来，需要添加 .size()>0，并且条件表达式和返回表达式是两个表达式。
其中条件表达式， 如果是要对字符串做比较可以写成这样：
`'params[0].{? #this.deviceKey =="KPmIDmPKMV"}.size()>0'`
即，外面用单引号，里面是双引号。