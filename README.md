# 电控组第二次培训作业

- **李米惠**
- ***20263002685***

## 流水灯实现思路

本次作业使用 STM32CubeMX 配置 GPIO 输出模式，并通过 CLion 编写代码。为了实现流畅且互不干扰的流水灯效果，我采用了**非阻塞延时**的方式，具体原理如下：

### 核心原理
摒弃了传统的 `HAL_Delay()` 阻塞式延时（会导致 CPU 空转，无法处理其他任务），转而使用系统滴答定时器 `HAL_GetTick()` 进行时间差判断。

1. **时间基准**：利用 `HAL_GetTick()` 获取当前系统运行时间。
2. **状态切换**：定义一个变量 `prevTick` 记录上一次亮灯的时间。在主循环中不断检测 `HAL_GetTick() - prevTick >= 500`。
3. **逻辑执行**：只有当时间差超过 500ms 时，才执行“灭旧灯、亮新灯”的操作，并更新 `prevTick`。

### 核心代码片段

```c
// 在 while(1) 循环中
if (HAL_GetTick() - prevTick >= 500) 
{
    prevTick = HAL_GetTick(); // 更新时间戳

    // 1. 先熄灭所有 LED 
    HAL_GPIO_WritePin(LED0_GPIO_Port, LED0_Pin, GPIO_PIN_RESET);
    HAL_GPIO_WritePin(LED1_GPIO_Port, LED1_Pin, GPIO_PIN_RESET);
    HAL_GPIO_WritePin(LED2_GPIO_Port, LED2_Pin, GPIO_PIN_RESET);

    // 2. 根据状态点亮对应的 LED
    switch (state) {
        case 0: HAL_GPIO_WritePin(LED0_GPIO_Port, LED0_Pin, GPIO_PIN_SET); break;
        case 1: HAL_GPIO_WritePin(LED1_GPIO_Port, LED1_Pin, GPIO_PIN_SET); break;
        case 2: HAL_GPIO_WritePin(LED2_GPIO_Port, LED2_Pin, GPIO_PIN_SET); break;
        default:state=0;break
    }

    