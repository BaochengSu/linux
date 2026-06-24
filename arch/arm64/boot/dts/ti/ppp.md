帮我制定修改计划：

IOT2050的 k3-am6548-iot2050-advanced-m2 和 k3-am6548-iot2050-advanced-pg2 这俩型号需要进行硬件的改版，
对arduino接口部分进行改动。主要的修改点：

1. SoC 的 WKUP_GPIO0_34 和 WKUP_GPIO0_41 改为not connected
2. SoC 的 WKUP_GPIO0_38 改接到 D4202 (PCA9535)的 22pin INT 用来做PCA9535的中断上报
3. SoC 的 WKUP_GPIO0_37 改接到 D4201 (PCA9535)的 22pin INT 用来做PCA9535的中断上报
4. SoC 的 WKUP_GPIO0_36 改接到 D4200 (PCA9535)的 22pin INT 用来做PCA9535的中断上报
5. D4200的I/O 0的 6 pin用作 IO14_DIR 功能，在设备树中要记作“IO14-direction”
6. D4200的I/O 0的 7 pin用作 IO15_DIR 功能，在设备树中要记作“IO15-direction”
7. D4200的I/O 1的 8 pin用作 IO16_DIR 功能，在设备树中要记作“IO16-direction”
8. D4202的I/O 1的 7 pin用作 IO17_DIR 功能，在设备树中要记作“IO17-direction”
9. D4200的I/O 1的 6 pin用作 IO18_DIR 功能，在设备树中要记作“IO18-direction”



