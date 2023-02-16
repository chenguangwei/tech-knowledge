## Arthas实践：解决由于druid版本造成的慢sql问题



### 故事背景

> 最近在一个技术交流群里遇到一个小兄弟请求问题，看着平淡无奇的问题，我建议查看mysql-server连接数是否正常，以及当前是否有活跃的慢连接在操作数据库。经过几次确认之后发现并没有什么异常。最后使用阿里开源的神器arthas。网址https://arthas.aliyun.com/doc/ 在我的指导下，小兄弟查看了占用cpu最高的http线程，并通过arthas的trace命令分析到druid的一处代码特别耗时。基本问题已经定位到了，但是怎么解决问题，不仅仅需要技巧还需要点运气，我建议他去github的druid仓库issue下去搜索有没有类似问题。在输入了关键字”loadClass”之后，果真有人遇到了类似问题，也贴出了解决方案。最终在我的指导下，小兄弟问题得到了解决。作为交换，小兄弟答应我给我写一遍完整的故障流程。于是，故事开始…..以下内容来自这位网友的投稿

### 项目背景

> 随着业务的扩展，组内的项目越来越多。经常是一个Tomcat里放好将近15个war包。随之而来的是Tomcat严重的宕机事故。痛定思痛之后决定采用Nginx代理解决问题。在部署方案上从原来的Ha代理转变为Ha代理到Nginx，再由Nginx代理到各个服务。其中将业务相关的应用全部独立部署，基础服务暂时保留在Tomcat中。决定了部署方案之后，在灰度环境做了各种测试。均没有发现问题。各种服务独立部署并确定无误之后，将Ha的代理切到Nginx上。随之而来的是部分Java应用接口响应缓慢。接口响应时间平均1.5秒。
> 部署方案更换的变化点
> 1.从原来的Tomcat单个端口转变为了多个端口
> 2.Ha代理从原来直接代理到6台服务器到目前代理到nginx，nginx再做路由转发。
> 注意：Ha代理是通过Cookie路由，通过Cookie的标志让登录的用户，在登录期间能够始终访问到一台机器。新的部署方案直接抛弃了这种方式，接口路由全部采用权重。
> 期待
> 部署方案的改变是为了让服务更稳定。系统的业务部分都是单节点部署，Nginx采用主备。相比之前Tomcat容器中的多线程，目前的多进程应该具有更好的稳定性和输出能力。
> 问题的发现和解决
>
> 1、网络原因？
> 在业务系统全部切换到Nginx之后，产品经理反映系统缓慢，部分Java系统的业务无法正常进行。
> 分析：可能是网络问题。
> 原因：通过F12查看页面的接口和静态资源文件的传输记录。发现很多Js的响应达到了1秒，甚至有的Js脚本的传输花了10秒。但还是比之前慢了很多，但是每晚7点以后网络总是反应较缓慢。对产品经理解释一波之后，已经晚上9点30，大家各回各家。
> 复盘：真的是网络问题吗？如果是网络问题，那么同机房的隔壁部门的服务为啥那么快？但是具体不知道是不是不同产品的是否部署在不同机房，但肯定这种概率比较低。虽然咋不是专业搞运维的。
> 当然如果第二天应用响应正常的话确实能够说明是机房相关的网络问题。至于为啥是晚上7点以后，我的解释是该机房所在的网段或存在大型游戏服务器，毕竟7点以后该下班的都下班了。下班之后还能干啥。但是平心而论这种概率确实很低。
>
> 二、JVM原因？
> 产品经理说第二天发现系统的响应确实快了。但是部分应用的响应时间还是需要1.3秒以上。考虑到部署方案的变化确实没有入侵业务。但是究竟这1秒花到了哪里？至少目前已经可以排除了网络的问题。但是对于接口整体的慢了一秒的这种问题，确实让人很难。有什么能够整体性的影响整个代码的运行？答：代码运行的环境JVM！
> 我立马上到服务器上查看项目的Jvm情况。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/0ibN2EzcLApNVB63rb9Aa5w5q4PaHZt7HwHw8fGibfL46vzAPrJHcVxtupUZOew3pYWhyClNnNRr7LGiavROYkDRw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> 发现Jvm很平缓，并没有发生剧烈垃圾回收事件。所以暂时排除Jvm的问题。
> 为了更好的定位问题，编写python脚本发送接口，每0.5秒发一次接口，查看响应时间。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/0ibN2EzcLApNVB63rb9Aa5w5q4PaHZt7HEsJeYfQqgXFv8ibRIxh9aD5CiavI15MaprqNdJAq1VSicHsNRSBibJib20Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/0ibN2EzcLApNVB63rb9Aa5w5q4PaHZt7Hv7mIf5blC1RzxIpuLbeJhQ3UXYMUQMvaMlXcZBUmTicHmCvaEoMS5Rw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> 同时在代码中打印日志，日志的打印结果显示，单个线程第一次访问数据库消耗了1秒。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/0ibN2EzcLApNVB63rb9Aa5w5q4PaHZt7HKnfwGKg9YibLHOibiaeAepw103UmBY5LSO1z4cIhFx6yepADHJEFRkIjQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> 反思：单个线程访问接口，第一次查库会消耗掉1秒，之后的查库操作并没有时间问题。多次测试之后发现每次请求均有1秒的时间浪费。那么单个Sql的脚本问题应该不大。毕竟更换部署方案之前都没问题。走投无路的还是决定试试，把Sql放到MySql的客户端执行，发现执行的速度飞快。这就奇怪了，怀着疑问，我没有一点思绪。但还是下班要紧，我还是早早撤了。临近下班，在各种群里开始提问。希望能够得到一点启发。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/0ibN2EzcLApNVB63rb9Aa5w5q4PaHZt7H9TVK7vpz9gd6NciaaYNlb0dA0WBjfIfmyskYhWIsy2KJibibHNoE8kQvQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> 三、Druid连接池？
> 通过网友的提示，我觉得应该和Druid的连接池有关系。可能是连接池被回收时间的问题。毕竟只是第一次查库慢，之后的还好。是否是连接别回收了？如果是的话，那么如果用户访问量大的话，系统应该是变快。
> 为了验证这个想法，我早早来到公司，将Nginx的负载全部切到3台机器上，发现接口响应快了很多。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/0ibN2EzcLApNVB63rb9Aa5w5q4PaHZt7HzvtjMJkFZibqU8qSIaaMKCYGJtHVuRJ6TxY0VdZXNk8ucC1xNMsKS7g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> 所以想着项目之前在Tomcat容器中也应该有这个问题，只不过之前采用的是Ha通过Cookie唯一定位到一台机器上，所以用户的访问只是第一次慢，之后的并不慢。而现在没有这种策略，导致接口在各个机器上发送，所以Druid连接无法有效复用。所以每次都要重新获取或者其他什么原因。
> 通过频繁的刷新页面，访问接口。发现Python脚本的访问时间确实提升了很多。但这个其他原因是什么，目前还不得而知。
> 查看Durid的配置，发现也没有问题。连接回收时间也有30秒，所以也排除配置问题。
>
> ```
> ​```spring.datasource.druid.initial-size=5spring.datasource.druid.min-idle=5spring.datasource.druid.max-active=20spring.datasource.druid.max-wait=500spring.datasource.druid.min-evictable-idle-time-millis=300000spring.datasource.druid.max-pool-prepared-statement-per-connection-size=20spring.datasource.druid.keep-alive=truespring.datasource.type=com.alibaba.druid.pool.DruidDataSource```
> ```
>
> 万般无奈至极，通过微信群认识的朋友建议我用arthas跟踪一下。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/0ibN2EzcLApNVB63rb9Aa5w5q4PaHZt7HUooiajhNacJLA5myQAAiajxoA8huC5PzuxoLXFmvuht2KhxEqmLAsibRQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> 在服务器上安装并启动arthas。选择要监控的项目。通过dashboard查看项目的整体情况。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/0ibN2EzcLApNVB63rb9Aa5w5q4PaHZt7H2lYeXgv0hA6knkODWUxyDZic4kibNiaZ5gXCBlLaImZiavejB7EC3wDjQw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> 发现堆没有什么异常，但有一个CPU 100%的tomcat线程。

> 执行thread -n 1 查看当前最忙的一个线程

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/0ibN2EzcLApNVB63rb9Aa5w5q4PaHZt7H0hddgS9SLAu43QaoYrXhON6nV6EmRDeUpb2fxlLHSq5S8xKAmXCNcw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> 发现当前线程的时间消耗花费在druid获取连接的代码上。使用trace指令查看调用链[trace 类路径 类方法名]，发现getLastPacketReceivedTimeMs消耗的时间最大。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/0ibN2EzcLApNVB63rb9Aa5w5q4PaHZt7HZicB4ia0aK6vU21buXaf4yWUUia1Rc9b5kKliaNLJVrHFcXFMC4oAucARw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> 继续跟踪这里的getLastPacketReceivedTimeMs方法。发现loadClass方法花费了大量的时间。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/0ibN2EzcLApNVB63rb9Aa5w5q4PaHZt7HafZYo0l8ovKWVpBaE4WO5H0Dse3L1JstDrnhDSqkWg6hLoYaXziafzw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> > 在Idea中找到Druid这MysqlUtils中的loadClass()方法;

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/0ibN2EzcLApNVB63rb9Aa5w5q4PaHZt7HTUa72CK8nQUQ4bdgEic7oyn4qOqsnGzQldvwgceVesev7dyXIeyCBrQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> 发现Durid 1.1.10在加载com.mysql.jdbc.MysqlIO的时候很慢。通过Github查找相关问题。https://github.com/alibaba/druid/issues/3808;感觉已经找到了问题的本质。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/0ibN2EzcLApNVB63rb9Aa5w5q4PaHZt7H6W9GbGxZ1NibwN1FFFVxmtsyunLcQ6xjUBgkwGnARSKePP0NZotXKqw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> 项目的Mysql驱动用的8.0.11版本。而自己使用的Durid是1.1.10，而1.1.10版本的Druid加载的MySql是com.mysql.jdbc.MysqlIO。
>
> ```
>     <dependency>        <groupId>mysql</groupId>        <artifactId>mysql-connector-java</artifactId>        <version>8.0.11</version>    </dependency>
> ```
>
> 看到这里立马将项目的Druid换成1.1.23。然后查看源代码，发现1.1.23是支持8.0+的Mysql驱动的。分析到这里豁然开朗。
>
> ```
> <dependency>    <groupId>com.alibaba</groupId>    <artifactId>druid-spring-boot-starter</artifactId>    <version>1.1.23</version></dependency>
> ```
>
> 修改好Druid的版本之后，将程序发布到线上进行测试。发现接口响应速度基本都在100ms以内。心中悬挂Bug终于解决了，心中一整狂喜。
>
> 总结
> 通过查看源代码，发现问题的原因是Druid和MySql版本的不匹配导致的。核心原因是Druid在不同版本中将要加载的MySql驱动类被写死了。所以我们在使用Druid的时候一定要注意选择与项目的MySql驱动兼容的版本。最简单靠谱的做法就是到MysqlUtils类中查看Utils.loadClass()方法。
> 除此之外，这也是我第一次遇到这么复杂的问题。也是第一次在线上学习arthas。在这个定位问题的过程中感慨自己分析问题总是基于之前的总是正确基础上而走了很多弯路。其实在学习确实需要一些怀疑精神。否则只能通过题海战术来认识世界了。



> 来源:https://mp.weixin.qq.com/s/7SQxy0hSm_urJY05QyIwMg