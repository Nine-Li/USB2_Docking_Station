# USB 2.0 扩展坞 · USB2 Docking Station

> 基于 **CH334U** 的 4 口 USB 2.0 扩展坞：1 路 USB-C 上行、1 路 USB-C 供电输入、4 路 USB-A 下行，带双路供电 ORing、限流保护、端口指示灯和复位按键。
>
> 硬件版本 **V0.3** · 嘉立创 EDA 专业版（`.eprj3` 文件夹化工程）· 2 层板 52.6 × 100 mm

---

## 概览

本工程是一块完整的 USB 2.0 HUB 扩展坞主板。上行口用一颗 16 pin 的 USB-C 母座接电脑，另用一颗 6 pin 的 USB-C 母座做 5 V 供电输入（也可只由上行口总线取电），下行由 CH334U 分出 4 个 USB-A 口。

5 V 输入经肖特基二极管与理想二极管做 ORing 汇入 `VBUS` 系统母线；`VBUS` 一路经 CH217A 限流电源开关供给 4 个下行口（`VBUS1`），另一路经 AMS1117-3.3 得到 `3V3` 供 HUB、上拉电阻和指示灯使用。CH334U 通过 `PWREN#` / `OVCUR#` 控制 CH217A 并读取过流状态，每个下行口都有独立指示灯。

---

## 特性

- **4 口 USB 2.0 HUB**：CH334U 单芯片方案，下行 4 口共享 480 Mbps 带宽
- **USB-C 上行**：16 pin 母座，走 `DPU` / `DMU` 到 HUB 上行端口，母座 5 V 可反灌为本机供电
- **USB-C 供电输入**：6 pin 母座，5 V 独立供电，与上行口双路 ORing
- **防反接 / 双路汇流**：`SS34` 肖特基 + `CH213K` 理想二极管
- **下行限流保护**：`CH217A` 电源开关，限流点由 R12 设定，故障信号回馈 HUB
- **Type-C 受电识别**：两路 CC 均以 5.1 kΩ 下拉（Rd）
- **时钟**：12 MHz 无源晶振 + 22 pF 匹配电容
- **人机交互**：1 个复位按键，5 颗 0603 指示灯（1 颗电源指示 + 4 颗端口状态指示）

---

## 功能框图

**供电链路**

```
  USB-C 6P  (5V IN)  -->  SS34 --+
                                 |
  USB-C 16P (UP 5V)  --> CH213K -+--> VBUS --+--> CH217A --> VBUS1 --> USB-A x4
                                             +--> AMS1117 --> 3V3 --> CH334U
```

**数据与控制**

```
  USB-C 16P --DPU/DMU--> CH334U --+--> USB-A 1  (DP1/DM1)
                                  +--> USB-A 2  (DP2/DM2)
                                  +--> USB-A 3  (DP3/DM3)
                                  +--> USB-A 4  (DP4/DM4)

  CH334U --PWREN#--> CH217A (EN#)        CH217A (FLAG#) --OVCUR#--> CH334U
  SW1 --RESET--> CH334U (RESET#)         X1 12 MHz --XI/XO--> CH334U
```

---

## 接口说明

| 位号 | 连接器 | 作用 | 关键网络 |
|------|--------|------|----------|
| `USB1` | USB-C 6P 母座 | 5 V 供电输入（丝印 USB IN） | `$1N115` → SS34 → `VBUS`；CC 经 5.1 kΩ 下拉 |
| `USB2` | USB-C 16P 母座 | 上行口，接主机 | `DPU` / `DMU` → CH334U；VBUS → CH213K → `VBUS` |
| `U7`–`U10` | USB-A 母座 ×4 | 下行口，接外设 | `VBUS1` 供电，`DP1`–`DP4` / `DM1`–`DM4` |
| `SW1` | 轻触按键 3×4×2 mm | 复位 CH334U（`RESET`） | `RESET` / `GND` |
| `LED1` | LED 0603 | 电源 / 工作指示 | 经 220 Ω 限流电阻接 `3V3` |
| `Port1`–`Port4` | LED 0603 ×4 | 各下行口状态指示 | 由 CH334U 的 LED 引脚经 220 Ω 限流驱动 |

> 两路 Type-C 的 CC1 / CC2 分别用 5.1 kΩ 下拉，对电源适配器宣告为 5 V 受电端。

---

## 电源与网络

| 网络 | 来源 | 去向 |
|------|------|------|
| `5V` | USB-C 上行口 VBUS | CH213K 理想二极管输入端 |
| `$1N115` | USB-C 6P 供电口 VBUS | SS34 肖特基输入端 |
| `VBUS` | 上述两路 ORing 后的系统 5 V 母线 | CH217A、AMS1117、330 µF 电解 |
| `VBUS1` | CH217A 限流开关输出 | 4 路 USB-A、330 µF 电解 |
| `3V3` | AMS1117-3.3 | CH334U、`OVCUR#` 上拉、LED 限流电阻 |

- 供电能力以 CH217A 的限流设定为准，丝印标注 **MAX 5V1A**。
- 上行口与独立供电口可单独或同时接入，二极管 ORing 保证两路互不倒灌。

---

## 主要器件

| 位号 | 型号 | 封装 | 说明 |
|------|------|------|------|
| `U1` | CH334U | QSOP-28 | USB 2.0 4 口 HUB |
| `U2` | AMS1117-3.3 | SOT-223 | 3.3 V LDO |
| `U3` | CH213K | SOT-23-3 | 理想二极管 / ORing |
| `U5` | CH217A | SOT-23-6 | 限流电源开关 |
| `USB2` | TYPEC-16P | USB-C SMD | 上行 Type-C 母座 |
| `USB1` | TYPE-C 6P | USB-C SMD | 供电 Type-C 母座 |
| `U7`–`U10` | USB-A 母座 | USB SMD | 下行端口 ×4 |
| `X1` | X322512MSB4SI | 3225 4 脚 | 12 MHz 无源晶振 |
| `D2`、`D3` | SS34 | SMA | 肖特基二极管 |
| `U4`、`U12` | 330 µF / 50 V | 10 × 10.3 mm | 电解电容（原理图沿用 U 位号） |
| `SW1` | 轻触按键 | 3 × 4 × 2 mm | 复位 |
| `LED1`、`Port1`–`Port4` | LED 0603 | 0603 | 指示灯 ×5 |
| `C2`、`C4`–`C9`、`C11`–`C15` | 10 µF / 0.1 µF / 22 pF | 0603 | 去耦、晶振匹配 |
| `R1`–`R4`、`R6`、`R8`、`R10`–`R12` | 5.1 kΩ / 220 Ω / 56 kΩ | 0603 | CC 下拉、LED 限流、限流设定 |

完整 BOM 以 EDA 工程为准，共 42 个元件位号。

---

## PCB 规格

| 项目 | 参数 |
|------|------|
| 层数 | 2 层（Top / Bottom） |
| 板厚 | 1.6 mm（FR4，外层铜 1 oz） |
| 尺寸 | 52.63 × 100.00 mm（2072 × 3937 mil） |
| 外形 | 圆角矩形，R = 3 mm |
| 安装孔 | Ø2.5 mm ×4，四角，孔距 44.63 × 92.00 mm（金属化） |
| 其他非金属化孔 | 0.65 mm ×2、1.3 mm ×8（连接器定位柱） |
| 丝印 | 端口名 `Port1`–`Port4`、`USB IN`、`DC IN`、`Active`、`MAX 5V1A`、`V0.3` |

---

## 工程结构

嘉立创 EDA 专业版使用文件夹化工程格式（`.eprj3`）；原理图、PCB、面板分别以文本 / JSON 记录的形式存放在子目录中，便于 Git 管理。

```
USB2_Docking_Station/
├── USB2_Docking_Station.eprj3       # 工程索引与元数据
├── sch/主板/                        # 原理图（A4，1 页）
│   ├── P1.esch2                     # 图页源文件
│   └── 主板.ecfg                    # 该原理图的设计规则与配置
├── pcb/PCB1.epcb2                   # PCB 源文件
├── panel/Panel1.epan2               # 亚克力面板（仅画布 / 工艺参数）
├── 0819f05c4eef4c71ace90d822a990e87 # 编辑器生成的工程库（SQLite，当前无器件记录）
└── README.md
```

工程信息：

| 项目 | 内容 |
|------|------|
| 工程名 | USB2_Docking_Station |
| 版本 | V0.3 |
| 图幅 / 页数 | A4 / 1 页 |
| 单位 | Nine-Li |
| 最后更新 | 2026-10-28 |

---

## 使用方法

### 打开工程

1. 用嘉立创 EDA 专业版（建议 4.1.78 及以上）打开 `USB2_Docking_Station.eprj3`（选择文件夹工程）。
2. 原理图位于 `sch/主板`，PCB 为 `pcb/PCB1.epcb2`，面板为 `panel/Panel1.epan2`。

---

## 变更记录

| 提交 | 内容 |
|------|------|
| `357fb1d` | USB IN 更换为 Type-C 接口，同时优化器件布局 |
| `50f885f` | 调整部分地过孔 |
| `743e0a3` | 晶振底部增加禁止铺铜区 |
| `f2763c7` | 完善丝印，优化部分设计 |
| `86ed5ec` | 修正已知封装错误，指示灯由九灯改为五灯 |
| `71553d7` | 完成 PCB 设计 |
| `f432cc0` | 完善原理图，完成 PCB 元件摆放 |
| `fc39198` | 完成电源、晶振、复位等核心电路 |
| `a7b0cf5` | 创建仓库 |

当前版本 **V0.3**（2026-10-28），原理图、PCB 与 Gerber 同版。

---

## 说明与限制

- 仅支持 USB 2.0（480 Mbps），4 个下行口共享同一段上行带宽。
- 下行总电流受 CH217A 限流约束，丝印标注 MAX 5V1A，不宜带大功率外设。
- 下行端口未放置独立 ESD 器件，静电防护依赖 CH334U 内置能力。
- `panel/Panel1.epan2` 只有参数配置，尚无实际拼板图形。
- `USB2_Docking_Station_backup/` 为编辑器自动快照，未纳入版本控制；`0819f05c…` 为编辑器生成的工程库文件，当前无器件记录，可忽略。

---

## English Overview

A 4-port USB 2.0 docking station board built around the **CH334U** hub controller. It provides one USB-C 16-pin upstream port, one USB-C 6-pin 5 V power input (either source may power the board), and four USB-A downstream ports.

Power is ORed from both inputs through an SS34 Schottky and a CH213K ideal diode into the `VBUS` rail; a CH217A current-limited switch feeds the downstream ports while an AMS1117-3.3 supplies 3.3 V to the hub. A 12 MHz crystal clocks the CH334U, which controls the switch and reads back overcurrent status via `PWREN#` / `OVCUR#`. Five 0603 LEDs indicate power and per-port status, and a tact switch resets the hub.

Board: 2 layers, 1.6 mm FR4, 52.63 × 100 mm, four Ø2.5 mm mounting holes. Design files are in EasyEDA Pro folder-project format (`.eprj3`); fabrication data is the V0.3 Gerber package.
