# Arthas

**Arthas** 是 `Alibaba` 在 2018 年 9 月开源的 **Java 诊断**工具。支持 `JDK6+`， 采用命令行交互模式，提供 `Tab` 自动不全，可以方便的定位和诊断线上程序运行问题

* 为什么需要在线排查？
  在生产上我们经常会碰到一些不好排查的问题，例如线程安全问题，用最简单的threaddump或者heapdump不好查到问题原因。为了排查这些问题，有时我们会临时加一些日志，比如在一些关键的函数里打印出入参，然后重新打包发布，如果打了日志还是没找到问题，继续加日志，重新打包发布。对于上线流程复杂而且审核比较严的公司，从改代码到上线需要层层的流转，会大大影响问题排查的进度。 
  
  



使用案例：

- **[一图掌握Arthas—常用命令汇总](arthas/一图掌握Arthas—常用命令汇总.md)**

- [使用Arthas排查线上应用日志打满问题](arthas/使用Arthas排查线上应用日志打满问题.md)

- [引发线程cpu占用率持续飙升的根因分析](arthas/引发线程cpu占用率持续飙升的根因分析.md)

- [利用Arthas精准定位Java应用CPU负载过高问题](arthas/利用Arthas精准定位Java应用CPU负载过高问题.md)

- [工商银行打造在线诊断平台的探索与实践](arthas/工商银行打造在线诊断平台的探索与实践.md)

- [[阿里云峰会-北京]Java诊断利器Arthas排查问题实践](arthas/[阿里云峰会-北京]Java诊断利器Arthas排查问题实践.md)

- [【Arthas问题排查集】活用ognl表达式.md](arthas/[Arthas问题排查集]活用ognl表达式.md)

- [分享及其资料：当DUBBO遇上Arthas - 排查问题的实践](arthas/当DUBBO遇上Arthas排查问题的实践.md)

- [记录一次redisson Bug的排查与修复过程](arthas/记录一次redissonBug的排查与修复过程.md)

- [利用Arthas排查Spring Boot应用NoSuchMethodError](arthas/利用Arthas排查Spring-Boot应用NoSuchMethodError.md)

- [使用 SkyWalking & Arthas 优化微服务性能.md](arthas/使用SkyWalking&Arthas优化微服务性能.md)

- [使用arthas+jprofiler做复杂链路分析](arthas/使用arthas+jprofiler做复杂链路分析.md)

- [Alibaba Arthas实践--获取到Spring Context，然后为所欲为](arthas/Alibaba-Arthas实践--获取到Spring-Context，然后为所欲为.md)

- [arthas获取spring被代理的目标对象](arthas/arthas-获取spring被代理的目标对象.md)

- [Arthas ByteKit 深度解读(1)：基本原理介绍 ](arthas/ArthasByteKit深度解读(1)：基本原理介绍.md)

- [Arthas ByteKit 深度解读(2)：本地变量及参数绑定](arthas/ArthasByteKit深度解读(2)：本地变量及参数绑定.md)

- [Arthas Web-Console一站式解决方案](arthas/Arthas-Web-Console一站式解决方案.md)

- [Arthas的一些特殊用法文档说明](arthas/Arthas的一些特殊用法文档说明.md)

- [Arthas排查Kubernetes中的应用频繁挂掉重启问题](arthas/Arthas排查Kubernetes中的应用频繁挂掉重启问题.md)

- [Arthas实践--使用redefine排查应用奇怪的日志来源](arthas/Arthas实践--使用redefine排查应用奇怪的日志来源.md)

- [Arthas实践--jadmcredefine线上热更新一条龙](arthas/Arthas实践--jadmcredefine线上热更新一条龙.md)

- [Arthas实践：解决由于druid版本造成的慢sql问题](arthas/Arthas实践：解决由于druid版本造成的慢sql问题.md)

- [Arthas实践：是哪个Controller处理了请求？](arthas/Arthas实践：是哪个Controller处理了请求？.md)

- [Arthas协助排查线上skywalking不可用问题](arthas/Arthas协助排查线上skywalking不可用问题.md)

- [Debug Arthas In IDEA](arthas/Debug-Arthas-In-IDEA.md)

- [lambda代码的trace](arthas/lambda代码的trace.md)

- [Spring Boot微服务性能下降九成！使用Arthas定位根因](arthas/Spring-Boot微服务性能下降九成！使用Arthas定位根因.md)

- [SpringBoot Admin集成Arthas实践](arthas/SpringBoot-Admin集成Arthas实践.md)

  



参考资料

>arthas技术手册：https://arthas.aliyun.com/doc/

