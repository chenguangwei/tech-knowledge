## 背景

尽管在生产环境热更新代码，并不是很好的行为，很可能导致：热更不规范，同事两行泪。

但很多时候我们的确希望能热更新代码，比如：

> 线上排查问题，找到修复思路了，但应用重启之后，环境现场就变了，难以复现。怎么验证修复方案？

又比如：

> 本地开发时，发现某个开源组件有bug，希望修改验证。如果是自己编译开源组件再发布，流程非常的长，还不一定能编译成功。有没有办法快速测试？

Arthas是阿里巴巴开源的Java应用诊断利器，深受开发者喜爱。

下面介绍利用Arthas 3.1.0版本的 `jad`/`mc`/`redefine` 一条龙来热更新代码。

- Arthas: https://github.com/alibaba/arthas
- jad命令：https://alibaba.github.io/arthas/jad.html
- mc命令：https://alibaba.github.io/arthas/mc.html
- redefine命令：https://alibaba.github.io/arthas/redefine.html

## Arthas在线教程

下面通过Arthas在线教程演示热更新代码的过程。

- [Arthas进阶教程](https://alibaba.github.io/arthas/arthas-tutorials?language=cn&id=arthas-advanced)

[![arthas-online-hotswap](https://camo.githubusercontent.com/c665b1f384286bcebc6cd1b2bf693c7b4ff0718863bfd25048b6279c2ff4eb21/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031392f322f31392f313639303462346637633965613034373f773d3130333826683d35353526663d706e6726733d313939313934)](https://camo.githubusercontent.com/c665b1f384286bcebc6cd1b2bf693c7b4ff0718863bfd25048b6279c2ff4eb21/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031392f322f31392f313639303462346637633965613034373f773d3130333826683d35353526663d706e6726733d313939313934)

在例子里，访问 `curl http://localhost/user/0`，会返回500错误：

```
{
    "timestamp": 1550223186170,
    "status": 500,
    "error": "Internal Server Error",
    "exception": "java.lang.IllegalArgumentException",
    "message": "id < 1",
    "path": "/user/0"
}
```

下面通过热更新代码，修改这个逻辑。

## jad反编译代码

反编译`UserController`，保存到 `/tmp/UserController.java`文件里。

```
jad --source-only com.example.demo.arthas.user.UserController > /tmp/UserController.java
```

## 修改反编绎出来的代码

用文本编辑器修改`/tmp/UserController.java`，把抛出异常改为正常返回：

```
    @GetMapping(value={"/user/{id}"})
    public User findUserById(@PathVariable Integer id) {
        logger.info("id: {}", (Object)id);
        if (id != null && id < 1) {
            return new User(id, "name" + id);
            // throw new IllegalArgumentException("id < 1");
        }
        return new User(id.intValue(), "name" + id);
    }
```

## sc查找加载UserController的ClassLoader

```
$ sc -d *UserController | grep classLoaderHash
 classLoaderHash   1be6f5c3
```

可以发现是spring boot的 `LaunchedURLClassLoader@1be6f5c3` 加载的。

## mc内存编绎代码

保存好`/tmp/UserController.java`之后，使用mc(Memory Compiler)命令来编译，并且通过`-c`参数指定`ClassLoader`：

```
$ mc -c 1be6f5c3 /tmp/UserController.java -d /tmp
Memory compiler output:
/tmp/com/example/demo/arthas/user/UserController.class
Affect(row-cnt:1) cost in 346 ms
```

## redefine热更新代码

再使用redefine命令重新加载新编译好的`UserController.class`：

```
$ redefine /tmp/com/example/demo/arthas/user/UserController.class
redefine success, size: 1
```

## 检验热更新结果

再次访问 `curl http://localhost/user/0`，会正常返回：

```
{
    "id": 0,
    "name": "name0"
}
```

## 总结

Arthas里 `jad`/`mc`/`redefine` 一条龙来线上热更新代码，非常强大，但也很危险，需要做好权限管理。

比如，线上应用启动帐号是 admin，当用户可以切换到admin，那么

- 用户可以修改，获取到应用的任意内存值（不管是否java应用）
- 用户可以attach jvm
- attach jvm之后，利用jvm本身的api可以redefine class

所以：

- 应用的安全主要靠用户权限本身的管理
- Arthas主要是让jvm redefine更容易了。用户也可以利用其它工具达到同样的效果

最后，Arthas提醒您： **诊断千万条，规范第一条，热更不规范，同事两行泪**。

## Arthas实践系列

- [Alibaba Arthas实践--获取到Spring Context，然后为所欲为](http://hengyunabc.github.io/arthas-spring-context/)
- [Arthas实践--快速排查Spring Boot应用404/401问题](http://hengyunabc.github.io/arthas-spring-boot-404-401/)
- [当Dubbo遇上Arthas：排查问题的实践](http://hengyunabc.github.io/dubbo-meet-arthas/)
- [Arthas实践--使用redefine排查应用奇怪的日志来源](http://hengyunabc.github.io/arthas-redefine-case/)
- [使用Arthas抽丝剥茧排查线上应用日志打满问题](http://hengyunabc.github.io/arthas-logger-problem/)
- [深入Spring Boot：利用Arthas排查NoSuchMethodError](http://hengyunabc.github.io/spring-boot-arthas-NoSuchMethodError/)