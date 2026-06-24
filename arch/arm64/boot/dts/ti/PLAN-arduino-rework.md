# IOT2050 Arduino 接口硬件改版 - 设备树修改计划（修订版）

## 设计原则

1. **不动 `k3-am65-iot2050-arduino-connector.dtsi`** — 该文件保持原样，供旧硬件使用
2. **在 board DTS 中 override** — 新硬件在 `advanced-m2.dts` 和 `advanced-pg2.dts` 中覆盖必要节点
3. **0-based 引脚编号** — PCA9535 每个 port 内引脚从 0 开始计数

---

## 涉及文件

| 文件 | 操作 |
|------|------|
| `k3-am65-iot2050-arduino-connector.dtsi` | **不动** |
| `k3-am6548-iot2050-advanced-m2.dts` | 添加 override 段 |
| `k3-am6548-iot2050-advanced-pg2.dts` | 添加 override 段 |
| (可选) 新建共享 dtsi | 如果两个 board override 完全相同，可提取为共享文件 |

---

## PCA9535 引脚编号约定（0-based within each port）

在本次改版中，PCA9535 每个 I/O port 的引脚从 0 开始编号：

| 用户描述 | 解读 (0-based) | 寄存器 bit |
|---------|---------------|-----------|
| I/O 0 pin 6 | IO0_6 | bit 6 |
| I/O 0 pin 7 | IO0_7 | bit 7 |
| I/O 1 pin 6 | IO1_6 | bit 14 |
| I/O 1 pin 7 | IO1_7 | bit 15 |
| I/O 1 pin 8 | IO1_7 (第 8 个 pin) | bit 15 |

> 注："I/O 1 pin 8" 在 0-based 范围内不存在（port 1 只有 0-7），这里按第 8 个引脚（1-based）解读为 IO1_7。

---

## Override 详细内容

### Override 1：`&wkup_pmx0` — 重定义 `arduino_io_oe_pins_default`

原内容（在 dtsi 中）：
```dts
arduino_io_oe_pins_default: arduino-io-oe-default-pins {
    pinctrl-single,pins = <
        AM65X_WKUP_IOPAD(0x0058, PIN_OUTPUT, 7)  /* WKUP_GPIO0_34 */
        AM65X_WKUP_IOPAD(0x0060, PIN_OUTPUT, 7)  /* WKUP_GPIO0_36 */
        AM65X_WKUP_IOPAD(0x0064, PIN_OUTPUT, 7)  /* WKUP_GPIO0_37 */
        AM65X_WKUP_IOPAD(0x0068, PIN_OUTPUT, 7)  /* WKUP_GPIO0_38 */
        AM65X_WKUP_IOPAD(0x0074, PIN_OUTPUT, 7)  /* WKUP_GPIO0_41 */
    >;
};
```

新内容（在 board DTS 中 override）：
```dts
arduino_io_oe_pins_default: arduino-io-oe-default-pins {
    pinctrl-single,pins = <
        /* WKUP_GPIO0_36 → D4200 INT (PIN_INPUT_PULLUP 因 INT 为开漏输出) */
        AM65X_WKUP_IOPAD(0x0060, PIN_INPUT_PULLUP, 7)
        /* WKUP_GPIO0_37 → D4201 INT */
        AM65X_WKUP_IOPAD(0x0064, PIN_INPUT_PULLUP, 7)
        /* WKUP_GPIO0_38 → D4202 INT */
        AM65X_WKUP_IOPAD(0x0068, PIN_INPUT_PULLUP, 7)
    >;
};
```

**变更摘要：**
| 动作 | 引脚 | 旧方向 | 新方向 |
|------|------|-------|-------|
| 删除 | WKUP_GPIO0_34 | OUTPUT | NC |
| 保留改向 | WKUP_GPIO0_36 | OUTPUT | INPUT_PULLUP (INT) |
| 保留改向 | WKUP_GPIO0_37 | OUTPUT | INPUT_PULLUP (INT) |
| 保留改向 | WKUP_GPIO0_38 | OUTPUT | INPUT_PULLUP (INT) |
| 删除 | WKUP_GPIO0_41 | OUTPUT | NC |

### Override 2：`&wkup_gpio0` — 更新 gpio-line-names

更新 lines 34、36、37、38、41：

```dts
&wkup_gpio0 {
    gpio-line-names =
        "wkup_gpio0-base", "", "", "", "UART0-mode1", "UART0-mode0",
        "UART0-enable", "UART0-terminate", "", "WIFI-disable",
        "", "", "", "", "", "", "", "", "", "",
        "", "A4A5-I2C-mux", "", "", "", "USER-button", "", "", "","IO0",
        "IO1", "IO2", "", "IO3", "", "A5",           /* 34→NC */
        "D4200-INT", "D4201-INT", "D4202-INT", "A3",  /* 36/37/38→INT */
        "", "", "A4", "A2", "A1", "A0", "", "", "IO13",  /* 41→NC */
        "IO11",
        "IO12", "IO10";
};
```

> 注：如果希望 owner 更明确，INT 标签也可以命名为 `"pcal9535_1-int"` 等。

### Override 3：`&pcal9535_1` (D4200) — 添加中断 + 更新 gpio-line-names

```dts
&pcal9535_1 {
    interrupt-parent = <&wkup_gpio0>;
    interrupts = <36 IRQ_TYPE_EDGE_FALLING>;
    gpio-line-names =
        "A0-pull", "A1-pull", "A2-pull", "A3-pull", "A4-pull",
        "A5-pull",                              /* IO0_5: 不变 */
        "IO14-direction",                       /* IO0_6: 新增 */
        "IO15-direction",                       /* IO0_7: 新增 */
        "IO14-enable", "IO15-enable", "IO16-enable",
        "IO17-enable", "IO18-enable", "IO19-enable",
                                                /* IO1_5(bit13): 不变 */
        "IO18-direction",                       /* IO1_6(bit14): 新增 */
        "IO16-direction";                       /* IO1_7(bit15): 新增 */
};
```

**变更摘要：**
| 寄存器位 | 旧标签 | 新标签 |
|---------|-------|-------|
| bit 6 (IO0_6) | (空) | IO14-direction |
| bit 7 (IO0_7) | (空) | IO15-direction |
| bit 14 (IO1_6) | (空) | IO18-direction |
| bit 15 (IO1_7) | (空) | IO16-direction |

### Override 4：`&pcal9535_2` (D4201) — 添加中断

```dts
&pcal9535_2 {
    interrupt-parent = <&wkup_gpio0>;
    interrupts = <37 IRQ_TYPE_EDGE_FALLING>;
};
```

gpio-line-names 不变（IO0-direction ~ IO13-direction + IO19-direction 依然在 D4201）。

### Override 5：`&pcal9535_3` (D4202) — 添加中断 + 扩展 gpio-line-names

```dts
&pcal9535_3 {
    interrupt-parent = <&wkup_gpio0>;
    interrupts = <38 IRQ_TYPE_EDGE_FALLING>;
    gpio-line-names =
        "IO0-pull", "IO1-pull", "IO2-pull", "IO3-pull",
        "IO4-pull", "IO5-pull", "IO6-pull", "IO7-pull",
        "IO8-pull", "IO9-pull", "IO10-pull", "IO11-pull",
        "IO12-pull", "IO13-pull",
        "IO17-direction",      /* IO1_6(bit14): 新增 */
        "";                    /* IO1_7(bit15): 保留空 */
};
```

> 原 D4202 的 gpio-line-names 只有 14 个条目（bits 0-13），扩展为 16 个。

---

## 各芯片最终功能分布

### D4200 (pcal9535_1 @ 0x20)

| bit | I/O | 功能 |
|-----|-----|------|
| 0-5 | IO0_0 ~ IO0_5 | A0-pull ~ A5-pull (不变) |
| 6 | IO0_6 | **IO14-direction** (NEW) |
| 7 | IO0_7 | **IO15-direction** (NEW) |
| 8-12 | IO1_0 ~ IO1_4 | IO14-enable ~ IO18-enable (不变) |
| 13 | IO1_5 | IO19-enable (不变) |
| 14 | IO1_6 | **IO18-direction** (NEW) |
| 15 | IO1_7 | **IO16-direction** (NEW) |
| INT | — | WKUP_GPIO0_36 (NEW) |

### D4201 (pcal9535_2 @ 0x21)

| bit | 功能 | 变化 |
|-----|------|------|
| 0-13 | IO0-direction ~ IO13-direction | 不变 |
| 14 | IO19-direction | 不变 |
| INT | WKUP_GPIO0_37 (NEW) | 新增中断 |

### D4202 (pcal9535_3 @ 0x25)

| bit | I/O | 功能 |
|-----|-----|------|
| 0-13 | IO0_0 ~ IO1_5 | IO0-pull ~ IO13-pull (不变) |
| 14 | IO1_6 | **IO17-direction** (NEW) |
| 15 | IO1_7 | (空) |
| INT | — | WKUP_GPIO0_38 (NEW) |

---

## 实施策略选项

### 选项 A：直接在两个 board DTS 中分别写 override（推荐）

在 `k3-am6548-iot2050-advanced-m2.dts` 和 `k3-am6548-iot2050-advanced-pg2.dts` 中分别添加上述 5 段 override 代码。优点：每个 board 自包含，依赖清晰。

### 选项 B：提取共享 override dtsi

新建 `k3-am65-iot2050-arduino-connector-pg2-rework.dtsi`，内容为上述 5 段 override，两个 board 在 `#include "k3-am65-iot2050-arduino-connector.dtsi"` 之后 include 它。优点：无代码重复。

---

## 需要确认的问题

1. **引脚编号确认**：上述映射假设 "I/O 0 pin 6" = IO0_6 (bit 6)，"I/O 1 pin 8" = IO1_7 (bit 15, 第 8 个引脚) 是否准确？
2. **INT 上拉配置**：PIN_INPUT_PULLUP 是否正确？如果 PCA9535 INT 脚在板上有外部上拉电阻，可能用 PIN_INPUT 即可。
3. **旧硬件兼容**：原 `k3-am65-iot2050-arduino-connector.dtsi` 不动，旧硬件继续使用旧的 compatible 即可区分。
