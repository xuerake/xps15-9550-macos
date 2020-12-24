## 配置

| 型号      | XPS15-9550/MacBookPro13,3  | 版本     | 10.15.7 19H2 |
| :-------- | :------------------------- | :------- | :------------------ |
| 处理器 | Intel Core i7-6700HQ       | 核显 | HD Graphics 530     |
| 内存   | 镁光 DDR4 2400MHz 16 * 2   | 存储     | THNSN5512GVU7 SAMPLE |
| 声卡   | Realtek ALC298            | 网卡     | Dell Wireless 1830  |
| 内屏   | Sharp SHP1453 UHD          | 显示器  | 无  |

### 不工作的设备

- 独立显卡 
- 雷电
- 蓝牙的两种模式不会自动切换 stereo handsfree, 视频通话时会调用handsfree,导致背景雪花音

## 安装

**请下载 [最新的 release](https://github.com/xxxzc/xps15-9550-macos/releases/latest)**。
  201106-2 这个版本OC才行，其它的版本都会卡Logo和无法识别OC

- BRCM：博通/戴尔网卡版本。

### Big Sur

最新 release 的 OpenCore 可以安装或者 OTA 到 Big Sur（**但**安装到 SM961/PM961 很有可能失败，我尝试了各种姿势都没用，换了 SN550 成功了），请自负风险，安装遇到问题请不要发 issue，除非有解决方法。

**对于 4K 内屏用户**，WhateverGreen [978cb8](https://github.com/acidanthera/WhateverGreen/commit/978cb8c7a744ac189074225fd8eb2f16feb5a4c0)  能让内屏运行于 60Hz 了，不再需要 48Hz 补丁，Release [201218](https://github.com/xxxzc/xps15-9550-macos/releases/tag/201218) 包含了这个 WhateverGreen 并且修改了相关属性，可以直接使用。如果想要自己修改，可以看提交改了啥。

### FHD内屏

如果你的笔记本内屏是1080p，你需要修改以下配置：

- OC:  `NVRAM/Add/4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14/UIScale`  -> `01`
- CLOVER: `BootGraphics/UIScale` -> `1`

或者运行 `python3 update.py --display fhd`

## 安装后

可以使用 *Clover Configurator* 或者 *OpenCore Configurator* 修改配置文件，但更建议直接使用代码编辑器。

如果你更新或增加了Kexts/Drivers，你可以运行 `python3 update.py --config`，它会自动将这些更新信息更新到 config.plist 中，如果修改了 ACPI，则运行 `python3 update.py --acpi`。

可以运行 `python3 update.py --self` 以更新 update.py。

可以参考 [wmchris's tutorial](https://github.com/wmchris/DellXPS15-9550-OSX) 的安装教程和一些常见问题的解决方法。但使用本库的配置遇到问题时，请在本库创建 issue。

### 静默启动

默认下开机参数中有 `-v` ，会在启动过程中打印 logs 到屏幕上，删除它以关闭啰嗦模式：

```python
python3 update.py --set bootarg--v
```

### 耳机

耳机在电池模式下有几率在使用一段时间后产生杂音，请下载 [ComboJack](https://github.com/hackintosh-stuff/ComboJack/tree/master/ComboJack_Installer) 并运行其中的 install.sh 安装该耳机守护进程。

### 睡眠和唤醒

请运行以下指令以保证正常睡眠：

```shell
sudo pmset -a hibernatemode 0
sudo pmset -a autopoweroff 0
sudo pmset -a standby 0
sudo pmset -a proximitywake 0
```

或者执行  `python3 update.py --fixsleep`。

除了“当显示器关闭时，防止电脑自动进入睡眠”是可选的外，请关闭设置-节能器里的所有其他选项。

### 三码

请使用你自己的三码（SN，MLB 和 SmUUID），你可以复制一份 [sample_smbios.json](./sample_smbios.json)，将其中的 `sn mlb smuuid` 修改为你自己的，然后运行 `python3 update.py --smbios xxx.json`，`xxx.json` 为你的 smbios.json 文件。

如果你没有三码，你可以运行 `python3 update.py --smbios gen` 来生成一份新的三码，会自动保存到 `gen_smbios.json` 和 config 中。

#### SmUUID

建议你使用 Windows 的 UUID 作为 SmUUID，特别是如果你需要使用 OpenCore 启动 Windows：在 Windows 的 CMD 中运行 `wmic csproduct get UUID` 即可得到该 UUID。
使用win10 安装盘的SmUUID，不用重复激活

#### ROM

ROM 是修复 iServices 的关键属性之一，你可以运行：

```python
python3 update.py --set rom=$(ifconfig en0 | awk '/ether/{print $2}' | sed -e 's/\://g')
```

以使用 en0 的 MAC 地址作为 ROM。

### 平滑字体

如果你的内屏是1080p，或者使用1080p的显示器，请运行以下指令以启动字体平滑：

```
defaults write -g CGFontRenderingFontSmoothingDisabled -bool NO
```

### CLOVER主题

可以使用如下执行设置 CLOVER 的主题为 [themes](https://sourceforge.net/p/cloverefiboot/themes/ci/master/tree/themes/) 中的某个主题（xxx 为主题名）：

```sh
python3 update.py --set theme=xxx # will download if not exist
```

### NTFS写入

你需要将 `UUID=xxx none ntfs rw,auto,nobrowse` 添加到 `/etc/fstab` 中，**xxx** 为你的 NTFS 分区的 UUID。

如果你的 NTFS 分区装有 Windows，你需要先在 Windows 的 powershell 上运行 `powercfg -h off` 关闭 Windows 的休眠。

1、打开应用程序 - 实用工具 - 终端 运行如下命令。来查看你的硬盘UUID。
diskutil info /Volumes/MACX | grep UUID
特别注意：用你的硬盘的名字替换掉 MACX
2、再运行如下命令：
echo "UUID=EC9AB3F7-9AF6-F2EC-C4EC-F22419F32464 none ntfs rw,auto,nobrowse" | sudo tee -a /etc/fstab
用上一步操作得到的硬盘UUID替换命令行UUID=后面的字符，并且输入账户的密码 (如密码为空，请先创建密码。 输入密码不显示但实际已经输入)
3、随后，当你再重新连接此 USB 设备的时候， 桌面上不再显示这个 USB 分区的白色盒子图标。 你需要按 Command-Shift-G  前往 /Volumes 卷宗目录。
重启后，其它盘的就可以当作仓库盘

### 触摸板单双击延迟

- 关闭拖拽或者使用三指拖拽可以避免单击的延迟
- 关闭智能缩放可以避免双击的延迟

参考 [is-it-possible-to-get-rid-of-the-delay-between-right-clicking-and-seeing-the-context-menu](https://apple.stackexchange.com/a/218181)

尝试过程
20201223
尝试升级15.7，使用的黑果小兵制作的镜像，macOS Catalina 10.15.7(19H15) Installer for Clover 5126 and OC 0.6.3 and PE 32G

第一次，配置好
OC: NVRAM/Add/4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14/UIScale -> AQ==
CLOVER: BootGraphics/UIScale -> 1
和三码
clover 卡logo OC: Failed to load configuration

第二次，直接用下载的clover 和oc 症状相同

update:
201210 故障依旧
201208 故障依旧
200807 故障依旧
201116-2 clover还是卡Logo OC能进入安装，但是磁盘抹除失败，EFI盘调到500M, 再次进入，识别不到硬盘
明天再尝试
update:201116-2
修改OC config
OC: NVRAM/Add/4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14/UIScale -> 01 每改成AQ==
用win 的UUID 和生成的三码
目前可以已经刷好 Catalina 15.7 9H2
目前问题 声音有爆音
bug
使用safari 声音有爆音
蓝牙bug, 蓝牙两种模式，视频，音乐等时，会调用stereo
但是一旦视频通话，就会调用 handsfree, 然后就会有很大的杂音
暂时解决办法，声音设置里，把输入改成电脑内置
问题：视频通话和音频播放，只能存在一个？

双系统问题
把OC EFI拷贝到ESP启动盘，然后默认会在OC里启动win10,出现引导错误
进PE修复，又会出现OC无法引导的总是，都在抢Boot.efi
现在暂时是U盘引导，然后ESP引导win10, PD可以正常用bootcamp 使用双系统
有知道的大佬麻烦 告知下双系统完美启动的方法

双系统引导的问题
尝试过以下路线
1 将ESP区，EFI文件夹清空，移入OC引导文件夹，引导win10失败
修复win10引导，OC失败
2 参考https://imacos.top/2020/04/06/1559/
用easyUEFI 加入OC选项，使用\EFI\OC\opencore.EFI
并放在首位，引导win10失败
3 参考https://imacos.top/2020/04/06/1559/
用easyUEFI 加入OC选项，使用\EFI\OC\opencore.EFI
将win10放在首位，OC 使用F12 手动引导
进入系统后，使用parallels desktop ,bootcamp 引导win10 又变成OC引导，失败
目前解决办法
win10 boot camp 放首位
OC放U盘引导，反正一般不断电就不重启

4 最终解决办法
win10 boot camp 放首位
系统默认进入win10
修改config文件，boot\ 移除microsoft 选项，使得parallels desktop 用boot camp 启动时，不调用OC而是直接调用win10的
后续再把OC调到首位看行不行
目前，开机可以手动选择进入 win10 或是 Mac parallels desktop 也可以用boot camp 启动

EFI文件打包上传至repl

## 感谢

- [acidanthera](https://github.com/acidanthera) 提供绝大部分的驱动
- [alexandred](https://github.com/alexandred) 提供 VoodooI2C
- [headkaze](https://github.com/headkaze) 提供非常有用的 [Hackintool](https://www.tonymacx86.com/threads/release-hackintool-v2-8-6.254559/)
- [daliansky](https://github.com/daliansky) 提供非常详尽的 OpenCore 补丁教程 [OC-little](https://github.com/daliansky/OC-little/) 以及最新的解决方法 [XiaoMi-Pro-Hackintosh](https://github.com/daliansky/XiaoMi-Pro-Hackintosh) [黑果小兵的部落阁](https://blog.daliansky.net/)
- [RehabMan](https://github.com/RehabMan) 提供的热补丁 [hotpatches](https://github.com/RehabMan/OS-X-Clover-Laptop-Config/tree/master/hotpatch) 和热补丁教程
- [knnspeed](https://www.tonymacx86.com/threads/guide-dell-xps-15-9560-4k-touch-1tb-ssd-32gb-ram-100-adobergb.224486) 提供的 Combojack 和描述详尽的热补丁以及 USB-C 热插拔解决方法
- [wmchris](https://github.com/wmchris/DellXPS15-9550-OSX/tree/10.15) 提供的 XPS15 详细安装教程
