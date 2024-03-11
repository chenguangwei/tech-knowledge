# Arthas Web-Console一站式解决方案



### 1.1编写目的

该文档主要是解决远程使用arthas诊断的功能需求,只需要一个Web-Console即可使用Arthas的功能,解决机器权限问题,控制必要的命令和禁止不必要的命令,也为公司的机器安全着想也解决研发同学利用好工具Arthas诊断他负责的系统潜在的威胁.

GitHub上官方命令文档的连接: https://alibaba.github.io/arthas/web-console.html

本文主要解决一站式使用Arthas的方案,该方案并非完美方案,但可用于生产级别,各位如果有新的更好的想法,可直接扩展实现属于自己公司的方案.

看此文章之前必须通过控制台部署和Client端注册到Server端成功才能进行阅读,否则你根本不知道此文对你有什么意义.

### 1.2项目背景

统一授权登录机器,对机器可以直接使用arthas诊断,无需登录到具体目标IP去启动arthas-boot.jar.
**2. 准备环境**
**2.1统一的web-console服务端**

下载arthas-tunnel-server-3.1.7.jar,我这里用的是3.1.7版本,请到官网直接下载即可: https://github.com/alibaba/arthas/releases

**2.2在目标机器上上传arthas包**

下载arthas-3.1.7-bin.zip,我这里用的是3.1.7版本, 请到官网直接下载即可: https://github.com/alibaba/arthas/releases

解压到你自己的目录下.

### **2. web-console****服务端使用说明** **2.1****部署web-console服务端**

启动arthas-tunnel-server-3.1.7.jar

这个在前一篇文件已经给出详细的部署方案了,各位参考之前的文章即可:https://blog.csdn.net/caodegao/article/details/104016095

### 3.需要解决的问题

前提:下载arthas的源码,里面已经包含了server的控制台代码了(tunnel-server).

把包复制一份为(arthas-tunnel-server),单独对他修改,不影响到远代码.也方便以后合并新分支做对比改动的部分合并.

![img](https://img-blog.csdnimg.cn/20200608162913505.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nhb2RlZ2Fv,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20200608162955240.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nhb2RlZ2Fv,size_16,color_FFFFFF,t_70)

**3.1如何用SSH连接到目标机器**

Linux机器直接用Java来打交道要用到SSH工具,个人使用的是ganymed的api来和Linux交互,目的是执行JPS命令拿到该机器的PID进程,在用java -jar arthas-boot.jar pid命令直接attach到进行并启动arthas工具.

我还用到httpclient,和apache文件读起的包,http大家可以用Spring的RestTemplate,这样就不用导入这些包了.

http是用到了获取用户有权限的机器列表和ip列表时使用的,因为公司提供了http的接口可以获取到这些东西.

如果其他的朋友有用到就可以用这种方式去拿,或者有些是通过dubbo接口提供的,那就用dubbo,我只是提一嘴.

![img](https://img-blog.csdnimg.cn/20200608163425712.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nhb2RlZ2Fv,size_16,color_FFFFFF,t_70)

写一个SSH交互的代码例子.红色框是用SSH生成的id_rsa文件,怎么生成大家去百度即可,我生成后就是这个样,记得是带BEGIN和END的这种杆杆.这个文件要放到目标机器的hosts文件里面做免密登录的,这点要跟大家说明白,否则Ssh登录不了命令无法执行.

特别注意一下

![img](https://img-blog.csdnimg.cn/20200608172120148.png)

然后用io加载为file,这个大家找例子都能有,用FileUtils挺方便的,所以我就导入了common-io.

![img](https://img-blog.csdnimg.cn/20200608171943500.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nhb2RlZ2Fv,size_16,color_FFFFFF,t_70)

所以第一步就先执行jps命令,把该目标IP所有的PID给展现出来,让用户选择要attach哪个

命令:String cmd = "source /etc/profile; jps -l"; 为啥要加source,因为用这个工具貌似必须要执行一下环境变量才能执行到这个命令,我也不知道为什么,暂时放下后面再研究,反正我直接jps -l是报没有这个命令的提示.

可以看看我改造后的页面

1:根据登录人,下拉列表中显示有权限的系统;

2:该系统对应的IP列表,选择IP,或者通过自定义IP直接填入IP;

3:显示进程,把IP的进程显示出来,用户选择要Attach的进程即可.

![img](https://img-blog.csdnimg.cn/20200609110159208.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nhb2RlZ2Fv,size_16,color_FFFFFF,t_70)

这些可以参考自己去做改造.

**3.2如何执行java -jar arthas-boot.jar**

点击上图的Attach按钮后,执行的命令是:source /etc/profile; java -jar %s %s --tunnel-server 'ws://%s:7777/ws' --agent-id '%s' --session-timeout 300 --attach-only

%s是String.format的转义字符串:

1:arthas-boot.jar的完整路径.

2:pid就是上图的7.

3:tunnel-server的ip我是在代码里面获取执行机的IP填进去的, 为什么呢,因为tunnel-server是多机器部署的,如果你用一个域名/LVS那么你让目标机器注册到的可能不是你现在这台执行远程命令的IP,那么你通过当前连接就会出现问题了,目标机器Attach到其他节点上去了,这里有点尴尬.

4:目标Ip也是页面传过来的目标的IP,上个文章说了,要用IP作为agent-id就是有这个作用,避免重复,也好标记意义.

重要提示:执行该命令必须执行退出该目标IP的命令,所以我在页面点击Attach按钮是做了两个动作,先Stop再启动arthas-boot,Stop命令很重要,我也找了很久才发现arthas-client.jar有这个命令支持,相当于踢出前个登录只允许一个地方登录.
String cmd = "source /etc/profile; java -jar 你的路径/arthas-client.jar -c stop";

**3.3如何使用xterm.js黑框框与目标机器交互**

上图的页面是通过Jquery和后台交互的,我在里面做了一些改造,

1:那个下拉选择菜单我用Bootstrap来做的,哎!我不懂前端,搞起来很麻烦,我有篇文章也写了怎么去用这个下拉框,这里就不再说了.

2:Jquery我换过,原本是最简单版的没有ajax,所以我就换了3.4.1版本进来.

3:my-web-console.js我自己写的ajax代码,这里也不方便直接展示给大家看,见谅,不过这也难不了大家,网上一堆例子.

![img](https://img-blog.csdnimg.cn/20200608175007861.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nhb2RlZ2Fv,size_16,color_FFFFFF,t_70)

重要的是黑框框的输入跟远程机器交互这里,我要稍稍给大家说一下,通过上面的命令我已经attach到远程机器了,我现在就出现这种效果:

![img](https://img-blog.csdnimg.cn/20200608175651872.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nhb2RlZ2Fv,size_16,color_FFFFFF,t_70)

这个是修改index.html页面后变成这样子的,我已经通过xterm.js设置不允许黑框输入,因为如果开放这个等于什么命令都能执行,权限太大,所以还是禁用掉,只能使用你控制的命令,我在下面新增了输入框,通过这个来控制.

![img](https://img-blog.csdnimg.cn/20200608180010194.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nhb2RlZ2Fv,size_16,color_FFFFFF,t_70)

那么现在问题来了,通过js来获取输入命令,该怎么和xterm交互呢,我也是摸索了很久才发现,根据源码是有这一句:ws.send(JSON.stringify({action: 'read', data: cmd}));

其实很简单,复制这句话,再执行命令的时候再cmd后面加 \n即可,等于换行执行.也挺逗的.

ws.send(JSON.stringify({action: 'read', data: cmd + '\n'}));

还有个问题:比如执行dashboard命令的时候是不停的刷的,怎么让他停下来呢?这个命令怎么用js来表达:Control+C

我找了好久是这个:\x03代表.

![img](https://img-blog.csdnimg.cn/20200608180610765.png)

**3.4进一步扩展根据登录人直接显示有权限的机器以及机器IP列表**

上面的下拉列表是根据http接口去获取的,这点小小说明一下,因为官方的这个前端控件没有这种下拉支持,所以我找了boostrap4来做的:https://blog.csdn.net/caodegao/article/details/104928117

就是风格有点相似所以我用它,但是我用这种东西导致按钮颜色变得有点奇怪了,我不知道怎么解决,希望后面有其他的代替吧.

**3.5对Web-Console页面进行修改,集成几个基础快捷键方便用户直接使用**

接下来看看我实现的快捷键功能:

我把常用的很多功能直接做成按钮在下面,并且支持快捷键,比如Dashboard直接按d就执行了.如果有其他命令不在里面的可以用input输入框执行,我会屏蔽校验redefine这种命令,因为这是修改内存代码的命令,有点危险.这个由大家去扩展.

![img](https://img-blog.csdnimg.cn/20200608180924760.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nhb2RlZ2Fv,size_16,color_FFFFFF,t_70)![img](https://img-blog.csdnimg.cn/20200608180928859.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nhb2RlZ2Fv,size_16,color_FFFFFF,t_70)

提示:每执行一行代码记得先执行Control+C退出,因为Dashboard这种你必须先退出才能执行下一个命令.

快捷键监听的方法给大家贴出来,如果要找确实得找好久.最后那个是Ctrl+C的组合键,送给各位了.

![img](https://img-blog.csdnimg.cn/20200608181830647.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nhb2RlZ2Fv,size_16,color_FFFFFF,t_70)

**3.6火焰图支持**

火焰图是个好东西,大家要多推广这个功能,但是也对这个目标机器有要求的,要开通机器诊断权限,否则执行命令的时候会显示一些不支持的返回,这点要在目标机器先测试一下看看,一般不是arthas的问题都是机器不给权限或者不装某些插件的问题,如果是公司的策略原因,那么得由公司管理定夺.

执行火焰图生成后,执行停止收集命令会有一个文件路径,大家可以执行停止的时候加上路径参数,保存到一个地方,让这个地方有权限浏览或下载,可以用一个挂载盘,如果没有挂载盘,那么先存到本地,用SSH命令下载出来.

我就是用SSH命令下载的,但是前提是机器要装SCP命令,否则无法使用的,代码很简单,参考这个即可.记得目标机器必须要装SCP命令,否则下载是异常的.

![img](https://img-blog.csdnimg.cn/2020060818285952.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nhb2RlZ2Fv,size_16,color_FFFFFF,t_70)

stop采样的代码,要设置自己的路径,然后文件名,不设置文件名会是一个随机串,这点你不好操作,所以我用年月日时间来做文件名,停止后直接填到input框里面,再点击下载火焰图出来.

![img](https://img-blog.csdnimg.cn/2020060818311564.png)

![img](https://img-blog.csdnimg.cn/20200608183315519.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nhb2RlZ2Fv,size_16,color_FFFFFF,t_70)

### 最后

![img](https://img-blog.csdnimg.cn/20200609104625140.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nhb2RlZ2Fv,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20200609110305487.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nhb2RlZ2Fv,size_16,color_FFFFFF,t_70)![img](https://img-blog.csdnimg.cn/2020060910561663.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nhb2RlZ2Fv,size_16,color_FFFFFF,t_70)

**总结一下:**

1:SSH登录认证要用ssh的根文件

2:SSH交互有java的jar api支持可以直接用,记得要source一下,执行arthas-boot.jar之前要先stop.

3:前端体验好要把登录人的权限给控制好,直接显示有权限列表的IP即可,其实我不建议填IP的,大家可以不做这个功能,这是测试方便用的,对了这个控制台没有登录的,我自己做了一个链接跳转,执行域名的时候默认会执行一个方法去获取登录人的应用权限列表,这个控制台也是从我们DevOps平台链接过来的,所以有登录人信息,这点大家自行实现.

4:前端xterm.js交互很简单,记得加换行命令.

5:优化体验控制权限,定义一些常用的命令出来,记得屏蔽掉一些危险的命令.

6:火焰图的好处要多用用,这是你推广这个技术的好切入点.

虽然写的粗糙些,但是很多关键点和逻辑已经给出来了,秉着开源的原则,我尽量做到回馈社区,感谢alibaba,我也是从里面出来的,吃水不忘挖井人,如果大家有哪些疑问的,需要探讨的请留言,我尽量回复,有时候忙不看消息也没办法.

有些点我没贴出代码的,大家也别上火,只要你用心开发过了找过资源了测试过了,你会很熟悉,如果全用我的代码你会失去乐趣.里面的index.html我改造了很多,大家凭自己的前端能力去改造即可,可以邀请你们公司的前端给你搞更好,我们组没有前端资源,只能自己整,很多JS自己摸索资料写的,能用即可,后期再优化.

谢谢各位!