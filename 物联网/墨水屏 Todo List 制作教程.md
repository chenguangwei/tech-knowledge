
墨水屏 Todo List 制作教程

我在 Twitter 发布的一条 tweet，写了使用了墨水屏制作了一个 Todo List，可以同步苹果提醒的全家待办列表。收到了很多推友的关注，所以我决定写一个教程，帮助大家制作一个墨水屏 Todo List。

同步服务由我开发的 einktodo.com 提供。


下面是制作墨水屏 Todo List 的教程。

必要材料和工具
材料
ESP32 开发板、墨水屏驱动板和微雪 7.5 寸墨水屏

ESP32 开发板
墨水屏驱动板
微雪 7.5 寸墨水屏
微雪 7.5 寸墨水屏可以在网上找到拆机屏，也可以买新的。

墨水屏驱动板在闲鱼搜索墨水屏驱动板，买 24 pin 的驱动板。

工具
电烙铁
电脑
制作步骤
1. 焊接转接板和 ESP32 开发板
   将墨水屏驱动板按下面的表格对应的脚位连接到 ESP32 开发板上。

墨水屏驱动板	ESP32 开发板
VCC	3.3V
GND	GND
SDA	Pin 14
SCK	Pin 13
CS	Pin 15
DC	Pin 0
RST	Pin 2
BUSY	Pin 4
将墨水屏的 FPC 排线连接到墨水屏驱动板。

注意排线的正反面。这个 FPC 排线端子的触点是在上面的，所以排线的触点朝上。我在第一次尝试点亮墨水屏的时候，就因为排线插反了，当时信心受到了极大的打击。

墨水屏连接到墨水屏驱动板

2. 安装固件编译和刷写环境
   下载 VSCode，并安装。

安装好 VSCode 后，打开 VSCode，点击左侧的扩展按钮，搜索 PlatformIO，点击安装。

安装 PlatformIO

3. 下载代码
   然后从 E-ink Todo List 固件代码开源仓库 下载固件代码。

点击右上角的绿色按钮，选择Download ZIP。下载完成后解压得到固件代码文件夹。

下载固件代码

4. 编译和烧录固件
   用 VSCode 打开固件代码文件夹。

打开文件夹

选择固件代码文件夹, PlatformIO 插件会自动安装编译固件所需依赖。

打开的代码

将 ESP32 开发板用 USB 线连接到电脑。

编译和烧录固件到 ESP32 开发板。

在 VScode 上点击蚂蚁头形状的PlatformIO菜单来打开 PlatformIO 的面板。选择 e-ink-todo-list -> General -> Upload and Monitor。

编译和烧录固件

等待烧录完成。

墨水屏 Todo List 设置
在 einktodo.com 获取 API Token
登录 einktodo.com。

在 API Key 页面 创建 API Key。

创建 API Key

设置墨水屏 Todo List
固件编译和烧录完成，墨水屏上会提示你连接墨水屏的热点和热点密码，接下来要连接热点进行设置。

连接墨水屏 Todo List 的 Wi-Fi: E-ink Todo List AP，密码: einktodo.com。

打开浏览器，进入墨水屏上显示的设置页面。

墨水屏 Todo List 初始化屏幕

设置页面

设置墨水屏要连接的 Wi-Fi 网络和要使用的 API Token。

点击Configure WiFi按钮进入设置页面。然后选择你想要让默水屏连接的 Wi-Fi 或在 SSID 输入框里输入 Wi-Fi 名。在 Password 输入框里输入这个 Wi-Fi 的密码。在 API Key 输入框里输入刚刚在 einktodo.com 获取的 API Key。

墨水屏 Todo List 设置页面

点击Save按钮。

墨水屏 Todo List 会自动重启并尝试连接 Wi-Fi。如果没有开始连接，你可以按 Reset 按钮重启 ESP32 开发板来触发连接。注意保持 Wi-Fi 信号良好。

如果尝试连接 3 次没有成功，墨水屏将会重新打开 Wi-Fi 热点，你可以重新连接 Wi-Fi 进行设置。

如果在 pin 16 （ESP8266 为 pin0）上连接了按键，长按按键也可以重新进入设置页面。

使用快捷指令同步苹果提醒
同步的 Todo List 目前支持苹果的提醒事项应用。其它常用的待办应用将来会逐步集成到 einktodo.com。

安装和配制快捷指令
点击下面的链接在 iPhone 上安装Sync Einktodo.com快捷指令。

Sync Einktodo.com 快捷指令

安装 Sync Einktodo.com 快捷指令

在 Safari 中打开上面的链接，点击获取捷径。

安装 Sync Einktodo.com 快捷指令

点击添加快捷指令。

添加完之后，找到 Sync Einktodo.com 快捷指令，点击它上面的...按钮，进入编辑界面。

Sync Einktodo.com 快捷指令编辑页面

在Sync Einktodo.com快捷指令中的文本块中输入刚刚在 einktodo.com 获取的 API Key。

按完成保存快捷指令。

设置快捷指令自动化
在快捷指令自动化中，设置打开或关闭“提醒事项”应用时运行此快捷指令来同步待办。

打开快捷指令应用，点击自动化 Tab

设置快捷指令自动化

点击+按钮创建自动化, 选择App，点击下一步。

创建自动化

设置为打开或关闭 提醒事项应用时运行快捷指令

App 那一项选择提醒事项,勾选打开或关闭，并选择立即执行，然后点击下一步。

设置同步触发条件

选择Sync Einktodo.com快捷指令。

选择 Sync Einktodo.com 快捷指令

这样设置后，当你打开或关闭提醒事项应用时，快捷指令会自动同步待办到墨水屏 Todo List。

允许快捷指令访问提醒事项

当你第一次触发这个自动化时，系统会提示你允许快捷指令将提醒事项应用的数据发送到 einktodo.com，点击始终允许

允许快捷指令访问提醒事项

对成品感兴趣吗？
我正在尝试将墨水屏 Todo List 制作成产品，如果你对这个产品感兴趣，可以在前往 einktodo.com 加入行等待邮件列表，你将会收到产品制作的最新进度信息。

我正在凑备量产产品的资源，在不久就会以原型产品为奖励的方式众筹启动资金。如果您加入了等待邮件列表，您将会收到众筹启动的通知和众筹的优惠信息。