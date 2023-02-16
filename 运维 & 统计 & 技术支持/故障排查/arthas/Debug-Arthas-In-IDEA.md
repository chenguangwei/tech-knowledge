# Debug Arthas In IDEA



Tips:

1. It is better to run `as-package.sh` before start debugging, it will install the newest version.
2. If you want to debug Arthas core like `Commands`, please check the second part.

------

The first part, debug Arthas how to attach to target JVM.

## Debug `com.taobao.arthas.core.Arthas`

### Start `com.taobao.arthas.core.Arthas`

```
$ ./as.sh --debug-attach
Arthas script version: 3.0.5
Found existing java process, please choose one and hit RETURN.
* [1]: 56707 org.jetbrains.idea.maven.server.RemoteMavenServer
  [2]: 56612
  [3]: 442
  [4]: 56879 Demo
4
Calculating attach execution time...
Attaching to 56879 using version 3.0.5.20180927185309...
Listening for transport dt_socket at address: 8888
```

Actually `as.sh` start the java process:

```
java -agentlib:jdwp=transport=dt_socket,address=8888,server=y,suspend=y -Djava.awt.headless=true -Xbootclasspath/a:/Users/hengyunabc/.sdkman/candidates/java/current/lib/tools.jar -jar /Users/hengyunabc/.arthas/lib/3.0.5.20180927185309/arthas/arthas-core.jar -pid 56879 -target-ip 127.0.0.1 -telnet-port 3658 -http-port 8563 -core /Users/hengyunabc/.arthas/lib/3.0.5.20180927185309/arthas/arthas-core.jar -agent /Users/hengyunabc/.arthas/lib/3.0.5.20180927185309/arthas/arthas-agent.jar
```

The main class of `arthas-core.jar` is `com.taobao.arthas.core.Arthas`.

### IDEA remote debug

1. Edit Configurations

   [![image](https://user-images.githubusercontent.com/1683936/46815581-bc7ac980-cdad-11e8-9414-2b77d5a5cc3b.png)](https://user-images.githubusercontent.com/1683936/46815581-bc7ac980-cdad-11e8-9414-2b77d5a5cc3b.png)

2. Add new remote configuration

   [![image](https://user-images.githubusercontent.com/1683936/46815695-fc41b100-cdad-11e8-9fd8-eb3dac73b276.png)](https://user-images.githubusercontent.com/1683936/46815695-fc41b100-cdad-11e8-9fd8-eb3dac73b276.png)

3. Add breakpoint at `com.taobao.arthas.core.Arthas#main`, then start debugging

   [![image](https://user-images.githubusercontent.com/1683936/46815872-5c385780-cdae-11e8-92cf-f07a32c5ebb9.png)](https://user-images.githubusercontent.com/1683936/46815872-5c385780-cdae-11e8-92cf-f07a32c5ebb9.png)

4. 

The second part, debug arthas code running in the target JVM, this is the part that developers care about.

1. Start Demo in debug mode

   Write a simple java Demo:

   ```
   import java.util.concurrent.TimeUnit;
   import java.util.concurrent.atomic.AtomicInteger;
   public class Demo {
       static class Counter {
           private static AtomicInteger count = new AtomicInteger(0);
           public static void increment() {
               count.incrementAndGet();
           }
           public static int value() {
               return count.get();
           }
       }
   
       public static void main(String[] args) throws InterruptedException {
           while (true) {
               Counter.increment();
               System.out.println("counter: " + Counter.value());
               TimeUnit.SECONDS.sleep(1);
           }
       }
   }
   ```

   Complie and start:

   ```
   javac Demo.java
   java -Xdebug -Xrunjdwp:transport=dt_socket,server=y,address=8000 Demo
   ```

2. IDEA add new remote configuration

   [![image](https://user-images.githubusercontent.com/1683936/46817170-2648a280-cdb1-11e8-9a62-3cae545c0163.png)](https://user-images.githubusercontent.com/1683936/46817170-2648a280-cdb1-11e8-9a62-3cae545c0163.png)

3. Add breakpoint at `com.taobao.arthas.agent.AgentBootstrap#agentmain`, then start debugging

   [![image](https://user-images.githubusercontent.com/1683936/46817267-63ad3000-cdb1-11e8-888f-e4cf1b8debc1.png)](https://user-images.githubusercontent.com/1683936/46817267-63ad3000-cdb1-11e8-888f-e4cf1b8debc1.png)

4. Run `as.sh` to attach to the `Demo` jvm

   ```
   $ ./as.sh
   Arthas script version: 3.0.4
   Found existing java process, please choose one and hit RETURN.
   * [1]: 59234 Demo
   [2]: 56707 org.jetbrains.idea.maven.server.RemoteMavenServer
   [3]: 56612
   [4]: 442
   
   Calculating attach execution time...
   Attaching to 59234 using version 3.0.5.20180927185309...
   ```

5. IDEA stop at breakpoint

   [![image](https://user-images.githubusercontent.com/1683936/46817646-082f7200-cdb2-11e8-8d98-9c3932210111.png)](https://user-images.githubusercontent.com/1683936/46817646-082f7200-cdb2-11e8-8d98-9c3932210111.png)