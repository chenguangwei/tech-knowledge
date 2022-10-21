# 在 java 中使用 ffmpeg 的四个阶段



### 前言

只要一个开发者需要稍微深入一点处理音视频，都免不了要接触 ffmpeg，它能够很方便的实现音视频的转码、解码，剪辑，合并，分割等。

ffmpeg 本身是一个用 c 实现的 sdk 库，默认带了编译出了可执行的文件，能够通过参数去实现这些功能。

于是在 java 中有两种方式去调用 ffmpeg，一种是直接通过命令行调用，一种就是通过调用 C API。

我是从命令行调用开始使用，在经过两年多时间，断断续续的优化使用方法之后，开始正式使用 C API。

在这期间，我经历了命令行到 c api 的四个阶段。下面总结一下，在这其中每个阶段的心得体会。  

## 阶段一：字符串阶段

对于一个 java 新手来说，刚刚得知 ffmpeg 的命令行使用方法，无疑是兴奋的开始去找 ffmpeg 命令的使用方法，然后编译。

于是就能有了下面这样的代码：  

        String cmd = "ffmpeg -i xxx.wav -ar 8000 xxx_8000.wav";
        try {
            Process process = new ProcessBuilder(cmd).start();
            process.waitFor();
        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
        }

这一阶段对我来说没有持续多久，我也知道这样直接拼接字符串的效率极低，于是就有了下面的阶段。

## 阶段二：封装阶段

由于自己明白拼接字符串的效率不高，所以从 github 上找到一个对 ffmpeg 命令进行简单封装的项目 [ffmpeg-cli-wrapper](https://link.segmentfault.com/?enc=Wu%2FnS5mgk1gPJk3zm9gPIg%3D%3D.tzI6Kb2I%2BkVX0gxvWB0dqYyMB7lATWTVHjLarhJrvuAIUEonndu82NAd%2BGLANf7Z)，由于自己需要的功能比较少，这个项目的封装的东西有点多，我只需要几个类就可以。

于是自己拿了下面几个类自己改了改。  

- FFmpeg：对命令的封装，最后组装成一段命令执行
- FFmpegBuilder：对参数的封装，用于添加参数
- FFmpegJob：对进程的封装，提供的任务状态的获取
- FFmpegExecutor：组装上面三个类

在这个阶段，自己学习到了有些事物的用法，不能直接暴露出来，需要经过一定的封装和控制。

这个阶段停留的时间比较久，其中也出了不少问题，比较有代表性的问题就是，ffmpeg 进程假死的问题，执行进程不知道为什么不结束，导致系统不再处理任务。  
于是乎加了进程执行时间的监控，说起来其实就是一行代码`TimeUnit#timedJoin(Thread, timeout)`；进去看里面的源码也很简单，就是这行代码`Thread#join(millis, nanos)`。

现在回过头看这个阶段的代码，简直是惨不忍睹。  

## 阶段三：工具类阶段

经过了一段时间的磨练，自己渐渐知道了，在 java 的世界里，很多东西实际上都不再需要自己去手动实现了，apache 的工具类家族，以及 github 上茫茫多的项目，都提供了海量的工具类。就像大家都熟知的使用 json 不需要自己去拼字符串，各种各样的工具类，能够帮你做这件事。

因为偶然间看到了一个同事在我搭建的 ffmpeg 转写架构中使用 `apache-exec` 这个工具类，自己来了兴趣，突然发现自己做的很多事情，这个工具类都已经完善的做好了。

于是就出现了下面这段代码，超时异常，失败异常，失败信息获取，甚至是命令执行的工作目录，都已经考虑到了。apache 家族的工具类，源码质量高，注释充足，考虑问题全面周道，非常值得去阅读。使用方法非常基本扎实，作为入门的工具类，可以说是最优秀的了。  

    public void run(List<String> args, long timeOut, File workDir) {
        CommandLine commandline = commandline(args);
        try (ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
             ByteArrayOutputStream errorStream = new ByteArrayOutputStream()) {
            DefaultExecutor exec = new DefaultExecutor();
            exec.setExitValues(null);
            ExecuteWatchdog watchdog = new ExecuteWatchdog(timeOut);
            exec.setWatchdog(watchdog);
            if (Objects.nonNull(workDir)) {
                exec.setWorkingDirectory(workDir);
            }
            PumpStreamHandler streamHandler = new PumpStreamHandler(outputStream, errorStream);
            exec.setStreamHandler(streamHandler);
            int ret = exec.execute(commandline);
            if (ret == 143) {
                throw new TimeoutException(String.format("Timed out waiting for to finish. Wait %d ms", timeOut));
            } else if (ret != 0) {
                log.error("arguments: {} , result code: {}, error info: {}",
                        String.join(" ", commandline.getArguments()),
                        ret,
                        errorStream.toString());
            }
        } catch (Exception e) {
            log.error(commandline.getExecutable(), e);
        }
    }

这个阶段发展了一段时间后，因为环境的复杂，运维的混乱，经常出现系统中安装 ffmpeg 不兼容的情况。

这时使用外部的 ffmpeg 的已经不能再满足我了，我突发奇想，要把编译生成好的 ffmpeg 打包到我的项目中去，我研究了一会之后，去找到了一个工具类 [a-schild/jave2](https://link.segmentfault.com/?enc=a2idVl25mubJ8fBeNVQEUA%3D%3D.866Ex8dWKj952F6XaqV47%2BDrqlLsch3a3uIkb7GA5ezyXWzrZ4WgRQKC%2F6uhkBB4)，这个工具类对 ffmpeg 进行了更加深入的封装，不仅能跨平台选择不同的 ffmpeg，还能根据 ffmpeg 的执行信息，更加深入的进行了文件格式、filter 的封装。

但是我只使用了其中的跨平台选择能力，因为之前的项目已经挺稳定了，不想再修改里面的使用方法。  

## 阶段四：最终阶段

这个阶段就比较有意思了，它可以说是使用 ffmpeg 的最佳方式了，项目来源于[javacpp](https://link.segmentfault.com/?enc=MiRyH40BD%2BU36bc1GSlMAQ%3D%3D.cV7XrBh5zncPOn6dq7fZbxdx1a6tfRbk%2BjUaIS4b3pv77WVFT2LvEEyOqmEk%2BINq)，这个项目有点厉害了，它的前置项目[javacpp-presets](https://link.segmentfault.com/?enc=bmzwsoDBzJpq1aLtN%2FKJuw%3D%3D.tjpUXKp8HixSIJzjvEv00Z46YV%2F7YieyC9qme80MN%2FWtIkf2IcZfw%2FmeqEmUzWD%2B)把大家经常用到的几十种 C 库都做成了 JNI 接口，比如 ffmpeg、dual、numpy、opencv，还有等等等等，实在是令人敬佩。

虽然说人家付出很值得让人尊敬，但是使用的时候不要一股脑把依赖都添加进去了。像下面这样，用到什么添加什么，用什么平台就添加什么平台，不然一个项目下来几个 G 的大小跑不掉了，作者可不会因为这个感激你，反而会觉得自己一番苦心被不懂欣赏的人糟蹋了。  

            <dependency>
                <groupId>org.bytedeco</groupId>
                <artifactId>javacpp</artifactId>
                <version>${javacpp.version}</version>
            </dependency>
            <dependency>
                <groupId>org.bytedeco</groupId>
                <artifactId>javacpp</artifactId>
                <version>${javacpp.version}</version>
                <classifier>linux-x86_64</classifier>
            </dependency>
            <dependency>
                <groupId>org.bytedeco</groupId>
                <artifactId>javacpp</artifactId>
                <version>${javacpp.version}</version>
                <classifier>macosx-x86_64</classifier>
            </dependency>
            <dependency>
                <groupId>org.bytedeco</groupId>
                <artifactId>ffmpeg</artifactId>
                <version>${javacpp-ffmpeg.version}-${javacpp.version}</version>
            </dependency>
            <dependency>
                <groupId>org.bytedeco</groupId>
                <artifactId>ffmpeg</artifactId>
                <version>${javacpp-ffmpeg.version}-${javacpp.version}</version>
                <classifier>linux-x86_64</classifier>
            </dependency>
            <dependency>
                <groupId>org.bytedeco</groupId>
                <artifactId>ffmpeg</artifactId>
                <version>${javacpp-ffmpeg.version}-${javacpp.version}</version>
                <classifier>macosx-x86_64</classifier>
            </dependency>
            <dependency>
                <groupId>org.bytedeco</groupId>
                <artifactId>javacv</artifactId>
                <version>${javacpp.version}</version>
                <exclusions>
                    <exclusion>
                        <groupId>*</groupId>
                        <artifactId>*</artifactId>
                    </exclusion>
                </exclusions>
            </dependency>

到了这里有两种使用方式，一种是原汁原味的使用 ffmpeg 最原生的 API，还有一种就是 javacpp 作者的又一良心作品 [javacv](https://link.segmentfault.com/?enc=JUGTg4TdzEhxHvvhF9DTDA%3D%3D.I3grp948phvCwj4YrPkWw0Hrn%2B%2Fb04UXXGOwDQH105RMHjwGoh1jyh3WtSFG%2BMk8)。

最原生的 API，代码量肯定会多一些，并且要控制好变量的销毁，所以我还是选用了 javacv 的调用。

于是就有了下面这段代码，下面调用了 MapUtil 这一段是使用了 [hutool工具类](https://link.segmentfault.com/?enc=7VARD2zzQ5ZjPZ%2BGA3YMPA%3D%3D.KKiHW9I1uRpMa8q2WqtwGwc6f7%2Fxsj5MGxlxrhp%2F8MV65Ivx3SvKEuH4X9RulQAO)，至于为什么这样用，感兴趣的可以自己去尝试一下。  

    @Test
    void test() {
        try (FFmpegFrameGrabber grabber = new FFmpegFrameGrabber("1111111.mp4");
             FFmpegFrameRecorder recorder = new FFmpegFrameRecorder("2.mp4", 1)) {
            grabber.start();
            grabber.setTimestamp(20 * 1000000);
            Map<String, Object> filter = MapUtil.filter(BeanUtil.beanToMap(grabber),
                    (Editor<Map.Entry<String, Object>>) stringObjectEntry ->
                            StrUtil.startWithAny(stringObjectEntry.getKey(),
                                    "image", "audio", "video") && !stringObjectEntry.getKey().endsWith("Stream")
                                    ? stringObjectEntry : null);
            BeanUtil.copyProperties(filter, recorder);
            recorder.start();
            Frame inFrame;
            boolean audioCompleted = false;
            boolean videoCompleted = false;
            while ((inFrame = grabber.grab()) != null) {
                if (inFrame.timestamp >= 50 * 1000 * 1000) {
                    if (AVMEDIA_TYPE_VIDEO == inFrame.streamIndex) {
                        videoCompleted = true;
                    } else if (AVMEDIA_TYPE_AUDIO == inFrame.streamIndex) {
                        audioCompleted = true;
                    }
                    if (audioCompleted && videoCompleted) {
                        break;
                    }
                }
                recorder.record(inFrame);
            }
        } catch (Exception exception) {
            exception.printStackTrace();
        }
    }

上面这段代码，作用是截取了视频中的其中一段，虽然看起来更加复杂了，代码量更加多了，但是相比于命令行形式，优势是压倒性的。

优势一：  
不会再产生新的进程，这就说明不用再监控进程运行时长了，使用起来不用考虑那么多，API 的调用基本上不可能出现阻塞的情况，ffmpeg 的 API 绝对是值得相信的，如果有异常情况，一定会第一时间通过返回值告知。

优势二：  
代码逻辑更加清晰，自定义能力更强。  
其实前面三个阶段，不管怎么优化，最终还是要处理成命令行的，处理成命令行之后从命令行本身看，目的是明确清晰的，但是从处理命令行的过程来看，逻辑是分化的，一会要处理输入路径，一会要处理时间参数，又要组装，又要执行，必然逻辑会有些零散。  
javacv 的这种方式，把音视频通过帧的方式传递给使用者，虽然入门有些困难，但是处理起来灵活多变，逻辑也更加清晰。

## 总结

从上面的这些使用方式来看，对你可能有一些启发，不仅是使用 ffmpeg，使用其它的工具比如 spring、reids，也是要有不断进步的方法，不能够一种方法能用了，能够解决目前的需求了，就不再最求进步，还要去思考怎么更加方便的解决后续更复杂的需求。
