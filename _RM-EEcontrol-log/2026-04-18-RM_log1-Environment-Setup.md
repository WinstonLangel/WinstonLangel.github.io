# 01 - STM32 Development Environment Setup Log
> 搭建RM战队考核所需的环境

**Date / 日期**: 2026-04-18  
**Author / 作者**: WinstonL  
**Log 01_Objective / 目标**: 完成 RM 电控组第一阶段考核所需 STM32 开发环境的搭建，并成功创建第一个 Keil 工程，打开 `main.c` 文件。

---

## 1. Preparation / 准备工作

- **Hardware / 硬件**: STM32F103C8T6 最小系统板、ST-Link 下载器、面包板、杜邦线等。
- **Software to Install / 需安装的软件**:
  - Java Runtime Environment (JRE) 8
  - STM32CubeMX (v6.13.0)
  - Keil MDK (μVision V5.38)
  - STM32F1 系列芯片支持包 (Pack)

---

## 2. Installing Java / 安装 Java

STM32CubeMX 依赖 Java 运行环境，需先安装 Java。

1. 访问 Java 官网 `java.com/zh-CN/download`，下载 Windows 离线安装包。
2. 双击运行安装程序，保持默认设置完成安装。
3. 验证安装：按 `Win + R` 输入 `cmd`，执行 `java -version`，显示版本号即成功。

---

## 3. Installing STM32CubeMX / 安装 STM32CubeMX

1. 从 ST 官网或网盘获取 `SetupSTM32CubeMX-6.13.0.exe`。
2. **以管理员身份运行**安装程序，安装路径避免中文和空格。
3. 安装完成后，打开 CubeMX，通过 `Help` → `Manage embedded software packages` 安装 `STM32F1` 固件包（推荐 1.8.0 及以上版本）。

---

## 4. Installing Keil MDK / 安装 Keil MDK

1. 获取 MDK-ARM 安装包（如 MDK538.EXE），**以管理员身份运行**。
2. 安装路径设为 `D:\Keil_v5`，确保路径不含中文。
3. 安装过程中弹出的驱动安装窗口（ST-Link 驱动）**务必勾选并安装**。
4. 安装完成后先不启动 Keil。

---

## 5. Installing STM32F1 Device Pack / 安装芯片支持包

Keil 需要安装对应芯片的支持包才能识别 STM32F103C8T6。

1. 打开 Keil，点击工具栏 **Pack Installer** 图标。
2. 在左侧搜索 `STM32F103C8`。
3. 在右侧 `Packs` 标签页中找到 `Keil::STM32F1xx_DFP`，点击 `Install` 完成安装。

---

## 6. Activating Keil MDK / 激活 Keil MDK

采用**注册机离线激活**方式（个人学习用途），步骤如下：

1. **关闭所有安全软件**：包括 Windows Defender 实时保护、第三方杀毒软件（如 360、电脑管家）。
2. 获取注册机 `keygen.exe`，将其所在文件夹加入 Windows Defender 排除项。
3. **以管理员身份运行 Keil μVision5**，点击 `File` → `License Management...`，复制 **CID** 码。
4. **以管理员身份运行 keygen.exe**：
   - 粘贴 CID，`Target` 选择 `ARM`。
   - 点击 `Generate` 生成激活码。
5. 将激活码粘贴至 Keil 的 `New License ID Code (LIC)` 框中，点击 `Add LIC`。
6. 看到 `LIC Added Successfully` 提示，有效期至 2032 年，激活成功。

---

## 7. Creating the First Project in STM32CubeMX / 在 CubeMX 中创建第一个工程

1. 打开 STM32CubeMX，点击 `Start My project from MCU` → `ACCESS TO MCU SELECTOR`。
2. 搜索并选择 `STM32F103C8Tx`。
3. 在 `Pinout & Configuration` 界面，将 `PC13` 引脚设置为 `GPIO_Output`（用于测试 LED）。
4. 切换到 `Project Manager` 标签页：
   - **Project Name**: `test`
   - **Project Location**: `D:\STM32`
   - **Toolchain / IDE**: `MDK-ARM`
5. 点击右上角 **`GENERATE CODE`**，生成 Keil 工程。

---

## 8. Opening main.c in Keil / 在 Keil 中打开 main.c

1. 进入工程目录 `D:\STM32\test\MDK-ARM\`，双击 `test.uvprojx` 用 Keil 打开。
2. 在 Keil 左侧 **Project** 窗口中，展开 `Application/User` 文件夹。
3. 双击 `main.c`，即可在右侧编辑区看到代码。

---

## 9. Common Problems and Solutions / 常见问题与解决方案

以下是整个环境搭建过程中我遇到的问题及解决方法。

### 🔧 Some problems I met... 
- **Problem1**: ST 官网下载 CubeMX 时邮箱无法验证，无法获取下载链接

  **My Solution 1**: 检查邮箱垃圾箱，ST 的验证邮件有时会被误判。

  **My Solution 2**: 改用百度网盘或夸克网盘的第三方资源下载稳定版本（如 v6.13.0）。

  **My Solution 3**: 尝试使用 `www.stmcu.com.cn` 中国区官网，下载流程可能更简化。

- **Problem2**: Keil 官网下载时报 HTTP 500 错误，页面无法访问

  **My Solution 1**: 清除浏览器缓存，更换浏览器（如 Chrome、Edge）重试。

  **My Solution 2**: 修改 DNS 为公共 DNS（如 `114.114.114.114`），刷新网络后再访问。
 
  **My Solution 3**: 使用第三方网盘下载稳定版本的 MDK 安装包（如 MDK538）。

- **Problem3**: Keil 社区版联网激活反复失败，报错 "Error processing response from the license server"。我在这里报错卡了一万年，气死我了。。。

  **My Solution 1**: 彻底关闭 Windows Defender 防火墙、实时保护、云提供的保护。

  **My Solution 2**: 确保 Keil 以管理员身份运行。
 
  **My Solution 3**: **最终解决方案**：放弃联网激活，改用注册机离线激活（详见第 6 节）。

- **Problem4**: Pack Installer 在线下载芯片支持包持续超时

  **My Solution 1**: 从 Keil 官网 `www.keil.com/dd2/Pack/` 手动下载 `.pack` 文件，双击离线安装。

  **My Solution 2**: 从网盘下载他人分享的已解压 Pack 文件夹，直接复制到 `Keil_v5\Packs` 目录下。

- **Problem5**: 注册机压缩包无法解压，提示“拒绝访问存档”

  **My Solution 1**: 将压缩包所在文件夹加入 Windows Defender 的排除项中，防止实时保护拦截。

  **My Solution 2**: 使用 7-Zip 等第三方解压软件代替 Windows 自带解压功能。
 
  **My Solution 3**: 寻找无需解压、可直接运行的 `.exe` 版本注册机。

- **Problem6**: 在 Keil 中打开工程后找不到 main.c 文件

  **My Solution 1**: 在 Keil 左侧 Project 窗口中，手动展开 `Application/User` 文件夹，双击 `main.c` 即可。

  **My Solution 2**: 若文件夹内确实没有，右键 `Application/User` → `Add Existing Files to Group...`，手动添加 `Core\Src\main.c`。

---

## 10. Summary / 总结

经过以上步骤，STM32 开发环境（STM32CubeMX + Keil MDK）已成功搭建并激活，第一个测试工程已生成并可在 Keil 中正常打开 `main.c` 文件。接下来将进入代码编写与调试阶段，开始电机控制相关任务的学习。

---

## 11. What's next? / 下一步

编写 LED 闪烁代码，测试编译、下载流程，验证硬件连接。
