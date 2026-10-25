# 电控组第二次培训作业

- **姓名：许锂**
- ***学号：20263008243***

## 硬件与引脚配置

本作业使用 STM32 的 GPIO 输出功能控制三个 LED。引脚在 STM32CubeMX 中使用 User Label 命名，具体配置如下：

| 功能   | 引脚  | User Label | GPIO 模式   | 初始输出 |
| ---- | --- | ---------- | --------- | ---- |
| LED0 | PA4 | `LED0`     | Push Pull | Low  |
| LED1 | PA6 | `LED1`     | Push Pull | Low  |
| LED2 | PA7 | `LED2`     | Push Pull | Low  |

CubeMX 生成代码后，可以在 `Core/Inc/main.h` 中找到对应的引脚宏定义：

```c
#define LED0_Pin        GPIO_PIN_4
#define LED0_GPIO_Port  GPIOA

#define LED1_Pin        GPIO_PIN_6
#define LED1_GPIO_Port  GPIOA

#define LED2_Pin        GPIO_PIN_7
#define LED2_GPIO_Port  GPIOA
```

程序通过 `LED0_Pin`、`LED0_GPIO_Port` 等宏控制引脚，不在代码中重复填写 PA4、PA6 和 PA7。

## 流水灯实现思路

本程序使用 `HAL_GetTick()` 实现非阻塞的 500 ms 定时，没有使用 `HAL_Delay()`，也没有使用空循环等待。

`HAL_GetTick()` 返回系统启动后经过的毫秒数。主循环每次读取当前时间，并计算它距离上次切换 LED 的时间：

```c
uint32_t now = HAL_GetTick();
uint32_t elapsed = now - last_tick;
```

当 `elapsed` 大于或等于 500 时，程序将流水灯状态切换到下一个 LED，并把 `last_tick` 更新为当前时间。时间尚未达到 500 ms 时，程序不执行切换，但仍会继续执行主循环中的其他内容。

时间差使用无符号整数 `uint32_t` 进行计算。即使 `HAL_GetTick()` 的计数值发生溢出，`now - last_tick` 仍能正确表示两次读取之间经过的时间。

## 核心代码

下面代码假设 LED 为高电平点亮。如果开发板上的 LED 是低电平点亮，交换 `GPIO_PIN_SET` 和 `GPIO_PIN_RESET` 即可。

```c
/* USER CODE BEGIN 2 */
uint32_t last_tick = HAL_GetTick();
uint8_t led_step = 0;
/* USER CODE END 2 */

while (1)
{
    /* USER CODE BEGIN 3 */
    uint32_t now = HAL_GetTick();

    if ((now - last_tick) >= 500U)
    {
        last_tick = now;

        led_step++;
        if (led_step >= 3U)
        {
            led_step = 0U;
        }
    }

    HAL_GPIO_WritePin(LED0_GPIO_Port, LED0_Pin,
        (led_step == 0U) ? GPIO_PIN_SET : GPIO_PIN_RESET);

    HAL_GPIO_WritePin(LED1_GPIO_Port, LED1_Pin,
        (led_step == 1U) ? GPIO_PIN_SET : GPIO_PIN_RESET);

    HAL_GPIO_WritePin(LED2_GPIO_Port, LED2_Pin,
        (led_step == 2U) ? GPIO_PIN_SET : GPIO_PIN_RESET);

    /* USER CODE END 3 */
}
```

程序启动时 `led_step` 为 0，因此首先点亮 LED0。每经过 500 ms，`led_step` 依次变为 1、2、0，对应点亮 LED1、LED2、LED0，从而形成三个 LED 的循环流水灯。
