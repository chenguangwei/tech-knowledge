# SpringBoot Admin集成Arthas实践



## 前言

Arthas 是 Alibaba开源的Java诊断工具，具有实时查看系统的运行状况；查看函数调用参数、返回值和异常；在线热更新代码；秒解决类冲突问题；定位类加载路径；生成热点；通过网页诊断线上应用。如今在各大厂都有广泛应用，也延伸出很多产品。

这里将介绍如何将Arthas集成进SpringBoot监控平台中。

## SpringBoot Admin

为了方便SpringBoot Admin 简称为SBA

版本：1.5.x
1.5版本的SBA如果要开发插件比较麻烦，需要下载SBA的源码包，再按照spring-boot-admin-server-ui-hystrix的形式copy一份,由于JS使用的是Angular,本人尝试了很久，虽然掌握了如何开发插件，奈何不会Angular，遂放弃💀

[![img](https://camo.githubusercontent.com/c05c3f8cfec9f1afa4668721c5509d4b56eeaa3b4a1d1e2048b274f8dadc7c47/68747470733a2f2f696d676b72322e636e2d626a2e7566696c656f732e636f6d2f38626635333166612d343262342d343532342d616130652d3430303832386161353563302e706e673f55436c6f75645075626c69634b65793d544f4b454e5f38643862373262652d353739612d346538332d626664302d356636636531353436663133265369676e61747572653d383168465642666a55357652306335465978336b7565634d586849253235334426457870697265733d31363037303637333834)](https://camo.githubusercontent.com/c05c3f8cfec9f1afa4668721c5509d4b56eeaa3b4a1d1e2048b274f8dadc7c47/68747470733a2f2f696d676b72322e636e2d626a2e7566696c656f732e636f6d2f38626635333166612d343262342d343532342d616130652d3430303832386161353563302e706e673f55436c6f75645075626c69634b65793d544f4b454e5f38643862373262652d353739612d346538332d626664302d356636636531353436663133265369676e61747572653d383168465642666a55357652306335465978336b7565634d586849253235334426457870697265733d31363037303637333834)

版本：2.x
2.x版本的SBA插件开发，官网有介绍如何开发，JS使用Vue，方便很多，由于我们项目还在使用1.5，所以并没有使用该版本，请读者自行尝试

不能使用SBA的插件进行集成，那还有什么办法呢？😅

## SBA 集成

鄙人的办法是将Arthas的相关文件直接copy到admin服务中，这些文件都来自arthas-all项目tunnel-server

[![admin目录结构](https://camo.githubusercontent.com/0ef69351f47cfea7e76d4c475e655ed06d3a9bfffef628ffa3c8f8a691fdf756/68747470733a2f2f696d676b72322e636e2d626a2e7566696c656f732e636f6d2f65313764376465322d393266392d346236662d613635612d3334643538323865383034622e706e673f55436c6f75645075626c69634b65793d544f4b454e5f38643862373262652d353739612d346538332d626664302d356636636531353436663133265369676e61747572653d62655561555a31695270716b583743496b44656637723557783234253235334426457870697265733d31363037303637343339)](https://camo.githubusercontent.com/0ef69351f47cfea7e76d4c475e655ed06d3a9bfffef628ffa3c8f8a691fdf756/68747470733a2f2f696d676b72322e636e2d626a2e7566696c656f732e636f6d2f65313764376465322d393266392d346236662d613635612d3334643538323865383034622e706e673f55436c6f75645075626c69634b65793d544f4b454e5f38643862373262652d353739612d346538332d626664302d356636636531353436663133265369676e61747572653d62655561555a31695270716b583743496b44656637723557783234253235334426457870697265733d31363037303637343339)

### arthas目录

该包下存放的是所有arthas的Java文件

- endpoint包下的文件可以都注释掉，没多大用
- ArthasController这个文件是我自己新建的，用来获取所有注册到Arthas的客户端，这在后面是有用的
- 其他文件直接copy过来就行

```
@RequestMapping("/api/arthas")
@RestController
public class ArthasController {
	@Autowired
	private TunnelServer tunnelServer;
  
	@RequestMapping(value = "/clients", method = RequestMethod.GET)
	public Set<String> getClients() {
		Map<String, AgentInfo> agentInfoMap = tunnelServer.getAgentInfoMap();
		return agentInfoMap.keySet();
	}
}
```

`spring-boot-admin-server-ui`
该文件建在resources.META-INF下，admin会在启动的时候加载该目录下的文件

[![img](https://camo.githubusercontent.com/511f60b578ed20030bbae2bd9d1d0915221a389bc583b6f65d9073ea9ca1d3d0/68747470733a2f2f696d676b72322e636e2d626a2e7566696c656f732e636f6d2f64623762633433392d396531612d343732622d626431332d6433383466316662623934372e706e673f55436c6f75645075626c69634b65793d544f4b454e5f38643862373262652d353739612d346538332d626664302d356636636531353436663133265369676e61747572653d6368302532353246744c326d636f6e784e57765473736279726866386e6e49253235334426457870697265733d31363037303634303637)](https://camo.githubusercontent.com/511f60b578ed20030bbae2bd9d1d0915221a389bc583b6f65d9073ea9ca1d3d0/68747470733a2f2f696d676b72322e636e2d626a2e7566696c656f732e636f6d2f64623762633433392d396531612d343732622d626431332d6433383466316662623934372e706e673f55436c6f75645075626c69634b65793d544f4b454e5f38643862373262652d353739612d346538332d626664302d356636636531353436663133265369676e61747572653d6368302532353246744c326d636f6e784e57765473736279726866386e6e49253235334426457870697265733d31363037303634303637)

### resources目录

- index.html 覆盖SBA原来的首页，在其中添加一个Arthas导航
  [![img](https://camo.githubusercontent.com/7de64a302040c4efc3afd00a6eff6945ec0b88d3456f2ce4e82b7a985ee1dc88/68747470733a2f2f696d676b72322e636e2d626a2e7566696c656f732e636f6d2f66643133643733622d623363332d343538312d613737332d3534653166316536393438612e706e673f55436c6f75645075626c69634b65793d544f4b454e5f38643862373262652d353739612d346538332d626664302d356636636531353436663133265369676e61747572653d527848512532353246797a306b3461334568484f4a6f4a664348364f4f3841253235334426457870697265733d31363037303634323638)](https://camo.githubusercontent.com/7de64a302040c4efc3afd00a6eff6945ec0b88d3456f2ce4e82b7a985ee1dc88/68747470733a2f2f696d676b72322e636e2d626a2e7566696c656f732e636f6d2f66643133643733622d623363332d343538312d613737332d3534653166316536393438612e706e673f55436c6f75645075626c69634b65793d544f4b454e5f38643862373262652d353739612d346538332d626664302d356636636531353436663133265369676e61747572653d527848512532353246797a306b3461334568484f4a6f4a664348364f4f3841253235334426457870697265733d31363037303634323638)

```
<!DOCTYPE html>
<html class="no-js">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>Spring Boot Admin</title>
    <meta name="description" content="">
    <meta name="viewport" content="width=device-width">
    <link rel="shortcut icon" type="image/x-icon" href="img/favicon.png"/>
    <link rel="stylesheet" type="text/css" href="core.css"/>
    <link rel="stylesheet" type="text/css" href="all-modules.css"/>
</head>

<body>
<header class="navbar header--navbar desktop-only">
    <div class="navbar-inner">
        <div class="container-fluid">

            <div class="spring-logo--container">
                <a class="spring-logo" href="#"><span></span></a>
            </div>
            <div class="spring-logo--container">
                <a class="spring-boot-logo" href="#"><span></span></a>
            </div>
            <ul class="nav pull-right">
              
              <!--增加Arthas导航-->
                <li class="navbar-link ng-scope">
                    <a  class="ng-binding" href="arthas/arthas.html">Arthas</a>
                </li>
                <li ng-repeat="view in mainViews" class="navbar-link" ng-class="{active: $state.includes(view.state)}">
                    <a ui-sref="{{view.state}}" ng-bind-html="view.title"></a>
                </li>
            </ul>
        </div>
    </div>
</header>

<div ui-view></div>

<footer class="footer">
    <ul class="inline">
        <li><a href="https://codecentric.github.io/spring-boot-admin/@project.version@" target="_blank">Reference
            Guide</a></li>
        <li>-</li>
        <li><a href="https://github.com/codecentric/spring-boot-admin" target="_blank">Sources</a></li>
        <li>-</li>
        <li>Code licensed under <a href="http://www.apache.org/licenses/LICENSE-2.0" target="_blank">Apache License
            2.0</a></li>
    </ul>
</footer>

<script src="dependencies.js" type="text/javascript"></script>
<script type="text/javascript">
  sbaModules = [];
</script>
<script src="core.js" type="text/javascript"></script>
<script src="all-modules.js" type="text/javascript"></script>
<script type="text/javascript">
  angular.element(document).ready(function () {
    angular.bootstrap(document, sbaModules.slice(0), {
      strictDi: true
    });
  });
</script>
</body>
</html>
```

- arthas.html
  新建页面，用于显示arthas控制台页面。
  这个文件中有两个隐藏文本域，这两个用于连接arthas服务端，在页面加载的时候会自动将admin的url赋值给ip

```
<input type="hidden" id="ip" name="ip" value="127.0.0.1">
<input type="hidden" id="port" name="port" value="19898">
<!DOCTYPE html>
<html class="no-js">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>Spring Boot Admin</title>
    <meta name="description" content="">
    <meta name="viewport" content="width=device-width">
    <link rel="shortcut icon" type="image/x-icon" href="../img/favicon.png"/>
    <link rel="stylesheet" type="text/css" href="../core.css"/>
    <link rel="stylesheet" type="text/css" href="../all-modules.css"/>
    <script src="js/jquery-3.3.1.min.js"></script>
    <script src="js/popper-1.14.6.min.js"></script>
    <script src="js/xterm.js"></script>
    <script src="js/web-console.js"></script>
    <script src="js/arthas.js"></script>
    <link href="js/xterm.css" rel="stylesheet" />

    <script type="text/javascript">
        window.addEventListener('resize', function () {
            var terminalSize = getTerminalSize();
            ws.send(JSON.stringify({ action: 'resize', cols: terminalSize.cols, rows: terminalSize.rows }));
            xterm.resize(terminalSize.cols, terminalSize.rows);
        });
    </script>
</head>

<body>
<header class="navbar header--navbar desktop-only">
    <div class="navbar-inner">
        <div class="container-fluid">
            <div class="spring-logo--container">
                <a class="spring-logo" href="#"><span></span></a>
            </div>
            <div class="spring-logo--container">
                <a class="spring-boot-logo" href="#"><span></span></a>
            </div>
            <ul class="nav pull-right">
                <li class="navbar-link ng-scope">
                    <a  class="ng-binding" href="arthas.html">Arthas</a>
                </li>
                <li class="navbar-link ng-scope">
                    <a  class="ng-binding" href="../">Applications</a>
                </li>
                <li class="navbar-link ng-scope">
                    <a  class="ng-binding" href="../#/turbine">Turbine</a>
                </li>
                <li class="navbar-link ng-scope">
                    <a  class="ng-binding" href="../#/events">Journal</a>
                </li>
                <li class="navbar-link ng-scope">
                    <a  class="ng-binding" href="../#/about">About</a>
                </li>
                <li class="navbar-link ng-scope">
                    <a  class="ng-binding" href="../#/logout"><i class="fa fa-2x fa-sign-out" aria-hidden="true"></i></a>
                </li>
            </ul>
        </div>
    </div>
</header>

<div ui-view>
    <div class="container-fluid">
        <form class="form-inline">
            <input type="hidden" id="ip" name="ip" value="127.0.0.1">
            <input type="hidden" id="port" name="port" value="19898">
            Select Application：
            <select id="selectServer"></select>
            <button class="btn" onclick="startConnect()" type="button"><i class="fa fa-connectdevelop"></i> Connect</button>
            <button class="btn" onclick="disconnect()" type="button"><i class="fa fa-search-minus"></i> Disconnect</button>
            <button class="btn" onclick="release()" type="button"><i class="fa fa-search-minus"></i> Release</button>
        </form>
        <div id="terminal-card">
            <div id="terminal"></div>
        </div>
    </div>
</div>

</body>
</html>
```

- arthas.js 存储页面控制的js

```
var registerApplications = null;
var applications = null;
$(document).ready(function () {
    reloadRegisterApplications();
    reloadApplications();
});

/**
 * 获取注册的arthas客户端
 */
function reloadRegisterApplications() {
    var result = reqSync("/api/arthas/clients", "get");
    registerApplications = result;
    initSelect("#selectServer", registerApplications, "");
}

/**
 * 获取注册的应用
 */
function reloadApplications() {
    applications = reqSync("/api/applications", "get");
    console.log(applications)
}

/**
 * 初始化下拉选择框
 */
function initSelect(uiSelect, list, key) {
    $(uiSelect).html('');
    var server;
    for (var i = 0; i < list.length; i++) {
        server = list[i].toLowerCase().split("@");
        if ("phantom-admin" === server[0]) continue;
        $(uiSelect).append("<option value=" + list[i].toLowerCase() + ">" + server[0] + "</option>");
    }
}

/**
 * 重置配置文件
 */
function release() {
    var currentServer = $("#selectServer").text();
    for (var i = 0; i < applications.length; i++) {
        serverId = applications[i].id;
        serverName = applications[i].name.toLowerCase();
        console.log(serverId + "/" + serverName);
        if (currentServer === serverName) {
            var result = reqSync("/api/applications/" +serverId+ "/env/reset", "post");
            alert("env reset success");
        }
    }
}

function reqSync(url, method) {
    var result = null;
    $.ajax({
        url: url,
        type: method,
        async: false, //使用同步的方式,true为异步方式
        headers: {
            'Content-Type': 'application/json;charset=utf8;',
        },
        success: function (data) {
            // console.log(data);
            result = data;
        },
        error: function (data) {
            console.log("error");
        }
    });
    return result;
}
```

- web-console.js
  修改了连接部分代码，参考一下

```
var ws;
var xterm;

/**有修改**/
$(function () {
    var url = window.location.href;
    var ip = getUrlParam('ip');
    var port = getUrlParam('port');
    var agentId = getUrlParam('agentId');

    if (ip != '' && ip != null) {
        $('#ip').val(ip);
    } else {
        $('#ip').val(window.location.hostname);
    }
    if (port != '' && port != null) {
        $('#port').val(port);
    }
    if (agentId != '' && agentId != null) {
        $('#selectServer').val(agentId);
    }

    // startConnect(true);
});

/** get params in url **/
function getUrlParam (name, url) {
    if (!url) url = window.location.href;
    name = name.replace(/[\[\]]/g, '\\$&');
    var regex = new RegExp('[?&]' + name + '(=([^&#]*)|&|#|$)'),
        results = regex.exec(url);
    if (!results) return null;
    if (!results[2]) return '';
    return decodeURIComponent(results[2].replace(/\+/g, ' '));
}

function getCharSize () {
    var tempDiv = $('<div />').attr({'role': 'listitem'});
    var tempSpan = $('<div />').html('qwertyuiopasdfghjklzxcvbnm');
    tempDiv.append(tempSpan);
    $("html body").append(tempDiv);
    var size = {
        width: tempSpan.outerWidth() / 26,
        height: tempSpan.outerHeight(),
        left: tempDiv.outerWidth() - tempSpan.outerWidth(),
        top: tempDiv.outerHeight() - tempSpan.outerHeight(),
    };
    tempDiv.remove();
    return size;
}

function getWindowSize () {
    var e = window;
    var a = 'inner';
    if (!('innerWidth' in window )) {
        a = 'client';
        e = document.documentElement || document.body;
    }
    var terminalDiv = document.getElementById("terminal-card");
    var terminalDivRect = terminalDiv.getBoundingClientRect();
    return {
        width: terminalDivRect.width,
        height: e[a + 'Height'] - terminalDivRect.top
    };
}

function getTerminalSize () {
    var charSize = getCharSize();
    var windowSize = getWindowSize();
    console.log('charsize');
    console.log(charSize);
    console.log('windowSize');
    console.log(windowSize);
    return {
        cols: Math.floor((windowSize.width - charSize.left) / 10),
        rows: Math.floor((windowSize.height - charSize.top) / 17)
    };
}

/** init websocket **/
function initWs (ip, port, agentId) {
    var protocol= location.protocol === 'https:'  ? 'wss://' : 'ws://';
    var path = protocol + ip + ':' + port + '/ws?method=connectArthas&id=' + agentId;
    ws = new WebSocket(path);
}

/** init xterm **/
function initXterm (cols, rows) {
    xterm = new Terminal({
        cols: cols,
        rows: rows,
        screenReaderMode: true,
        rendererType: 'canvas',
        convertEol: true
    });
}


/** 有修改 begin connect **/
function startConnect (silent) {
    var ip = $('#ip').val();
    var port = $('#port').val();
    var agentId = $('#selectServer').val();
    if (ip == '' || port == '') {
        alert('Ip or port can not be empty');
        return;
    }
    if (agentId == '') {
        if (silent) {
            return;
        }
        alert('AgentId can not be empty');
        return;
    }
    if (ws != null) {
        alert('Already connected');
        return;
    }
    // init webSocket
    initWs(ip, port, agentId);
    ws.onerror = function () {
        ws.close();
        ws = null;
        !silent && alert('Connect error');
    };
    ws.onclose = function (message) {
        if (message.code === 2000) {
            alert(message.reason);
        }
    };
    ws.onopen = function () {
        console.log('open');
        $('#fullSc').show();
        var terminalSize = getTerminalSize()
        console.log('terminalSize')
        console.log(terminalSize)
        // init xterm
        initXterm(terminalSize.cols, terminalSize.rows)
        ws.onmessage = function (event) {
            if (event.type === 'message') {
                var data = event.data;
                xterm.write(data);
            }
        };
        xterm.open(document.getElementById('terminal'));
        xterm.on('data', function (data) {
            ws.send(JSON.stringify({action: 'read', data: data}))
        });
        ws.send(JSON.stringify({action: 'resize', cols: terminalSize.cols, rows: terminalSize.rows}));
        window.setInterval(function () {
            if (ws != null && ws.readyState === 1) {
                ws.send(JSON.stringify({action: 'read', data: ""}));
            }
        }, 30000);
    }
}

function disconnect () {
    try {
        ws.close();
        ws.onmessage = null;
        ws.onclose = null;
        ws = null;
        xterm.destroy();
        $('#fullSc').hide();
        alert('Connection was closed successfully!');
    } catch (e) {
        alert('No connection, please start connect first.');
    }
}

/** full screen show **/
function xtermFullScreen () {
    var ele = document.getElementById('terminal-card');
    requestFullScreen(ele);
}

function requestFullScreen (element) {
    var requestMethod = element.requestFullScreen || element.webkitRequestFullScreen || element.mozRequestFullScreen || element.msRequestFullScreen;
    if (requestMethod) {
        requestMethod.call(element);
    } else if (typeof window.ActiveXObject !== "undefined") {
        var wscript = new ActiveXObject("WScript.Shell");
        if (wscript !== null) {
            wscript.SendKeys("{F11}");
        }
    }
}
```

- 其他文件
  jquery-3.3.1.min.js 新加Js
  copy过来的js
  popper-1.14.6.min.js
  web-console.js
  xterm.css
  xterm.js
- bootstrap.yml

```
# arthas端口
arthas:
  server:
    port: 9898
```

这样子，admin端的配置完成了

## 客户端配置

- 在配置中心加入配置

```
#arthas服务端域名
arthas.tunnel-server = ws://admin域名/ws
#客户端id,应用名@随机值，js会截取前面的应用名
arthas.agent-id = ${spring.application.name}@${random.value}
#arthas开关，可以在需要调式的时候开启，不需要的时候关闭
spring.arthas.enabled = false
```

- 需要自动Attach的应用中引入arthas-spring-boot-starter
  需要对starter进行部分修改，`要将注册arthas的部分移除`，下面是修改后的文件。
  我这里是将修改后的文件重新打包成jar包，上传到私服，但有些应用会有无法加载arthasConfigMap的情况，可以将这两个文件单独放到项目的公共包中

```
@EnableConfigurationProperties({ ArthasProperties.class })
public class ArthasConfiguration {
	private static final Logger logger = LoggerFactory.getLogger(ArthasConfiguration.class);

	@ConfigurationProperties(prefix = "arthas")
	@ConditionalOnMissingBean
	@Bean
	public HashMap<String, String> arthasConfigMap() {
		return new HashMap<String, String>();
	}

}
@ConfigurationProperties(prefix = "arthas")
public class ArthasProperties {
	private String ip;
	private int telnetPort;
	private int httpPort;

	private String tunnelServer;
	private String agentId;

	/**
	 * report executed command
	 */
	private String statUrl;

	/**
	 * session timeout seconds
	 */
	private long sessionTimeout;

	private String home;

	/**
	 * when arthas agent init error will throw exception by default.
	 */
	private boolean slientInit = false;

	public String getHome() {
		return home;
	}

	public void setHome(String home) {
		this.home = home;
	}

	public boolean isSlientInit() {
		return slientInit;
	}

	public void setSlientInit(boolean slientInit) {
		this.slientInit = slientInit;
	}

	public String getIp() {
		return ip;
	}

	public void setIp(String ip) {
		this.ip = ip;
	}

	public int getTelnetPort() {
		return telnetPort;
	}

	public void setTelnetPort(int telnetPort) {
		this.telnetPort = telnetPort;
	}

	public int getHttpPort() {
		return httpPort;
	}

	public void setHttpPort(int httpPort) {
		this.httpPort = httpPort;
	}

	public String getTunnelServer() {
		return tunnelServer;
	}

	public void setTunnelServer(String tunnelServer) {
		this.tunnelServer = tunnelServer;
	}

	public String getAgentId() {
		return agentId;
	}

	public void setAgentId(String agentId) {
		this.agentId = agentId;
	}

	public String getStatUrl() {
		return statUrl;
	}

	public void setStatUrl(String statUrl) {
		this.statUrl = statUrl;
	}

	public long getSessionTimeout() {
		return sessionTimeout;
	}

	public void setSessionTimeout(long sessionTimeout) {
		this.sessionTimeout = sessionTimeout;
	}

}
```

- 实现开关效果
  为了实现开关效果，还需要一个文件用来监听配置文件的改变
  我这里使用的是在SBA中改变环境变量，对应服务监听到变量改变，当监听`spring.arthas.enabled`为true的时候，注册arthas, 到下面是代码

```
@Component
public class EnvironmentChangeListener implements ApplicationListener<EnvironmentChangeEvent> {

    @Autowired
    private Environment env;

    @Autowired
    private Map<String, String> arthasConfigMap;

    @Autowired
    private ArthasProperties arthasProperties;

    @Autowired
    private ApplicationContext applicationContext;

    @Override
    public void onApplicationEvent(EnvironmentChangeEvent event) {
        Set<String> keys = event.getKeys();
        for (String key : keys) {
            if ("spring.arthas.enabled".equals(key)) {
                if ("true".equals(env.getProperty(key))) {
                    registerArthas();
                }
            }
        }
    }

    private void registerArthas() {
        DefaultListableBeanFactory defaultListableBeanFactory = (DefaultListableBeanFactory) applicationContext.getAutowireCapableBeanFactory();
        String bean = "arthasAgent";
        if (defaultListableBeanFactory.containsBean(bean)) {
            ((ArthasAgent)defaultListableBeanFactory.getBean(bean)).init();
            return;
        }
        defaultListableBeanFactory.registerSingleton(bean, arthasAgentInit());
    }

    private ArthasAgent arthasAgentInit() {
        arthasConfigMap = StringUtils.removeDashKey(arthasConfigMap);
        // 给配置全加上前缀
        Map<String, String> mapWithPrefix = new HashMap<String, String>(arthasConfigMap.size());
        for (Map.Entry<String, String> entry : arthasConfigMap.entrySet()) {
            mapWithPrefix.put("arthas." + entry.getKey(), entry.getValue());
        }
        final ArthasAgent arthasAgent = new ArthasAgent(mapWithPrefix, arthasProperties.getHome(),
                arthasProperties.isSlientInit(), null);
        arthasAgent.init();
        return arthasAgent;
    }


}
```

## 结束

到此可以愉快的在SBA中调式应用了，看看最后的页面

[![img](https://camo.githubusercontent.com/c483bbcf66659797d64517c76ff4f3b969aaceca9fad80269247f0df199a8072/68747470733a2f2f696d676b72322e636e2d626a2e7566696c656f732e636f6d2f31353265363466612d333735302d346262302d623964312d6466343539396237636231662e706e673f55436c6f75645075626c69634b65793d544f4b454e5f38643862373262652d353739612d346538332d626664302d356636636531353436663133265369676e61747572653d516c51596557414251666f673068753364366743366d594c25323532466963253235334426457870697265733d31363037303636323830)](https://camo.githubusercontent.com/c483bbcf66659797d64517c76ff4f3b969aaceca9fad80269247f0df199a8072/68747470733a2f2f696d676b72322e636e2d626a2e7566696c656f732e636f6d2f31353265363466612d333735302d346262302d623964312d6466343539396237636231662e706e673f55436c6f75645075626c69634b65793d544f4b454e5f38643862373262652d353739612d346538332d626664302d356636636531353436663133265369676e61747572653d516c51596557414251666f673068753364366743366d594c25323532466963253235334426457870697265733d31363037303636323830)

- 调式流程
  开启Arthas
  [![img](https://camo.githubusercontent.com/09d8ec53d0d534c65460159348623df06f3516275dc4576ab1a6f36f02affa73/68747470733a2f2f696d676b72322e636e2d626a2e7566696c656f732e636f6d2f33366634646434362d633764652d346163302d616634302d6165326365643162653536312e706e673f55436c6f75645075626c69634b65793d544f4b454e5f38643862373262652d353739612d346538332d626664302d356636636531353436663133265369676e61747572653d575439513455474d70593149505030656c456b6932567635766567253235334426457870697265733d31363037303636343835)](https://camo.githubusercontent.com/09d8ec53d0d534c65460159348623df06f3516275dc4576ab1a6f36f02affa73/68747470733a2f2f696d676b72322e636e2d626a2e7566696c656f732e636f6d2f33366634646434362d633764652d346163302d616634302d6165326365643162653536312e706e673f55436c6f75645075626c69634b65793d544f4b454e5f38643862373262652d353739612d346538332d626664302d356636636531353436663133265369676e61747572653d575439513455474d70593149505030656c456b6932567635766567253235334426457870697265733d31363037303636343835)

在Select Application中选择应用
Connect 连接应用
DisConnect 断开应用
Release 释放配置文件

一些缺陷：

- 使用jar包的方式引入应用，具有一定的侵略性，如果arthas无法启动，会导致应用也无法启动
- 如果使用docker，需要适当调整JVM内存，防止开启arthas、调试的时候，内存炸了
- 没有使用SBA插件的方式集成
- 如上集成仅供参考，请根据自己企业的情况来集成