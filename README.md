# MSPM0G3507 FreeRTOS Modern Embedded Framework

<p align="center">
  <img src="https://img.shields.io/badge/MCU-TI%20MSPM0G3507-004488?style=for-the-badge&logo=texasinstruments&logoColor=white" alt="MCU">
  <img src="https://img.shields.io/badge/Language-Pure%20C%20(C11)-00599C?style=for-the-badge&logo=c&logoColor=white" alt="Language">
  <img src="https://img.shields.io/badge/Build-CMake%20%7C%20Ninja%20%7C%20Makefile-064F8C?style=for-the-badge&logo=cmake&logoColor=white" alt="Build">
  <img src="https://img.shields.io/badge/RTOS-FreeRTOS%20(0%20Heap)-orange?style=for-the-badge" alt="RTOS">
  <img src="https://img.shields.io/badge/Flash%20Tool-OpenOCD%20%2F%20DAPLink-brightgreen?style=for-the-badge" alt="Flash">
  <img src="https://img.shields.io/badge/License-MIT-green?style=for-the-badge" alt="License">
</p>

<p align="center">
  <b>面向德州仪器 (TI) MSPM0 系列微控制器的现代、模块化、工程级纯 C 固件底座。</b><br>
  彻底脱离传统商业 IDE（Keil / CCS）依赖，提供基于 <b>CMake + Ninja + Makefile</b> 的全套自动化交叉编译、DAPLink 一键烧录、PC 宿主机单元测试与工业级解耦架构。
</p>

---

## 💡 为什么做这个项目？

传统嵌入式开发和电赛/工训备赛中，大家往往困在笨重、闭源、跨平台体验极差的传统 IDE 中（工程配置黑盒、无法融入现代 CI/CD、Git 协作痛苦、构建缓慢）。

本项目为 **TI MSPM0G3507 (ARM Cortex-M0+)** 提供了一整套**符合现代软件工程标准**的纯 C 固件模板与比赛级外设组件库：
- **纯命令行驱动**：支持 VS Code / CLion / Vim 等任意编辑器，跨平台一键编译与烧录。
- **极致资源优化**：彻底剥除无用代码，FreeRTOS 纯静态分配（0 动态堆），将 Flash 和 RAM 压榨至极限。
- **软硬件深度解耦**：核心控制算法与状态机 100% 独立于底层单片机，可在 PC 端直接通过本地 GCC 进行自动化仿真与单元测试。

---

## ✨ 核心特性

### 1. 现代双构建体系 (CMake & Makefile)
- **底层架构**：采用现代 Target-Based CMake 脚本，依赖通过 `PUBLIC` 属性链式传递，杜绝全局变量污染。
- **极速构建**：搭配 `Ninja` 构建后端，秒级增量编译，输出标准的 `.elf`、`.hex`、`.bin` 以及 `.map` 文件。
- **跨平台脚本**：
  - Windows 端：内置 `run.bat`（`run.bat build` / `run.bat flash` / `run.bat test`）
  - Linux / macOS 端：内置跨平台 `Makefile`（`make build` / `make flash` / `make test`）

### 2. 深度定制与精准裁剪的 FreeRTOS
- **零动态堆（0 Heap）**：全系统禁用动态内存分配，所有任务栈与内核对象纯静态分配，杜绝内存碎片与隐蔽泄漏。
- **中断向量精准链接（破除 Flash 浪费）**：将启动引导与中断向量从底层库抽离融入主程序，让 GCC 能够依靠 `--gc-sections` 精确剔除未使用的 FreeRTOS 模块，告别传统暴力使用 `-Wl,--whole-archive` 导致的巨大空间占用。
- **微安级浅睡眠低功耗**：结合 FreeRTOS 空闲钩子与 Cortex-M0+ 底层 `__WFI()` 指令，无任务时立即挂起 CPU 时钟，秒级响应中断。

### 3. 工业级异常防御与异步日志系统
- **异步非阻塞彩色日志 (Async Logger)**：基于 FreeRTOS StreamBuffer 环形缓冲，业务层 `printf` / `LOG_INFO` 微秒级入队返回；后台最低优先级任务轮询输出，绝不抢占核心业务。
- **中断防死锁嗅探**：底层的 `_write` 重定向内置对 ARM Cortex-M0+ `SCB->ICSR` 寄存器的状态侦测，在 ISR 中断上下文中自动拦截阻塞调用，杜绝死锁。
- **栈溢出“遗言”串口硬件轮询**：重写 `vApplicationStackOverflowHook`，当任务栈发生溢出卡死前，绕过中断调度，直接使用纯硬件串口轮询打印出违规任务名，方便现场秒级排查。

### 4. 0 硬件依赖的 PC 宿主机单元测试
- 内置 **Unity** 单元测试框架（位于 `test/` 目录）。
- 核心算法、状态机、协议解析等业务代码完全与硬件解耦，直接使用宿主机本地 GCC（MinGW / Linux GCC）编译并秒级运行测试套件，在无硬件电路板的情况下即可验证业务逻辑并护航重构。

### 5. 开箱即用的模块化赛级组件库 (`components/`)
项目中已预置并规范封装了丰富的独立外设驱动组件，即插即用：
- **`oled`**：SSD1306 7-Pin SPI 模拟驱动（内置标准 Linux VGA 8x16 字体库）
- **`tb6612`**：双路直流电机 PWM 驱动及正反转控制
- **`gray_adc`**：8 路模拟灰度传感器高精度多通道 ADC 采样与数据解算
- **`control`**：小车智能循迹闭环控制与状态机
- **`easy_menu`**：超轻量按键菜单 UI 状态机与界面渲染
- **`zdt_motor`**：步进/伺服总线绝对位置控制与 RX 状态反馈
- **`dummy_sensor`**：标准外设驱动接入范例模板

### 6. 纯净代码规范与质量守护
- 配套 `.clang-format` 统一风格。
- 精准靶向配置的 `clang-tidy`：仅扫描业务源码，自动过滤和屏蔽第三方库产生的海量无意义编译告警。
- 手动受控的底层硬件初始化配置（`ti_msp_dl_config.c`），摆脱 SysConfig GUI 自动生成黑盒代码难以维护的困境。

---

## 📁 目录结构

```text
MSPM0/
├── components/         # 独立外设驱动组件库 (OLED, 电机驱动, 灰度传感器, UI菜单等)
│   ├── control/        # 小车闭环控制逻辑
│   ├── easy_menu/      # 界面菜单组件
│   ├── gray_adc/       # 8路模拟灰度ADC传感器
│   ├── oled/           # SSD1306 OLED 显示驱动
│   ├── tb6612/         # TB6612 电机驱动
│   └── zdt_motor/      # 步进伺服电机驱动
├── include/            # 核心业务层头文件 (Logger, Board, 任务定义等)
├── src/                # 核心业务层源码与单片机硬件桥接
├── test/               # PC 宿主机本地单元测试 (Unity 框架)
├── third_party/        # 第三方依赖区 (TI MSPM0 DriverLib SDK, FreeRTOS, 链接脚本)
├── readme/             # 架构演进与深层上下文说明文档 (context.md)
├── .vscode/            # VS Code 一键编译、烧录与调试配置
├── .clang-format       # 代码格式化规范
├── CMakeLists.txt      # 主 CMake 构建脚本 (Target-Based 现代化架构)
├── gcc-arm-none-eabi.cmake # ARM GCC 交叉编译工具链配置
├── Makefile            # 跨平台 (Linux/macOS) 极简构建入口
└── run.bat             # Windows 环境极简构建入口
```

---

## 🛠️ 环境准备

开始前，请确保您的电脑已安装以下常用工具并将其添加至环境变量（`Path`）：

| 工具 | 推荐版本 | 说明 |
| :--- | :--- | :--- |
| **Arm GNU Toolchain** | 10.3+ / 13+ | `arm-none-eabi-gcc` 交叉编译器 |
| **CMake** | >= 3.20 | 跨平台项目构建系统 |
| **Ninja** | 最新版 | 超高速构建后端 |
| **OpenOCD** | 0.11+ / 0.12+ | 配合 CMSIS-DAP / DAPLink 进行固件烧录与复位 |
| **GCC (可选)** | MinGW / Linux GCC | 仅在运行 PC 端本地单元测试（`test`）时需要 |

---

## 🚀 快速上手

### 1. 编译固件

无论是 Windows 还是 Linux/macOS，都提供了最简化的指令：

**Windows (PowerShell / CMD):**
```bash
.\run.bat build
```

**Linux / macOS:**
```bash
make build
```
> 底层自动调用 `cmake -DCMAKE_TOOLCHAIN_FILE=gcc-arm-none-eabi.cmake -G Ninja -B build`，编译生成的目标 `.elf`、`.hex`、`.bin` 位于 `build/` 目录。

---

### 2. DAPLink 一键烧录

连接 CMSIS-DAP / DAPLink 调试器至目标板，执行：

**Windows:**
```bash
.\run.bat flash
```

**Linux / macOS:**
```bash
make flash
```
> 脚本将自动启动 OpenOCD，烧录固件并立即复位目标单片机。在 VS Code 中也可直接按快捷键使用预设的 `DAPLink: Flash` 任务。

---

### 3. PC 本地单元测试 (无需硬件)

在开发复杂状态机、滤波算法或协议解析时，无需接板烧录，直接在宿主机进行单元测试：

**Windows:**
```bash
.\run.bat test
```

**Linux / macOS:**
```bash
make test
```

测试执行结果示例：
```text
[AI-Script] Running host unit tests...
--- Test Suite Started ---
test_app_control_init_status:PASS
test_app_control_single_press:PASS
test_app_control_long_press:PASS
test_app_control_key_release:PASS
test_app_control_multiple_presses:PASS
-----------------------
5 Tests 0 Failures 0 Ignored
OK
```

---

### 4. 清理构建产物

```bash
# Windows
.\run.bat clean

# Linux / macOS
make clean
```

---

## 📚 进阶架构与避坑指南

如果您需要深入了解本工程的中断链接机制、波特率特殊引脚漂移避坑、异步日志安全规范等细节，请查阅核心架构文档：
👉 **[深入架构设计与避坑指南 (readme/context.md)](readme/context.md)**

---

## 📄 开源许可证

本项目基于 [MIT 许可证](LICENSE) 开源。欢迎 Star、Fork 以及提交 PR！
