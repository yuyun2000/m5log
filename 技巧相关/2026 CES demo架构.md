![](17e2e637c788e686866a7ab2a5cabf13.jpg)
左侧为StackChan，主控ESP32，可爱的桌面机器人；右侧为AI-Pyramid，主控AX8850，为本次demo的计算中心
![](eff7cdf28a4e50bc8c22fc3fca2c6d4c.jpg)

HA-Server运行在AI-Pyramid，其次上面运行着 HA-离线LLM语音控制服务-运行Qwen0.5B模型和SensenVoice高精度ASR；VLM实时画面理解-InternVL3-1B；YOLO行人、骨骼检测

面板上的传感器和设备为：AirQ高精度空气传感器；ENVUnit环境传感器；D-LIGHT光照传感器；这些都通过cores3接入ha并在TAB5上显示，并且支持设置ha内部的自动任务，比如天黑开灯。

三个led灯，Master Room Light、Guest Room Light和最上面的State Light通过Dial接入Ha，并且支持Dial的旋转控制，同时支持Ha的自动化控制，同时在Tab5上显示并控制。

离线的LLM语音控制支持以下的功能

| 功能类别          | 场景说明              | 中文指令示例 (Chinese)   | 英文指令示例 (English)                                   |
| :------------ | :---------------- | :----------------- | :------------------------------------------------- |
| **基础单项控制**    | 单一设备的开关、亮度或色彩调节   | “把主卧灯调成红色”         | "Turn the master room light to red"                |
| **进阶色彩调节**    | 支持单色、混合色及特殊色彩名称   | “客房灯颜色变更为咖啡色”      | "Change the guest light to coffee color"           |
| **多设备连续指令**   | 一句话包含多个不相关的控制动作   | “打开风扇，顺便把客房灯调绿”    | "Turn on the fan and set the guest light to green" |
| **同类统一控制**    | 针对某一类设备（如：灯具）批量操作 | “关闭所有的灯”           | "Turn off all the lights"                          |
| **统一 + 连续控制** | 批量操作后紧接特定设备动作     | “把所有灯变咖啡色，然后打开加热器” | "Set all lights to coffee and turn on the heater"  |
| **全局一键控制**    | 针对所有预设设备的集合控制     | “关掉家里所有的东西”        | "Turn off everything in the house"                 |
以及上述指令的连续触发，比如开灯1，关灯2，开电扇，关加热器，但是每控制一个设备，LLM的输出耗时就要多1s（因为输出了更多token）
