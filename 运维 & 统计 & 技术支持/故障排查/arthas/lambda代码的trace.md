代码：

```
public class Test {

    public static void main(String[] args) {
        while (true) {
            Arrays.asList(1, 2, 3).forEach(i -> {
                System.out.println("test:");
                print(i);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
    }

    public static void print(int i) {
        System.out.println(i);
    }
}
```

**如何使用trace，跟踪到lambda代码段的执行？**

调试断点到lambda里面，发现生成到类为accept:-1, 920011586 (com.yzl.test.Test$$Lambda$1)
用
`trace com.yzl.test.Test$$Lambda$1 *`
发现不行，直接使用
`trace com.yzl.test.Test *`
发现跟踪到的trace：

```
---ts=2018-10-29 11:01:17;thread_name=main;id=1;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@18b4aac2
    ---[1004.867423ms] com.yzl.test.Test:lambda$main$0()
        +---[0.159ms] java.io.PrintStream:println()
        +---[0.004751ms] java.lang.Integer:intValue()
        +---[0.044232ms] com.yzl.test.Test:print()
        ---[1004.445067ms] java.lang.Thread:sleep()
```

看到结果，猜测到：

```
trace com.yzl.test.Test lambda$main$0
```

果然正确，使用通配符*效果更好

```
trace com.yzl.test.Test *
```