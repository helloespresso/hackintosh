# 前言

hackintosh很有趣，尤其对于我这种家境贫寒的人来说，但仅供爱好者交流，请勿用作商业用途。

# 平台硬件

| 硬件                              | 型号                                                         | 备注                                                         |
| --------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 主板                              | GIGABYTE AORUS Z390 I PRO WIFI                               | BIOS版本号：F8c，**解锁CFG**  ！！！                         |
| CPU                               | Intel i9 9900K                                               | 建议买带核显的，否则一些重要功能不完善                       |
| 内存                              | ADATA-DDR4-2133-8G  *2                                       | 我的条子用了3～4年了，不建议超频，否则可能会造成macOS运行不稳定 |
| 显卡                              | 华硕猛禽R9-380-4G                                            | BIOS中显示输出设置为PCIe Slot（独立显卡），显示器视频连接线一定要直接接在独立显卡的接口上，否则你需要更改opencore的配置文件的相关选项 |
| 硬盘                              | 三星（SAMSUNG） 960EVO  M.2接口(NVMe协议)单面SSD固态硬盘     | 用于Mac OS                                                   |
| 硬盘                              | 闪迪（SanDisk）240GB SSD固态硬盘 SATA3.0接口                 | 用于Windows 10                                               |
| 机箱                              | Sunmilo-T03-A4机箱，支持240水冷，自带PCIEx16延长线，支持双风扇长显卡 | 支持镂空定制和侧板材质                                       |
| 散热器                            | 美商海盗船 (USCORSAIR) H100i PRO RGB冷头 一体式CPU水冷散热器 | 240mm水冷                                                    |
| 电源                              | 美商海盗船 (USCORSAIR) 额定600W SF600                        | SFX电源，如果都是默认频率，没发现带不动，我使用keyshot渲染4K，运行正常，macOS下你也没什么3A大作可玩 |
| 显示器                            | 28英寸，4K（3840x2160）                                      | 使用DP视频连接线                                             |
| 无线网卡+蓝牙                     | BCM943602CS，2.4G：300M，5GAC：867M，蓝牙4.0，NGFF-Key B/M   | macOS免驱动！建议搭配转接卡食用！                            |
| 无线网卡转接卡                    | BCM943602CS模块转m.2-M key（NVME）, 搭配3条内置天线          | 转接卡需要一个USB2.0的9Pin接口以支持蓝牙数据传输             |
| 板载USB 9 Pin一分二转接板（可选） | 这块主板只有一个USB 9pin接口，BCM943602CS的蓝牙需要一个      | 我的水冷头其实也需要一个，用来在Windows下使用iCUE，但这不是必须的，所以我还没加，水冷的基本功能是正常的，只是不能自定义灯光和模式，用的是水冷的预设 |

# 项目特点

- **原生NVRAM**
- **OpenCore 0.5.5**
- **macOS 10.15.3 + Windows10** 双系统（建议准备两块SSD，一块NVME接口的，一块SATA接口的）
- **支持 随航**（有线无线均可），说白了就是BCM943602CS免驱，功能正常
- **支持原生电源管理**（电源5项，点完睡眠按钮要等10秒以上，不要猴急锤键盘中断了进入睡眠的过程）
- **USB定制，支持iPad快速充电**，我把type c接口一块屏蔽了，你需要的话可以重新定制
- **CPU变频正常**，核心显卡能够达到峰值频率且功能正常（否则`随航`功能也无法保证正常工作）
- **板载声卡工作正常**，暂无测试独立显卡的HDMI的音频输出是否正常，因为我使用的是主板的耳机孔
- **推荐 Mos 这个App**，使你的鼠标操作体验更加顺畅
- 用Clover引导我也试过，即使使用`AptioMemoryFix.efi`也能正常工作，甚至内存超频也没问题，geekbench5的跑分与OC相当

# BIOS设置

BIOS版本：F8c

我已经给出了我的BIOS预设文件，将其放置到FAT32格式的U盘里，可通过BIOS界面加载该预设，或者遵循以下规则

*以下配置项的具体路径可以从技嘉官方网站的主板的PDF格式的使用手册中查询*

## 禁用

- Fast Boot
- VT-d(可以设置为enable 如果你将`config.plist`中的`DisableIoMapper` 设置为YES/true)
- CSM
- Software Guard Extensions (SGX)
- Intel Platform Trust
- CFG Lock(MSR 0xE2 write protection)----BIOS里无该选项，需要通过其他方式解锁，自行网上搜索

## 启用

- Above 4G decoding
- Hyper-Threading Technology
- XHCI Hand-off
- OS type: Windows 8.1/10 Mode----不必设置为other OS，那是为老爷板子预备的
- Legacy USB Support

## 重点设置（非常重要，是顺利开启核显的关键）

- **System memory Mutiplier——DDR4-Auto**----`使用OpenCore，内存超频会导致系统不稳定，我2133的内存超频超过2400可能卡在引导界面，或者即使进入了操作系统界面，也会马上死机，clover引导却不会，初步判断是OC与Clover的内存管理差异造成`
- **Initial Display Output----->PCIe 1 Slot**----`设置初始化输出为独立显卡`
- **Internal Graphics---->Enabled**-----`启用核心显卡`
- **DVMT Pre-Allocated---->32M**----`设置核显内存大小，设置为其他值会卡在引导界面提示内存注入方面的错误`
- **DVMT Total Gfx Mem---->256M**----`设置总显存大小，建议维持默认值`

# 额外的设置

你需要在`config.plist`文件中的`gerneric`条目替换你自己的机型信息，包括MLB、UUID、ROM、Serial Number，机型信息的生成工具请自行搜索。

主板的CFG一定要解锁，解锁教程我不再赘述，就是先获取CFG在BIOS中的key值（需要用到你正在使用的BIOS版本固件，主板官网可下载），然后使用`modGRUBShell.efi`改写CFG的Key对应的value值，实现解锁。CFG 解锁状态可以通过hackintool查看。要注意的是每次BIOS固件升级都会重置CFG的状态。

# EFI文件结构及说明

## EFI

- **BOOT**
  - BOOTx64.efi----`必须的引导文件`
- **OC**
  - **OpenCore.efi**----`opencore引导器，核心文件`
  - **ACPI**
    - SSDT-PLUG.aml ----`启用原生电源管理`
    - SSDT-PMC.aml----–`启用原生NVRAM`
  - **Drivers**
    - HFSPlus.efi----`支持HFS文件系统格式的硬盘数据读写`
    - ApfsDriversLoader.efi----`支持APFS文件系统格式的硬盘数据读写`
    - FwRuntimeServices.efi----`核心驱动，完善NVRAM并提供一些内存管理功能`
  - **Kext**
    - Lilu.kext----`必不可少的其他补丁的黄金搭档`
    - VirtualSMC.kext----`仿冒白苹果硬件的补丁`
    - AppleALC-----`声卡补丁`
    - WhateverGreen.kext----`显卡修复补丁`
    - IntelMausiEthernet.kext----`有线网卡驱动`
    - USBPorts.kext----`USB定制补丁，需要与机型匹配，在文件内的plist文件中编辑`
    - USBPower.kext---`USB电源补丁，需要与机型匹配，在文件内的plist文件中编辑`
  - **Tool**
    - CleanNvram.efi----`用于清理Nvram的内容，也可以使用Opencore自带的Nvram重置功能`

## 特别说明

### APCI

1. 没有SSDT-EC？
   - 按照官方说明，该文件是运行Mac OS 10.15必须的，但是技嘉主板（已经禁用了EC）不需要这个玩意儿也可以正常运行
2. 没有SSDT-AWAC？
   - Mac OS使用RTC时钟，目前不兼容AWAC 系统时钟，在主板没有或未开启RTC时钟的情形下使用该文件，而技嘉这块主板恰好默认就是RTC时钟，是不需要这个文件的
3. 没有SSDT-USBX？
   - 我们使用USBPower.kext来实现相同的USB电源功能，可以实现诸如iPad的快速充电
4. 没有SSDT-UIAC ？
   - USB定制可以使用SSDT，也可以通过加载 kext 文件实现，我使用了后者，即USBPorts.kext .
   - 但要注意，不论SSDT还是USBPorts，都是根据自己的硬件配置和设置的苹果机型生成，因此除非你的硬件（主板型号gigabyte z390 i wifi pro）和机型设置（iMac19,1）跟我的完全一致，否则不建议直接使用我的USBPorts.kext文件。

### Kext

1. 没有无线网卡和蓝牙驱动？
   - BCM943602CS这块（网卡+蓝牙）模块在Mac OS下免驱动，因此不需要 BCMFix 一类的修复补丁
2. WhateverGreen.kext能够全频率驱动核显么？
   - 我目前使用的是1.3.6版本的补丁，使用 Intel Power Gadget.app 监测，打开 VideoProc.app 测试，峰值频率正常（1.1GHz以上）

# config.plist自定义选项说明

*我使用 ProperTree 进行编辑，不同的编辑器的 True/False 对应 Yes/No*

## NVRAM

- Add
  - 4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14
    - UIScale----01    `*我的显示器是4K，01确保显示不会被放大而虚化*`
  - 7C436110-AB2A-4BBB-A880-FE41995C9F82
    - boot-args----slide=1    `*内存注入使用连续注入的方式，我去除了-v，因此引导不会显示任何debug信息*`
- LegacyEnable----False    `*因为我们使用SSDT-PMC开启了原生NVRAM，因此选择false*`

## Booter

- Quirks
  - DevirtualiseMmio----False   `*因为实测KASLR方式无法使用，所以内存注入采用连续注入，况且KASLR也并非绝对安全*`
  - DisableVariableWrite----False   `*我们是原生NVRAM*`
  - ProvideCustomSlide----True    `*提供自定义Slide值支持，我的 slide=1*`

## DeviceProperties

- Add
  - PciRoot(0x0)/Pci(0x1f,0x3)    `*板载声卡的设备路径*`
    - layout-id----01000000     `*我的板载声卡为ALC1220，我选择注入声卡驱动id值为1*`
  - PciRoot(0x0)/Pci(0x2,0x0)     `*核心显卡的设备路径*`
    - AAPL,ig-platform-id----0300983E     `*我们显示输出使用独立显卡，核显用作加速*`

## kernel

- Quirks
  - AppleCpuPmCfgLock----False     `*CFG-Lock解锁后设置为False*`
  - AppleXcpmCfgLock----False     `*CFG-Lock解锁后设置为False*`
  - XhciPortLimit----False     `*用于解除USB的15个端口限制，因为我们已经定制了USBPorts，所以不需要启用*`

## Misc

- Boot
  - PollAppleHotKeys----True     `启用苹果原生快捷键，需要与KeySupport=Yes`
  - Resolution----3840x2160     `*我的显示器为4K显示器，因此我设置为4K分辨率*`
  - ShowPicker----True     `*显示Opencore的启动菜单，设置为False可以跳过显示启动项直接进入引导，在macOS中将启动磁盘设置为macOS所在的启动盘，每次开机或重启后就会默认引导macOS系统，这就是原生NVRAM的效果之一，关于NVRAM中存储了哪些信息，可以通过Hackintool查看*`
- Debug
  - Target----0     `*关闭日志记录，还你一个干干净的EFI分区，前提是Bug已经排除完毕*`
- Security
  - AllowNvramReset----True     `*允许在引导选择界面和快捷键按下时重置NVRAM*`
  - AllowSetDefault----True     `*允许使用CTRL+Enter和CTRL+数字锁定默认启动项*`
  - ExposeSensitiveData----True     `*暴露传感器数据，结合监测软件使用，如HWMonotorSMC2*`

## UEFI

- Input
  - KeySupport----True  `*开启OC的内置键盘支持*`

# 参考

Opencore Vanilla Desktop Guide：https://khronokernel-2.gitbook.io/opencore-vanilla-desktop-guide/

#  感谢

核心的开发者们搭建了hackintosh的基本框架，众多的爱好者们进行了各色各样的装修，我们共同维持着这样的开源生态。

对opencore、内核扩展、教程博客的开发者们表示感谢，也对苹果公司表示感谢，虽然这有违版权保护的初衷，但是我们穷，而且希望在这个开源社区里找到更多乐趣。

# Buy Me Coffee

如果你觉得我的项目对你有用，欢迎请我喝杯咖啡，当然，我们都是家境贫寒的伙伴，请随意，嘻嘻～～

|                                                              |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![img](https://raw.githubusercontent.com/helloespresso/basicinfo/master/Alipay.JPG) | ![img](https://raw.githubusercontent.com/helloespresso/basicinfo/master/WeChat.JPG) |
