# Digital Electronics Learning Log #1: The Bedrock of the Digital World — Number Systems, Codes & Logic Fundamentals

## 📝 Overview | 写在前面的概述

大一下学期的数字电路课程悄无声息地接近了尾声。回头看，从最初连AND gate（与门）和OR gate（或门）的symbol都要在纸上画半天，到期末Final将要在FPGA上独立跑通一个Finite State Machine（有限状态机），这门课的信息密度比开学时想象的大得多。

现在趁记忆还烫手，把整个学习过程按专题拆解整理——既是给自己的一份**"digital electronics世界观"**存档，也是为后续更复杂的体系（计算机组成、嵌入式系统等）打桩。

这是系列第一篇，覆盖最底层也最容易被人觉得"没什么意思"的东西：digital signal（数字信号）到底是什么、number systems（数制）之间怎么转换、logic gates（逻辑门）怎么用truth table（真值表）说话、以及第一次用FPGA验证电路时那种"纸上推演终于变成灯光"的体验，但这才是一切的起点，初恋的体验哈哈。

## 🎯 What's Covered | 本专题范围

理论部分（参考Topic 1-3, Topic 5讲义）：
- Analog signal（模拟信号）vs Digital signal（数字信号）的本质区别
- Number system conversion（数制转换）：binary、decimal、hexadecimal之间的数学关系
- Encoding（编码）：BCD code、Gray code为什么被设计出来？它们解决了什么实际问题？
- 基本Logic gates：AND、OR、NOT、XOR的truth table和symbol
- 两种standard form（标准形式）：SOP（Sum of Products，与或式）和POS（Product of Sums，或与式）

实验部分（Lab01 + Lab02）：
- FPGA开发板上验证基本gate circuits（门电路）
- 用两个2-input AND gate搭建一个3-input AND gate
- 验证Boolean algebra theorem（布尔代数定理），比对理论推导和硬件实测结果

## 📖 Learning Content | 理论学习

### Part 1: Analog or Digital? 一个工程哲学问题

从Topic 1里第一次认真思考一个看似简单的问题：**什么是signal（信号）**？

**Analog signal**是连续的——任何voltage（电压）值在理论上都有意义，像一条光滑的曲线。

**Digital signal**是离散的——只认几个预设的"档位"。在positive logic（正逻辑）系统里，high voltage（高电压）就是logic 1，low voltage（低电压）就是logic 0，中间那段灰色地带是"unusable"的。

听起来简单，但背后藏着一个工程哲学的分叉口：为什么digital systems比analog systems更流行？讲义给了三个理由——digital signal传输和存储更可靠、功耗更低、抗noise（噪声）能力更强。我的理解是：digital system把物理世界的"连续不确定性"粗暴地二值化，牺牲了精度密度，换来了可复现性和抗干扰能力。这几乎是整个digital electronics设计哲学的根基。

### Part 2: Number System Conversion — 同一个数字的不同"方言"

Topic 2的核心是positional notation（位置计数法）。任意进制下的数都可以写成各digit（数位）乘以base（基数）的幂次之和。比如 (101.01)₂ 转decimal：1×2² + 0×2¹ + 1×2⁰ + 0×2⁻¹ + 1×2⁻² = 5.25。

Decimal转binary：整数部分用"repeated division-by-2"（除2取余），小数部分用"radix multiply"（乘2取整）。转octal和hexadecimal同理，只是base分别换成8和16。

有意思的是binary↔hexadecimal↔octal之间的"捷径"转换——4位binary直接对应1位hex，3位binary对应1位octal。这不是巧合，是因为base本身就是2的幂次。

### Part 3: Encoding — 不只是0和1的堆砌

Digital circuit只认识0和1，但人类习惯decimal。Encoding（编码）就是两套"语言"之间的翻译规则。

**8421 BCD code**最直白：拿4个binary bit对应0-9，权重分别是8、4、2、1。但**Gray code**才是真正让我觉得"设计"二字有分量的编码——相邻两个码值只差一个bit。比如000→001→011→010→110→111→101→100。

**为什么要这样设计** 如果一个binary counter（二进制计数器）从0111跳到1000，四个bit同时翻转，在真实电路里不可能绝对同步——中间会短暂出现0111→1111→1000这样的**glitch**（毛刺，即瞬态错误状态）。Gray code每次只翻转一个bit，从根本上消灭了glitch。

这个设计思路让我第一次意识到：**encoding不只是在纸上定义映射关系，它是在和物理世界的非理想性博弈。**

### Part 4: Logic Gates & Truth Table — 电路的语言

Topic 3把几种基本logic gate讲透了。核心记忆：

- **AND gate**（与门）：全1才1
- **OR gate**（或门）：有1就1
- **NOT gate / Inverter**（非门/反相器）：0变1，1变0
- **XOR gate**（异或门）：不一样就1，一样就0
- **NAND gate / NOR gate**：在AND/OR基础上加inversion（取反）；唯二的**UNIVERSAL gates**!!!! 我后面才意识到，**They're so crucial!!!**

**Truth table是描述逻辑功能的唯一标准**——不同的Boolean expression（布尔表达式）可以有相同的truth table，但truth table本身是唯一的。这个性质后面贯穿了整个课程：化简、转换、验证，最后都回到truth table上对答案。

### Part 5: SOP and POS — 两种标准形式

- **SOP**（Sum of Products，与或式）：一堆product terms（与项/乘积项）用OR连起来。输出为1的条件是"任一product term为1"。

- **POS**（Product of Sums，或与式）：一堆sum terms（或项/和项）用AND连起来。输出为0的条件是"任一项为0"。

- **Minterm**（最小项）和**Maxterm**（最大项）的概念：minterm是包含所有input variables（每个变量出现一次，以true form或complemented form）的product term；maxterm是包含所有input variables的sum term。SOP用Σm()表示，POS用ΠM()表示。两种形式可以互相转换，**DeMorgan's theorem（德摩根定理）是转换的桥梁**。

## 🔬 Lab Practice | 实验验证

### Lab01：基本Gate Circuits

内容看起来简单：画symbol、写truth table、在FPGA上验证。但第一次打开Quartus画schematic（原理图）、编译、烧录到开发板，看到LED按照truth table亮灭的那一刻，才意识到"纸上的1和0"和"物理世界的voltage"之间真的有一条需要跳过去的沟。

用两个2-input AND gate搭建3-input AND gate的逻辑：**Y = (A·B)·C**。纸上写只要三个字母，但在schematic里要拉两个AND2 module、连三条input、注意引脚不能重名、输出不能悬空——这些细节在纸上推导时根本不会出现。

**2-input AND gate truth table**（Lab01实测）：

| A | B | Y |
|---|---|---|
| Low | Low | Low |
| Low | High | Low |
| High | Low | Low |
| High | High | High |

MY circuit:
<img width="978" height="786" alt="524566acd520aa0d33b5c0572c69a121" src="https://github.com/user-attachments/assets/0562d899-140f-4fe1-b7ad-bd831c25f3b0" />


### Lab02：Boolean Algebra Theorem验证

核心是验证这个等式：

**(A + B)(A + B' + C) = (A + B)(A + C)**

左右两边的circuit实现不同——左边多一个inverter处理B'、多一个OR gate——但truth table完全一样。在FPGA上分别下载左右两个circuit，比较对应input组合下两个LED output是否一致。


MY circuit:
<img width="1234" height="700" alt="ef0a9bc8fa88740c666d9df211f43f42" src="https://github.com/user-attachments/assets/9c305b34-e270-4d58-960b-f24a43a701bd" />


这个实验让我第一次直观感受到logic minimization（逻辑化简）的意义：**右边的circuit比左边少用了gate、少用了连线，实现了简便的要求，而功能完全一样**。后面学K-map的时候，这个经验让我立刻理解了为什么要绞尽脑汁化简——每一根被省掉的wire、每一个被省掉的gate，都对应着物理芯片上实实在在的面积和功耗。

## 💡 Some Problems I Met... | 踩到的坑

### Problem 1: Lab02的两个circuit到底要干嘛？Input名字能重复用吗？

**What's wrong?** 一开始完全没理解Lab02的实验意图。讲义上给了两个Boolean expression（布尔表达式）——左边是 (A + B)(A + B' + C)，右边是 (A + B)(A + C)——要求分别画circuit、分别填truth table。我心想：这不是同一组input吗？A、B、C要在两个circuit里各用一次？在Quartus里给右边circuit的input也取名A，编译直接报signal name conflict（信号名冲突）。

**Reason** **没有理解这个实验的核心目的是验证Boolean algebra theorem（布尔代数定理）的等价性**。左右两边虽然表达式不同、circuit实现不同，但truth table应该完全一样。而Quartus里每个input pin的名字在同一工程里是全局唯一的，**两个circuit共享同一组input时，应该让它们共用同一组input pin，而不是各起一套名字**。

**Solution** 搞清楚实验目的之后思路就顺了：同一组A、B、C同时连到左边和右边的circuit，然后分别看两个output（Yleft和Yright）在每种输入组合下是否一致。Quartus里input pin只建一次，两个circuit从同一组pin引线即可。这个实验也让我第一次直观感受到**logic minimization（逻辑化简）的意义——右边的circuit比左边少用了gate和inverter，功能却完全一样**。

### Problem 2: 连电路效率低，gate找半天，线拉得一团乱

**What's wrong?** Lab01和Lab02都是在Quartus的schematic editor里手动拉线。第一次上手，找logic gate（逻辑门）symbol就要在元件库里翻半天——AND2在哪？OR2叫什么？inverter是哪个？找到了之后连线更崩溃，几条线交叉在一起，编译报`unconnected pin`（引脚悬空），回头查又分不清哪根是哪根。

**Reason** 对Quartus的元件库命名不熟悉——AND2是2-input AND gate，OR2是2-input OR gate，not是inverter。这些命名规则没提前背，每次现翻效率极低。另外，一开始没有先在纸上画schematic的习惯，直接在软件里边想边连，导致线序混乱。

**Solution** 两个改变：第一，实验前先在纸上把schematic完整画好，每个gate的input/output标清楚，intermediate node（中间节点）取好名字；第二，常用gate元件名——AND2、OR2、NOT、NAND2、NOR2，后面实验速度明显快了。至于接线乱的问题，学会先画草图，构建整体意识，然后再在软件上连线。

## 💡 Learning Insights | 学习心得

### Key Takeaways

1. **Number system和encoding不是数学家没事干搞出来的游戏。** Binary简化circuit实现，Gray code消灭physical glitch，BCD方便human-machine interface——*每一种编码设计背后都有工程上的具体motivation*。

2. **Truth table是唯一的锚。** 同一个逻辑功能可以写出无数个Boolean expression，但truth table就一张。化简、转换、验证，最后都以它为准。这个思想后续贯穿了整个课程。

3. **纸面和硬件之间有一道沟。** Lab01让我意识到：逻辑正确只是必要条件，实际实现还要考虑`pin assignment`（引脚分配）、`schematic`连线规范、`intermediate node`命名——这些细节在纸上推导时根本不存在。

4. **Logic minimization有真金白银的意义。** Lab02的theorem验证直接展示了化简的效果：相同的truth table，更少的gate和wire。后面学**K-map**的时候会把这个思想推到极致。

---


*Uptime: +1 Day*

*Status: Yep, still running...*

*Log Series: Digital Electronics #1/7 — Number Systems, Codes & Logic Fundamentals*
