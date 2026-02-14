<div align="center">

# Acoustic Communication System on STM32F7
### 基于二进制频率编码的声波控制信号传输与实时解码系统

[![Platform](https://img.shields.io/badge/Platform-STM32F746%20Discovery-brightgreen)](https://www.st.com/en/evaluation-tools/32f746gdiscovery.html)
[![Middleware](https://img.shields.io/badge/Middleware-STemWin%20%7C%20CMSIS--DSP-blue)](https://www.st.com/en/embedded-software/stemwin.html)
[![License](https://img.shields.io/badge/License-Apache2.0-orange)](LICENSE)

**全链路嵌入式通信实践：音频采集 → FFT 频谱分析 → 协议解码 → 实时交互响应**

*具体技术实现细节、代码架构及调试记录详见同目录报告：* [**Acoustic_Comm_System_Technical_Report.pdf**](./Acoustic_Comm_System_Technical_Report.pdf)

</div>

---

## 🚀 项目亮点 (Highlights)

- **硬件原生集成**：深度调用 STM32F746 Discovery 板载 WM8994 音频编解码器、MEMS 麦克风及双扬声器。
- **高效 DSP 处理**：基于 **ARM CMSIS-DSP** 库实现 512 点实数 FFT，确保毫秒级指令响应。
- **鲁棒性编码协议**：设计了包含“起始位+数据位”的声波帧格式，支持数字 (0-9)、RGB (512色) 及 LED 远程控制。
- **交互式 UI 系统**：利用 **STemWin** 构建多级图形界面，支持实时波形显示与触屏控制。

## 🛠 实验环境 (Design Environment)
- **硬件平台**：STM32F746G-DISCO (Cortex-M7, 216MHz)
- **开发工具**：STM32CubeMX, Keil uVision5 / STM32CubeIDE
- **核心算法**：
  - **FFT Engine**: 基于 CMSIS-DSP 的快速傅里叶变换
  - **Synchronization**: 基于边缘检测 (Edge Detection) 的帧同步技术
  - **GUI Stack**: STemWin 图形库

---

## 📡 模块一：发送端逻辑与编码 (Transmitter & Encoding)

发送端通过软件合成特定频率的正弦波信号，通过 SAI 接口驱动 WM8994 转换为声波信号。

<details open>
<summary>展开查看详细协议定义</summary>

### 频率映射逻辑 (Frequency Mapping)
系统采用 FSK (频移键控) 的变体思想，定义如下频率：
- **起始位 (Start Bit)**: $f_{start1} = 3\text{kHz}$ (数字/LED), $f_{start2} = 6\text{kHz}$ (颜色)
- **逻辑 "0"**: $f_{low} = 8\text{kHz}$
- **逻辑 "1"**: $f_{high} = 16\text{kHz}$

</details>

---

## 🔍 模块二：接收端信号处理 (Receiver & DSP Analysis)

接收端是本项目的核心挑战，需要从混有环境噪声的音频流中精准锁定特征频率。

<details>
<summary>展开查看信号处理流程</summary>

### FFT 实时解码流程
1. **音频采集**：SAI2 接口以 $48\text{kHz}$ 采样率捕获 PDM 麦克风信号。
2. **频谱转换**：
   $$X[k] = \sum_{n=0}^{N-1} x[n] e^{-j\frac{2\pi}{N}nk}$$
   执行 512 点快速傅里叶变换，提取频域特征。
3. **阈值决策**：设置动态能量阈值，当特定频点幅值超过背景噪声阈值时判定有效，结合边缘检测防止重复触发。
</details>

---

## 📈 性能指标 (System Performance)

| 指标项目 (Metric) | 测量结果 (Results) | 备注 (Notes) |
| :--- | :--- | :--- |
| **采样率** | 48 kHz / 512-pt FFT | 兼顾频率分辨率与计算实时性 |
| **有效距离** | 0.5 米 | 典型办公/实验室安静环境 |
| **误码率 (BER)** | < 5% | 引入边缘检测与动态阈值优化后 |
| **响应延迟** | < 700 ms | 从触摸发送到接收端响应的全链路延迟 |

---

## 📝 总结与感悟 (Lessons Learned & Conclusion)

通过本次从物理层到应用层的声波通信实战，我深入实践了嵌入式系统开发中**实时性**与**健壮性**的平衡。

### 1. 信号同步的艺术 (Synchronization)
在异步通信中，捕捉信号的“瞬态”比持续读取电平更为重要。通过引入**上升沿边缘检测算法**，我们确保了每一帧信号在持续期间仅被解码一次，解决了由于采样重叠导致的“重复采集”痛点。

### 2. 抗噪声设计 (Noise Immunity)
真实环境中的高频干扰是巨大的挑战。实验证明，**动态阈值 (Dynamic Threshold)** 远优于固定阈值。通过实时测量背景噪声基准并动态调整门限，系统在复杂环境下依然保持了极高的识别精度。

### 3. 嵌入式性能优化 (Performance Optimization)
在 Cortex-M7 上充分利用 **FPU 浮点运算单元** 和 **DSP 指令集**，极大减少了计算耗时，使得高频 UI 刷新与复杂的 FFT 解码任务能够流畅并行。

---
**开发者 (Developers)**: 敖亚运 (21211175), 李科瑜 (21291172)  
**指导教师**: 郭薇薇  
**完成日期**: 2024年10月30日
