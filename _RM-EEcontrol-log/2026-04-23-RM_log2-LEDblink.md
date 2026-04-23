# STM32 Learning Notes 01  
> LED Blink | 我的第一个点灯程序

## 📝 Overview | 为啥学这些  
在正式进入 RM 电控组考核的电机 PID 控制之前，我必须先打通整个嵌入式开发流程——从软件安装、工程配置、代码编写到程序烧录。点灯，这个嵌入式世界的“Hello World”，就是我迈出的第一步。

- **智视寻迹（商场室内定位）**：虽然这个项目目前还在软件层面（YOLO + OpenCV），但未来一定会涉及嵌入式部署。STM32 是嵌入式视觉的入门必修课。
- **RM 电控组考核**：这是最直接的驱动力。考核要求 PID 控制电机，而点灯是验证整个工具链是否打通的唯一标准。

说实话，我本以为点灯是“有手就行”的操作，结果从安装软件到看见 LED 闪烁，我整整踩了3小时的坑。各种玄学报错、驱动问题、接线迷惑、编译器版本不兼容……一度让我怀疑人生。

**开搞！！Let's do it!!**

---

## 🎯 Learning Goals! | 学习目标  
- 完成 STM32 开发环境搭建：Java、STM32CubeMX、Keil MDK、芯片支持包  
- 理解 STM32 最小系统板的供电与 SWD 调试接口  
- 学会用 CubeMX 配置 GPIO 输出并生成 Keil 工程  
- 掌握 Keil 工程的编译、下载流程  
- 成功点亮 STM32F103C8T6 核心板上的 PC13 LED  
- 积累嵌入式开发中常见“玄学”问题的排查经验  

---

## 📖 What have I learned? | 学习内容  

### Part 1: Development Environment Setup | 开发环境搭建  

#### 1.1 软件三件套：Java + CubeMX + Keil  

**Java 环境**  
STM32CubeMX 是基于 Java 开发的，所以必须先装 Java。去官网下载 JDK 8，一路默认安装，最后在 CMD 里输入 `java -version` 验证成功。

**STM32CubeMX 安装**  
ST 官网的下载流程极其反人类，要注册账号、验证邮箱、还经常收不到邮件。最后我走“游客下载”，填了个邮箱才拿到链接。安装路径**绝对不能有中文**，否则后面会出各种莫名其妙的错。

**Keil MDK 安装**  
官网下载 MDK-ARM，以管理员身份运行，路径无中文。装完后需要安装 STM32F1 系列的芯片支持包（Pack）。

**项目应用场景**：这套环境是后续所有 STM32 开发的基础。无论是点灯、电机控制还是视觉处理，都得从这里开始。

#### 1.2 Keil 激活——最折磨人的环节  

我先后尝试了：
- **联网激活社区版**：反复报错 `Error processing response from license server`，关防火墙、改 DNS、管理员运行全试了，无效。
- **PSN 在线激活**：官网登录后拿到 PSN，但 Keil 里的激活向导依然失败。
- **注册机离线激活（最终方案）**：关闭 Windows Defender 实时保护，以管理员身份运行注册机，填入 CID，Target 选 ARM，生成 LIC 码，Add 到 Keil。看到 `LIC Added Successfully` 时，我差点哭出来。

**项目应用场景**：Keil 激活是每个 STM32 新手的“成人礼”。过了这关，后面编译下载的限制就解除了。

#### 1.3 硬件认知：开发板 vs ST-Link  

| 物品 | 外观 | 作用 |
| :--- | :--- | :--- |
| STM32 核心板 | 绿色小板，上有 STM32F103C8T6 芯片 | 执行程序的大脑 |
| ST-Link 下载器 | U 盘形状，上有 10 针或竖排接口 | 把程序从电脑烧进芯片 |

**接线（SWD 方式）**：
- ST-Link `3.3V` → 核心板 `3.3V`
- ST-Link `GND` → 核心板 `GND`
- ST-Link `SWDIO` → 核心板 `PA13`
- ST-Link `SWCLK` → 核心板 `PA14`

我用的 ST-Link 是竖排引脚，丝印直接标了 `SWDIO`、`SWCLK`、`3.3V`、`GND`，接起来很方便。接好后插上 USB，核心板红灯亮，说明供电正常。

一开始忘了配置ST-link，程序直接没跑动。抓耳挠腮半天才想起来。。。。

---

### Part 2: First Project with CubeMX & Keil | 第一个工程  

#### 2.1 CubeMX 配置  

1. 新建工程，选 `STM32F103C8Tx`。  
2. 在 `Pinout` 界面找到 `PC13`，左键点击设为 `GPIO_Output`。  
3. `System Core` → `SYS` → `Debug` 选 `Serial Wire`（**这一步漏了会导致芯片锁死**）。  
4. `Project Manager` 里：  
   - Project Name: `test`  
   - Project Location: `D:\STM32`（无中文）  
   - Toolchain/IDE: `MDK-ARM`  
5. 点击 `GENERATE CODE`。

#### 2.2 Keil 工程配置与代码编写  

用 Keil 打开生成的 `.uvprojx` 文件。  
在左侧 `Project` 窗口展开 `Application/User`，双击 `main.c`。  

在 `while(1)` 循环里找到 `USER CODE BEGIN 3` 和 `USER CODE END 3`，写入：

```c
/* USER CODE BEGIN 3 */
HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_13);  // 翻转 PC13 电平
HAL_Delay(500);                          // 延时 500ms
/* USER CODE END 3 */
```

**关键**：代码必须写在 USER CODE 之间，否则下次 CubeMX 重新生成时会被覆盖。

#### 2.3 编译与下载

- 按 F7 编译。如果报错 `missing: compiler version 5`，是因为 Keil 新版默认不带 AC5 编译器。需要手动下载 AC5 安装包，装到 Keil_v5\ARM\ARMCC 路径下，然后在工程选项里切换编译器版本。
- 编译通过后，按 F8 下载。
- 下载前务必在 **Options for Target → Debug** 里选择 `ST-Link Debugger`，并确认 Settings 里 Port 为 **SW**，**SW Device** 框能识别到芯片。（这里电脑犯病很久，给我气够呛，配置有点拉。最后用了**按住reset点F8，看到开始运行立刻松手**的邪修办法才成功。）

**项目应用场景**：这套 **CubeMX → Keil → 下载**的流程，就是后续电机控制开发的固定动作。

---

### Part 3: Debugging & Troubleshooting | 调试与排错

#### 3.1 下载报错 No target connected

**原因**：ST-Link 与芯片通信失败。
**解决**：

1. 检查接线是否松动。
2. 按一下核心板的 **RESET** 键再试。
3. 按住 RESET 不放，点击下载，看到进度条走动时松手（祖传秘方，成功率极高）。
4. 确认 CubeMX 里 SYS → Debug 为 **Serial Wire**。

#### 3.2 下载报错 `Flash Download failed - Could not load file .axf`

**原因**：工程输出路径问题或编译未成功。
**解决**：

1. 先按 F7 编译，确保 0 Error(s)。
2. 在 `Options for Target` → Output 里勾选 **Create HEX File**，并设置输出路径为无中文。
3. 检查 `Flash Download` 选项卡里是否有 `STM32F10x Med-density Flash` 算法。

#### 3.3 下载成功但 LED 不闪

**现象**：程序烧进去了，PC13 的 LED 常亮或不亮。
**排查**：

- 确认代码写对了位置。
- 把 `HAL_Delay(500)` 改成 `HAL_Delay(2000)`，观察闪烁节奏是否变化。如果没变化，说明新程序没烧进去。
- 执行 Project → Clean Targets，再重新编译下载。

最终发现：我之前改代码后没保存，Keil 编译的一直是旧文件……保存后重新编译下载，LED 终于开始有节奏地闪烁。

**项目应用场景**：这些坑在后续电机控制中会反复遇到，提前积累经验能少走很多弯路。

---

## 💡 Some Problems I Met... | 踩到的坑。。。

**Problem 1**: ST 官网下载 CubeMX 邮件验证失败
- What's wrong? 点了“Download as guest”后收到邮件，但点击邮件里的链接提示邮箱未验证。
- Reason ST 官网的验证系统有时会出 Bug，即使在同一浏览器打开链接也识别不到。
- Solution 改用中国区官网 `www.stmcu.com.cn` 下载，或从网盘下载稳定版本（如 v6.13.0）。

**Problem 2**: Keil 社区版联网激活反复报错
- What's wrong? 输入 `https://mdk-preview.keil.arm.com` 后报错 `Error processing response from license server`。
- Reason 国内网络连接 Keil 海外服务器不稳定，防火墙、DNS 都可能干扰。
- Solution 最终改用注册机离线激活。关闭 Windows Defender 实时保护，以管理员身份运行注册机，填入 CID，Target 选 ARM，生成 LIC 码添加到 Keil。

**Problem 3**: Pack Installer 在线下载芯片支持包超时
- What's wrong? 在 Pack Installer 里点击 Install 后，出现 `HTTP get timed out` 报错。
- Reason Keil 服务器在国外，国内直连不稳定。
- Solution 从官网 `www.keil.com/dd2/Pack/` 手动下载 **.pack** 文件，双击离线安装；或从网盘下载已解压的文件夹，直接复制到 Keil_v5\Packs 目录。

**Problem 4**: Keil 编译报错 missing: compiler version 5
- What's wrong? 打开工程按 F7 编译，提示缺少 ARM Compiler 5。
- Reason 新版 Keil MDK（5.37 及以上）默认不再自带 AC5 编译器。
- Solution 手动下载 ARM Compiler 5.06 安装包，安装路径选在 Keil 目录下的 ARM\ARMCC。然后在工程选项 Target 选项卡里切换编译器为 **Use default compiler version 5**。

**Problem 5**: 下载成功但 LED 闪烁频率不变
- What's wrong? 修改了 `HAL_Delay(2000)`，但 LED 依然按原来的 500ms 节奏闪烁。
- Reason Keil 没有重新编译修改后的 main.c，下载的是旧的 .axf 文件。
- Solution 先确认代码已保存（Ctrl+S），然后执行 Project → Clean Targets 清理中间文件，再重新按 F7 编译、F8 下载。必要时可删除工程目录下的 .uvoptx 和 .axf 文件。

**Problem 6**: 找不到 PA13/PA14 引脚，不会接杜邦线（。。。有点不好意思。体谅一下哈刚开始学习，新手村都没出）
- What's wrong? 核心板上丝印没有 PA13/PA14，也不清楚杜邦线公母区别。
- Reason 部分核心板将调试引脚标记为 SWDIO、SWCLK 而非 PA13/PA14；公母头概念对新手不直观。
- Solution 母对母杜邦线两端都是孔，用于连接排针。找核心板上标有 SWDIO、SWCLK、3.3V、GND 的独立 4 针接口，或看背面丝印。ST-Link 上的竖排引脚也有对应丝印，直接按功能名称对接即可。

---

## 💡 Learning Insights | 学习心得

### Key Takeaways | 关键收获

- **环境搭建是嵌入式开发的第一道坎**：Java、CubeMX、Keil、Pack、驱动、激活……任何一个环节出错都会卡住。耐心和搜索引擎是最好的老师。
- **CubeMX 的 SYS → Debug 必须选 Serial Wire**：漏了这一步，芯片下载一次后就“锁死”，只能靠 RESET 大法救回。
- **Keil 编译器版本问题很常见**：新版 MDK 默认不带 AC5，手动装一下就行。
- **下载失败的“祖传秘方”**：按住 RESET 再点下载，看到进度条松手。这招解决了我 80% 的连接问题。太爽了这个。
- **点灯虽小，流程俱全**：从硬件接线、CubeMX 配置、代码编写、编译下载到调试排错，整个嵌入式开发的标准流程都在里面了。

### Applications and Inspirations to MY Projects | 项目应用与启发

**RM 电控组考核**

- 点灯打通了工具链，后续电机控制的 PWM、编码器、PID 算法，无非是在这个流程上叠加外设配置和算法逻辑。
- 调试中积累的“复位大法”、“Clean Targets”等经验，在电机调试中同样适用。

**智视寻迹（商场室内定位）**

- 未来若将 YOLO 部署到嵌入式平台（如 STM32 + OV2640 摄像头），这套开发流程就是基础。
- 理解了 GPIO 控制 LED 的原理，也就理解了如何用 GPIO 控制其他外设（如灯光、蜂鸣器、继电器）。

### Looking Ahead | 未来方向铺垫

| 技术点 | 当前实现（点灯） | 嵌入式/电机控制延伸 |
|------|----------------|-------------------------|
| GPIO 输出 | `HAL_GPIO_TogglePin` | 方向控制引脚（AIN1/AIN2） |
| 延时函数 | `HAL_Delay` | 定时器中断实现非阻塞延时 |
| PWM 输出 | 未涉及 | 控制电机转速的核心信号 |
| 编码器模式 | 未涉及 | 读取电机实时转速 |
| PID 算法 | 未涉及 | 闭环速度控制 |
| 串口通信 | 未涉及 | 打印调试信息、上位机交互 |

这些知识是未来转向**电机控制、机器人感知、嵌入式视觉**时必须提前储备的底层基础。现在用点灯把流程跑通，将来叠加复杂功能时会理解得更透彻。**Come on man! Keep going!!!**

---

## 📊 Summary! | 近日成果

| Knowledge Point | 在 RM 考核项目中的应用 |
|---------------|----------------------------|
| Java 环境安装 | CubeMX 运行前提 |
| STM32CubeMX 安装与配置 | 图形化生成 Keil 工程 |
| Keil MDK 安装与激活 | 代码编译、下载的核心工具 |
| STM32F1 Pack 安装 | 让 Keil 识别 STM32F103C8T6 |
| SWD 接线 | 程序烧录的物理通道 |
| GPIO 输出配置 | 控制 LED、电机方向引脚的基础 |
| HAL 库点灯代码 | 嵌入式“Hello World” |
| 编译、下载流程 | 每次修改代码后的固定操作 |
| 复位大法、Clean Targets | 解决 80% 下载失败问题的万能钥匙 |

---

## 📚 CODE Repository | 代码仓库

```c
/* main.c 中的点灯核心代码 */
while (1)
{
    /* USER CODE BEGIN 3 */
    HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_13);  // 翻转 PC13 电平
    HAL_Delay(500);                          // 延时 500ms
    /* USER CODE END 3 */
}
```

---

## 🔗 Some Resources | 相关资源

- [STM32CubeMX 官方下载](https://www.st.com/en/development-tools/stm32cubemx.html)
- [Keil MDK 官方下载](https://www.keil.arm.com/mdk-community/)
- [ST-Link 驱动下载](https://www.st.com.cn/zh/development-tools/stsw-link009.html)
- [B站 中科大RM电控教程](https://space.bilibili.com/337732684)
- [江协科技 STM32 入门教程](https://b23.tv/czHIfdu)

---

> Uptime: +1 Days
>
> Status: LED is blinking... Finally!!!

---

*Learning Date: 2026-04-23*

*Current Projects: RM 电控组考核（PID 电机控制） | 智视寻迹 | SuperAgent*

*Future Direction: 嵌入式视觉 / 机器人系统 / 智能车*

*Log: STM32-Learning-01*

---
