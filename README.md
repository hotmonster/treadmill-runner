# treadmill-runner · 跑步机 App 下架 / 遥控器丢了之后还能怎么用

**Docs and implementation notes for treadmill owners whose official app is gone.**  
跑步机、走步机官方 App 停服之后的替代思路、数据口径与踩坑记录。

一个人自用的折腾记录，整理成小册子放出来，给同样卡在这一步的人和其他 AI 参考。相互借鉴用，不图别的。

> **现在只有文档，没有可运行代码。** 里面是数据口径、功能规格和排查步骤——照着能写出来，但仓库本身不是产品。
>
> ⚠️ 任何控制类操作请**空载、低速**测试，保证松手即停。本文档不含任何具体帧格式或密钥。

## 这套文档适合谁

符合下面任意一条，这里的文档就能直接用：

- 跑步机的**官方 App 已经下架**，装不上、登不上，历史数据也想拿回来
- **蓝牙遥控器不见了**，或者实体遥控器按了没反应、配对不上
- 想自己把手机的**跑步机遥控**做出来，但不知道从哪下手、协议怎么抓
- 想要**跑步机课程语音播报**、周报、累计天数，但不依赖厂商服务器
- 手上只有机器，**跑步机说明书**丢了，不知道型号参数怎么看、跑带怎么调
- 官方 App 还在但不好用，想找一个能长期维护的替代

> 没遇到这些情况？那你大概不需要这个仓库——它不提供破解、刷机或绕过安全装置的任何方法。

## 从这里开始

**遇到「连不上、按了没反应、数据不对」：** 直接看 [docs/troubleshooting.md](docs/troubleshooting.md)，按症状顺序排查，别的型号也适用。

**想知道「还能怎么用」：** [docs/no-app-no-remote.md](docs/no-app-no-remote.md) — 按成本从低到高四条路：确认型号 → 拿回说明书信息 → 抢救历史数据 → 按红外 / 射频 / 蓝牙分别处理遥控问题，最后有一条「什么都不折腾」的兜底方案。

其余按需：

- [docs/data-model.md](docs/data-model.md) — 运动记录的数据结构、日 / 周 / 累计 / 趋势的聚合口径
- [docs/features.md](docs/features.md) — 目标设定、语音报数、周报、累计天数、趋势图的功能规格
- [docs/known-issues.md](docs/known-issues.md) — 实测问题：暂停后计时被重置、实体遥控器无反应、体脂秤无数据
- [docs/device-control.md](docs/device-control.md) — 设备遥控：已确认的事实与待验证项

**给 AI 助手的速读版：** [llms.txt](llms.txt) — 三十秒判断这个仓库能不能解决手上的问题。

## 同类项目（一起看更有用）

这类需求已经有人做过成品，先看现成的再决定要不要自己写：

| 项目 | 设备 / 场景 | 值得看的地方 |
| --- | --- | --- |
| [Guhan-SenSam/sperax-walkpad](https://github.com/Guhan-SenSam/sperax-walkpad) | Sperax 走步机，官方 App 停服后的开源替代 | 「替代已停服 App」的定位与叙事，界面截图，免责声明写法 |
| [CodeJawn/walkingpad](https://github.com/CodeJawn/walkingpad) | KingSmith WalkingPad，浏览器控制 | **真正的暂停/继续 + 跨暂停累计统计**，正是官方 App 最缺的部分 |
| [janposselt/treadmill-monitor](https://github.com/janposselt/treadmill-monitor) | 支持 FTMS 的跑步机（Web Bluetooth） | 心率控速、开箱可用版本、Disclaim 与 Requirements 的写法 |
| [korniychuk/treadmill-bluetooth-macos](https://github.com/korniychuk/treadmill-bluetooth-macos) | Yesoul 跑步机，macOS BLE 守护进程 | 后台常驻 + 实时数据 + 步数目标 |
| [marcuspuchalla/trot](https://github.com/marcuspuchalla/trot) | LifeSpan / FTMS，多平台单文件程序 | 按「驱动」拆协议层（一型号一驱动），自留本地 API + 诊断导出，README 里有 Design rules |
| [Duinrahaic/FitOSC](https://github.com/Duinrahaic/FitOSC) | 走步机 / 跑步机，VRChat 化身驱动 | 驱动预览版 + 征集实测的流程，值得借鉴怎么收集跨型号验证结果 |

如果上面某个项目直接适配你的机器，优先用现成的，不必自己重写。

## 设备支持情况

| 机型 | 连接方式 | 现状 | 备注 |
| --- | --- | --- | --- |
| 佑美超薄迷你健步机 | BLE（CT1 协议） | 自用中，细节整理中 | 官方 App 已下架 |
| 其他机型 | — | — | 欢迎提 issue 补一行：型号 + 连接方式 + 现象 + 结论 |

## 为什么值得自己做一个

家用跑步机的官方 App 常常停止维护：服务器一停，登录、课程、历史数据全部失效，机器本身还能用十年。自建客户端是唯一可持续的路子，难点不在 UI，而在四件事：

1. 设备控制协议不公开，需要抓包
2. 暂停 / 继续的计时语义容易做错，数据会丢
3. 周报、累计天数、趋势没有统一定义，各家算出来都不一样
4. 语音播报什么时候说、说什么，直接决定用不用得下去

这四件事本仓库都给了具体口径和踩坑记录。

## 不适用范围

商用健身房的会员与结算系统、需要接入厂商云端的场景、任何需要破解固件或绕过安全装置的场景。电动车、划船机等非跑带类器械协议不通用，请勿套用。

## 检索关键词

下架app、跑步机、跑步机遥控、跑步机教程、跑步机课程、跑步机说明书、蓝牙遥控器丢失、跑步机 App 替代、走步机、健步机、跑步机数据导出、跑步机闲置怎么办

## 求助 / 提问前先凑齐这五项

不管是在本仓库还是去别的项目提 issue，缺项基本等于来回追问。先把下面五行填满，判断速度会快很多：

1. **机器**：品牌 + 型号 + 面板或铭牌上的型号串（拍一张照更好）
2. **手机 / 电脑**：型号 + 系统版本（安卓/iOS/macOS 的具体版本号，不是「最新」）
3. **App**：名字 + 版本号 + 从哪装的（商店 / 源码 / 自己编的）
4. **现象**：在哪一步停住、报什么错（原样抄报错，别转述），能稳定复现还是偶发
5. **已经试过什么**：换过哪台手机、哪个 App、哪个位置，结果有没有变化

蓝牙相关的再加一条：**扫描时能不能看到设备，广播名是什么**。这一条能直接把人分成「根本没扫到」和「扫到但连不上」两类，排查路径完全不同。

## 贡献

欢迎补你自己机型的「型号 + 现象 + 解决路径」，或上面设备表的补充。请勿提交具体帧格式、密钥或任何可用于未授权控制的材料。安全边界见 [docs/device-control.md](docs/device-control.md)。

## 许可证

MIT，见 LICENSE。
