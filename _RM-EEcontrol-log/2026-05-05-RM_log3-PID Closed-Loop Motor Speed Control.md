# 03 - STM32 PID Closed-Loop Motor Speed Control Log | PID闭环电机速度控制
> 来到了经典算法--PID！P-proportional I-integral D-derivative

## 📝 Overview | 为啥学这些  
点灯和环境搭建都搞定了，现在正式进入 RM 电控组考核的核心任务——**PID 控制直流减速电机在固定速度**。

- **RM 电控组考核**：这是最直接的驱动力。考核要求用 PID 算法让电机稳在目标转速，编码器反馈、PWM 输出、PID 计算三者形成闭环。
- **智视寻迹（商场室内定位）**：虽然目前还在软件层面，但 PID 是机器人控制的基石。理解了电机速度闭环，将来做云台控制、小车底盘控制就一通百通。

说实话，在这个过程我差点命都没了。从接线到调参，踩了无数坑。电机一抖一抖转、编码器读数乱跳、VOFA+ 数据不出来、Keil 编译器版本报错……每次以为快解决了，又冒出新的问题。但最终看到电机平稳转动的那一刻，一切都值了。

**开搞！！Let's do it!!**

---

## 🎯 Learning Goals! | 学习目标  
- 掌握 STM32 + TB6612 驱动直流减速电机的完整硬件接线  
- 学会 TIM2 编码器模式配置与 M 法测速公式  
- 理解增量式 PID 算法原理与代码实现  
- 掌握 PID 参数整定方法（抖降 Kp，慢加 Ki）  
- 学会用 VOFA+ 串口工具可视化速度曲线  
- 积累嵌入式开发中编码器反馈异常、电机抖动等常见问题的排查经验  

---

## 📖 What have I learned? | 学习内容  

### Part 1: Hardware Setup | 硬件搭建  

#### 1.1 模块清单与功能

| 模块 | 功能 | 关键接线 |
| :--- | :--- | :--- |
| **STM32F103C8T6** | 系统大脑，运行 PID 代码 | — |
| **TB6612 电机驱动** | 电力搬运工，放大信号驱动电机 | VM=12V, VCC=5V, STBY=3.3V |
| **JGB37-520 编码电机** | 执行转动 + 反馈转速 | 红线→AO1, 白线→AO2 |
| **编码器（电机尾部）** | 测速传感器 | 黄→PA0, 绿→PA1, 蓝→5V, 黑→GND |
| **12V 铁盒开关电源** | 给电机提供动力电 | +V→VM, -V→GND |
| **ST-Link** | 程序下载与调试 | SWDIO→PA13, SWCLK→PA14 |
| **CH340 串口模块** | 速度数据传到电脑 | TX→PA10, RX→PA9 |

#### 1.2 核心接线表

**STM32 ↔ TB6612**
```

PA6 → PWMA（PWM调速）
PA4 → AIN1（方向控制）
PA5 → AIN2（方向控制）
3.3V → STBY（使能，必须接！）
5V   → VCC（芯片供电）
GND  → GND

```

**TB6612 ↔ 电机 & 12V电源**
```

VM  → 12V电源 +V
GND → 12V电源 -V
AO1 → 电机红线（拧端子）
AO2 → 电机白线（拧端子）

```

**编码器 ↔ STM32**
```

蓝线（细）→ 5V
黑线（细）→ GND
黄线（细）→ PA0（A相）
绿线（细）→ PA1（B相）

```

**项目应用场景**：这套硬件连接是电机闭环控制的基础。PID 控制、编码器反馈、PWM 输出都依赖这些接线正确。

---

### Part 2: CubeMX Configuration | CubeMX 配置  

#### 2.1 定时器配置

| 外设 | 模式 | 关键参数 | 引脚 |
| :--- | :--- | :--- | :--- |
| **TIM3 CH1** | PWM Generation CH1 | Prescaler=71, Counter Period=999 | PA6 |
| **TIM2** | Encoder Mode TI1 and TI2 | Prescaler=0, Period=65535 | PA0, PA1 |
| **TIM1** | Internal Clock + Update Interrupt | Prescaler=7199, Period=99 | 无外部引脚 |
| **PA4, PA5** | GPIO_Output | — | 方向控制 |
| **SYS Debug** | Serial Wire | — | PA13, PA14 |
| **USART1** | Asynchronous | BaudRate=115200 | PA9(TX), PA10(RX) |

#### 2.2 关键参数计算

**PWM 频率**：
```

72MHz / (71+1) / (999+1) = 72MHz / 72 / 1000 = 1kHz

```

**PID 中断周期**：
```

72MHz / (7199+1) / (99+1) = 72MHz / 7200 / 100 = 100Hz = 10ms

```

**每转脉冲数**：
```

11（编码器线数） × 30（减速比） × 4（四倍频） = 1320

```

---

### Part 3: PID Algorithm Implementation | PID 算法实现  

#### 3.1 增量式 PID 公式

```

Δu = Kp × [e(k) - e(k-1)] + Ki × e(k) + Kd × [e(k) - 2×e(k-1) + e(k-2)]
本次输出 = 上次输出 + Δu

```

**为什么选增量式**：
- 输出平稳无突变，电机不会突然猛转
- 系统故障时影响小，不会积分饱和
- 更适合电机速度环控制

#### 3.2 M 法测速公式

```

RPM = (脉冲增量 × 60) / (每转脉冲数 × 采样周期秒数)
= (deltaCount × 60) / (1320 × 0.01)

```

#### 3.3 闭环控制流程

```

编码器读数 → 脉冲增量(deltaCount) → 实际速度(RPM)
实际速度(RPM) → 和目标速度比较 → 误差(e)
误差(e) → PID算法 → PID增量(Δu)
PID增量(Δu) → 累加到上次PWM值 → 新的PWM值(pidOutput)
新的PWM值 → `__HAL_TIM_SET_COMPARE` → TB6612 → 电机

```

#### 3.4 核心代码结构

**PID 结构体**：
```c
typedef struct {
    float Kp, Ki, Kd;
    float targetVal;           // 目标速度
    float err, errLast, errPre; // 三次误差（一次现在的，两次以前的），用于计算增量，增量法PID确实大大提高了稳定性
    float outputVal;            // 输出PWM值
} PID_TypeDef;
```

中断回调函数（每10ms执行一次）：

```c
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim) {
    if (htim->Instance == TIM1) {
        // 1. 设置方向
        HAL_GPIO_WritePin(GPIOA, GPIO_PIN_4, GPIO_PIN_SET);
        HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET);
        
        // 2. 读编码器增量
        static int16_t lastCount = 0;
        int16_t now = __HAL_TIM_GET_COUNTER(&htim2);
        int16_t delta = now - lastCount;
        lastCount = now;
        
        // 3. M法算转速
        actualSpeed = (float)(delta * 6000) / (1320.0f * 10.0f);
        
        // 4. PID计算
        float pidOutput = PID_Compute(&motorPid, actualSpeed);
        
        // 5. 更新PWM
        __HAL_TIM_SET_COMPARE(&htim3, TIM_CHANNEL_1, (uint16_t)pidOutput);
    }
}
```

---

### Part 4: PID Parameter Tuning | PID 参数整定

#### 4.1 调参口诀

**抖了就降 Kp，慢了就加 Ki。啸叫立刻断电，Kp 先降一半再说。**

#### 4.2 调参三步法

| 步骤 | 操作 | 目标 |
|-----|-----|-----|
| 1 | Ki=0, Kd=0, 只调 Kp | 找到不抖的 Kp 起点 |
| 2 | 加 Ki（每次 0.02） | 消除稳态误差 |
| 3 | 微调 Kp 和 Ki | 找到最佳组合 |

#### 4.3 参数范围参考

| 参数 | 作用 | 参考范围 | 调试经验 |
|-----|-----|-----------|-----------|
| Kp | 响应快慢，主力 | 0.02 ~ 0.5 | 太小：电机慢悠悠。太大：剧烈抖动、啸叫 |
| Ki | 消除稳态误差 | 0.0 ~ 0.1 | 太小：仍有误差。太大：超调巨大 |
| Kd | 抑制震荡 | 0.0 ~ 0.05 | 对噪声敏感，速度环通常不用 |

**项目成果展示：**
- 电机及其他模块。还不会合理布线的结果。。。一团乱麻
<img width="2376" height="1080" alt="微信图片_20260507001940_755_2" src="https://github.com/user-attachments/assets/0596b3a1-732e-420f-b4c1-f22edc46def7" />

- 大功告成！！终于匀速转动了！
以下是视频链接。

https://github.com/user-attachments/assets/083c1657-41d7-48df-82b6-4d0e1abfe906

**项目应用场景**：调参是 PID 的核心环节。实际工程中很少靠公式算，都是先找到不抖的 Kp，再微调 Ki，最后视情况加 Kd。这一套调参思路在后续做云台控制、小车底盘控制时同样适用。

---

## 💡 Some Problems I Met... | 踩到的坑。。。

**Problem 1**: 电机一抖一抖的转，VOFA+ 速度曲线正负跳变（负的只有一点，不明显）

- **What's wrong?** 我一开始慌了，问AI，它说很可能编码器 A 相和 B 相接反了，STM32 把正转误读成了正反转交替，PID 收到错误的方向反馈。结果核查发现没问题，尝试了各种方法解决不了问题，卡了一个世纪。我发现用之前点灯工程开环测试，测试是正常的，而用同样的开环代码放在PID工程中永远失败。最终得出结论：整个PID工程底层有问题。
- **Reason** 整个PID工程底层有问题。
- **Solution** 使用点灯工程，将PID相关代码放到那个工程里，在点灯工程中应用PID。

**Problem 2**: 开环能匀速转，PID 闭环纹丝不动

- **What's wrong?** 1.while(1) 循环和 TIM1 中断里都缺少方向设置代码，TB6612 没收到 AIN1/AIN2 方向信号。2.工程底层错误。（见Problem1）
- **Reason** 1.开环时代码在 while(1) 里反复写了 `HAL_GPIO_WritePin(PA4/PA5)`，而 PID 版本只在初始化写了一次，中断回调里也没补上。2.工程底层错误。
- **Solution** 1.在 `HAL_TIM_PeriodElapsedCallback` 中断回调里，PID 计算之前加上方向设置代码：

```c
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_4, GPIO_PIN_SET);
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET);
```
2.使用点灯工程。

**Problem 3**: Keil 编译报错 `missing: compiler version 5`

- **What's wrong?** 新版 Keil MDK（5.37+）默认不带 ARM Compiler 5，而 CubeMX 生成的默认工程使用 AC5 编译。
- **Reason** ST 和 Keil 的版本更新不同步，AC5 已被逐步淘汰但仍被大量教程使用。
- **Solution** 手动下载 ARM Compiler 5.06 安装包，安装到 Keil_v5\ARM\ARMCC，然后在工程 `Options for Target` → `Target` → `ARM Compiler` 里切换到 `Use default compiler version 5`。

**Problem 4**: VOFA+ FireWater 模式看不到第二条曲线

- **What's wrong?** sprintf 打印的两个数值格式不一致（一个浮点、一个整型），VOFA+ 解析失败只显示第一条。！！！！VOFA+这个软件的格式要求很严格，一定注意！！
- **Reason** sprintf(buf, "%.1f,%d\n", ...) 混用了 %f 和 %d，VOFA+ 要求两个数值用同一种格式并用逗号分隔。
- **Solution** 统一改成浮点数格式：`sprintf(buf, "%.1f,%.1f\r\n", (float)actualSpeed, (float)pidOutput)`;

**Problem 5**: VCC 没接但电机也能转？？？

- **What's wrong?** TB6612 的 VCC 引脚悬空，STM32 的 GPIO 通过内部保护二极管给 TB6612 漏过去了微弱电压，勉强能工作。
- **Reason** TB6612 内部电路结构允许信号引脚在 VCC 悬空时通过保护二极管倒灌供电，但极不稳定。
- **Solution** 把 VCC 接上 STM32 的 5V，保证 TB6612 在稳定电压下工作，避免高负载时停转。

**Problem 6**: 电源铁盒没有品字形插座，需要自己接 220V 线

- **What's wrong?** 买的工业开关电源只有 L/N/GND 螺丝端子，没有直接的 220V 插头。
- **Reason** 工业电源设计给设备内部安装使用，默认用户有电工基础，不自带电源线。
- **Solution** 买一根"三插头裸尾电源线"（0.75平方），棕色接 L、蓝色接 N、黄绿色接 GND。接线时务必断电，用电工胶带处理好绝缘。直接化身电工了，有点紧张，第一次处理这种危险的工作。。。

---

## 💡 Learning Insights | 学习心得

### Key Takeaways | 关键收获

- 硬件接线是嵌入式的基石：STBY 接 3.3V、VCC 接 5V、VM 接 12V，三个电压一个都不能少。接线不对，代码再对也没用。
- 多种方向排查错误，必要时尝试新的思路（如直接换一个工程），有时候会有很抽象令人意想不到的问题。
- 增量式 PID 最适合电机控制：输出平稳、抗积分饱和、故障时不会突然失控。公式只有三行，但调参需要耐心。
- 调参是门手艺活：先找不抖的 Kp，再消稳态误差的 Ki，最后抑制震荡的 Kd（如果需要）。口诀：抖降 Kp，慢加 Ki。
- VOFA+ 是调试神器：能实时看到速度曲线和 PWM 输出，比盲调效率高十倍。FireWater 协议要求两个数值用同种格式、逗号分隔。
- 芯片被锁不要慌：按住 RESET 再下载，或者 BOOT0 接高电平复位，都是常规操作。
- 代码什么的可以问AI，但**涉及硬件不确定性排查和调试尽量不要问AI**，**我在这个项目中受过的气和踩过的坑有80%都是它给的！！！！** That's so important!!

### Applications and Inspirations to MY Projects | 项目应用与启发

**RM 电控组考核**

- PID 稳速是必做任务的核心。
- 编码器测速逻辑、增量式 PID 代码、TIM2/TIM3/TIM1 配置，都是考核必备技能。

**智视寻迹（商场室内定位）**

- 未来若将 YOLO 部署到嵌入式平台，机器人移动底盘的速度控制就靠这套 PID 逻辑。
- 理解了电机速度闭环，再做云台角度控制、小车转向控制都是一样的思路。

### Looking Ahead | 未来方向铺垫

| 技术点 | 当前实现（电机恒速） | 嵌入式延伸 |
|-------|----------------------|---------------|
| 电机速度环 PI 控制 | TIM1 中断 + 增量式 PID | 位置环 PID、多环串级控制 |
| 编码器反馈 | TIM2 编码器模式 + M 法测速 | T 法测速（低速）、M/T 混合法 |
| PWM 输出 | TIM3 PWM Generation | 舵机控制、BLDC 电机控制 |
| 串口可视化调试 | VOFA+ `FireWater` | 上位机开发、蓝牙无线调试 |
| PID 参数整定 | 手动调参（抖降 Kp） | 自整定算法、模糊 PID |

这些知识是未来转向**机器人系统、无人机飞控、嵌入式智能车视觉**时必须提前储备的底层基础。现在用电机把 PID 跑通，将来叠加更复杂的控制算法时会理解得更透彻。Come on man! Keep going!!!

---

其实哈，现在我在不断学习，在各种道路上探索，尝试找到我最感兴趣的领域和方向，目前看**嵌入式机器人系统和视觉**这方面也许最契合，不过，一切还未盖棺定论且我仍在学习！！

**我可能的方向**
- 数字逻辑与FPGA架构师
- 嵌入式系统与机器人系统和视觉集成工程师
- 通信与信号处理算法工程师
- 芯片设计与验证工程师

---

## 📊 Summary! | 近日成果

| Knowledge Point | 在 RM 考核项目中的应用 |
|----------------|---------------------------|
| TB6612 电机驱动接线 | VM/VCC/STBY 三个电压不能少 |
| TIM2 编码器模式 | 四倍频测速，每转 1320 脉冲 |
| M 法测速公式 | 脉冲增量 × 60 ÷ 1320 ÷ 0.01 = RPM |
| 增量式 PID 算法 | 输出平稳，抗积分饱和 |
| PID 中断周期 10ms | TIM1 每 10ms 完成一次闭环计算 |
| VOFA+ 串口调试 | 实时观察速度曲线和 PWM 输出 |
| PID 参数整定 | 抖降 Kp，慢加 Ki，不用 Kd |
| 芯片被锁解决 | 按住 RESET 键再下载 |

---

## 📚 CODE Repository | 代码仓库

```c
// 增量式 PID 计算函数
float PID_Compute(PID_TypeDef *pid, float actual) {
    float increment;
    pid->err = pid->targetVal - actual;
    increment = pid->Kp * (pid->err - pid->errLast)
              + pid->Ki * pid->err
              + pid->Kd * (pid->err - 2*pid->errLast + pid->errPre);
    pid->outputVal += increment;
    if (pid->outputVal > 999) pid->outputVal = 999;
    if (pid->outputVal < 0)  pid->outputVal = 0;
    pid->errPre = pid->errLast;
    pid->errLast = pid->err;
    return pid->outputVal;
}

// M 法测速宏定义
#define PULSES_PER_REV (11 * 30 * 4)  // 1320
#define RPM_CALC(delta) ((float)(delta) * 6000.0f / (1320.0f * 10.0f))

// rad/s 转换（选做加分）
#define RPM_TO_RAD_PER_SEC(rpm) ((float)(rpm) * 2.0f * 3.14159f / 60.0f)
```

---

## 🔗 Some Resources | 相关资源

- [B站 中科大RM电控教程](https://space.bilibili.com/337732684)
- [VOFA+ 官网下载](https://www.vofa.plus)
- [TB6612 官方数据手册](https://toshiba-semicon-storage.com/cn/semiconductor/product/motor-driver-ics/brushed-dc-motor-driver-ics/detail.TB6612FNG.html)

---

> Uptime: +1 Day
>
> Status: Yep, Still Running...

---

*Learning Date: 2026-05-05*

*Current Projects: RM 电控组考核（PID 电机控制） | 智视寻迹 | SuperAgent*

*Future Direction: 待定 嵌入式机器人系统及视觉等* 

*Log: STM32-03-PID Closed-Loop Motor Speed Control*

---
