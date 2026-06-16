# FM VCO Control Module (BB910 Varactor)

## 概述 (Overview)

这是一个基于BB910变容二极管的FM发射机数控电压调频模块。通过可变电阻分压，可实现平滑的频率调节。

This is a voltage-controlled FM transmitter module using BB910 varactor diode. Smooth frequency adjustment is achieved through variable resistor voltage divider.

## 技术指标 (Specifications)

| 参数 | 规格 |
|------|------|
| **输入电压** | 9V DC |
| **输出控制电压** | 0 - 8.5V |
| **频率调范** | ±2 - 5 MHz |
| **中心频率** | 88 - 108 MHz (FM Band) |
| **功耗** | <50 mA |
| **调节方式** | 模拟电位器分压 |
| **PCB尺寸** | 50mm × 30mm (紧凑版) |

## 电路原理 (Circuit Principle)

### 工作流程:

1. **电源分压阶段** (Voltage Divider Stage)
   - 9V输入通过R1(100K)限流
   - R2(100K可变电阻)分压产生0-8.5V
   - 输出电压范围可通过R2调节

2. **滤波阶段** (Filter Stage)
   - R3(10K) + C1(10uF)组成低通滤波网络
   - 截止频率: ~1.6 kHz
   - 作用: 消除PWM纹波，提供干净的控制电压

3. **变容二极管** (Varactor Diode)
   - BB910变容二极管
   - 正极(K脚)连接到FM发射机振荡器OUT
   - 负极(A脚)接地
   - 改变反向偏置电压 → 改变结电容 → 改变谐振频率

## 元器件清单 (BOM)

| 编号 | 元器件 | 值 | 数量 | 说明 |
|------|--------|-----|------|------|
| J1 | 电源接头 | 9V | 1 | 从FM发射机电源 |
| J2 | 输出接头 | - | 1 | 连接到FM发射机 |
| R1 | 精密电阻 | 100K 1/4W | 1 | 限流电阻 |
| R2 | 可变电阻 | 100K滑片式 | 1 | **核心调频元件** |
| R3 | 精密电阻 | 10K 1/4W | 1 | 滤波网络 |
| C1 | 电解电容 | 10µF 16V | 1 | 低通滤波 |
| D1 | 变容二极管 | BB910 | 1 | **核心调频元件** |

## 接线方式 (Wiring Guide)

### 连接器定义:

```
J1 (电源输入):
  Pin 1: +9V (从FM发射机电源)
  Pin 2: GND

J2 (控制电压输出):
  Pin 1: Control Voltage (0-8.5V)
  Pin 2: GND

J3 (到振荡器):
  Pin 1: 连接到FM发射机的C7点(振荡器OUT)
  Pin 2: GND (返回FM发射机)
```

## 调试步骤 (Debugging Steps)

### 1. 静态测试 (Static Testing)

```
使用万用表测量:

• R2在最左端(0Ω):
  J2 Pin 1 应测得 ≈ 0.8V

• R2在中间(50K):
  J2 Pin 1 应测得 ≈ 4.5V

• R2在最右端(100K):
  J2 Pin 1 应测得 ≈ 8.5V
```

### 2. 动态测试 (Dynamic Testing)

```
• 使用频率计连接FM天线
• 旋转R2电位器
• 观察频率是否平滑变化
• 调范应该 ±2-5 MHz
```

### 3. 音质测试 (Audio Quality Testing)

```
• 接收端播放音乐
• 调节电位器改变频率
• 应能清晰收听全频率范围
• 无明显杂音或失真
```

## 焊接建议 (Soldering Tips)

1. **焊接顺序**:
   - 先焊被动元件 (R1, R3, C1)
   - 再焊可变电阻 R2 (避免过多加热)
   - 最后焊BB910 (变容二极管怕静电)
   - 最后焊接头

2. **注意事项**:
   - BB910焊接前要放电(触摸金属物体)
   - 焊接温度不超过350°C
   - 焊接时间 <3秒/脚
   - C1电解电容注意极性!

## 频率调整曲线 (Frequency Tuning Curve)

```
控制电压(V) → 频率偏移(MHz)

  0.8V → -5MHz (下限)
  2.5V → -2MHz
  4.5V →  0MHz (中心)
  6.5V → +2MHz
  8.5V → +5MHz (上限)

曲线特性: 近似线性,可通过调整R2精确校准
```

## 常见问题 (FAQ)

### Q1: 频率调不动?
A: 
- 检查BB910是否焊接正确
- 确认J3连接到振荡器OUT
- 用万用表测量控制电压是否有变化

### Q2: 频率调范太小?
A:
- 检查BB910型号是否正确(应为BB910)
- 尝试调整R1或R3的值
- 检查FM发射机的L1电感值

### Q3: 有杂音或失真?
A:
- 增大C1滤波电容值(试试22uF)
- 缩短控制电压线长度
- 检查GND连接是否良好

### Q4: 能用PWM代替电位器吗?
A:
- 可以!需要添加RC低通滤波器
- 推荐使用 1K电阻 + 10uF电容
- MCU PWM频率 >20kHz效果最好

## 升级方案 (Upgrade Options)

### 方案A: Arduino自动扫频
```cpp
void setup() {
  pinMode(9, OUTPUT);  // PWM输出
}

void loop() {
  for(int i = 0; i < 256; i++) {
    analogWrite(9, i);  // 0-255对应0-8V
    delay(50);
  }
}
```

### 方案B: STM32高精度DAC
- 使用STM32L4的12bit DAC
- 可实现精确的频率锁定
- 支持EEPROM保存频率预设

### 方案C: 直接集成到主板
- 将此模块集成到FM发射机PCB
- 减少连接线数量
- 改善EMI性能

## 文件说明 (Files)

- `FM-VCO-Module.kicad_sch` - KiCad原理图
- `FM-VCO-Module.kicad_pcb` - KiCad PCB布局
- `FM-VCO-Module.kicad_pro` - KiCad项目文件
- `BOM.csv` - 物料清单
- `README.md` - 本文件

## 许可证 (License)

MIT License - 自由使用和修改

## 作者 (Author)

jay11-11 - 2026

---

**需要帮助?** 提交Issue或讨论!

**Questions?** Submit an Issue!
