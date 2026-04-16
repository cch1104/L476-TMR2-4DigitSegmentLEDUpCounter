# STM32 4-Digit 7-Segment Display Counter (Timer Interrupt Scan)

## Project Overview

This project uses STM32 with a 4-digit 7-segment display to build an automatic counter.

Features:

* Count from 0000 to 9999
* Initial value = 9970
* Increase every 1 second
* Timer interrupt handles display scanning
* Stable display with no flicker
* Efficient CPU usage

---

## Hardware

* STM32 Microcontroller
* 4-Digit 7-Segment Display
* GPIO Pins
* TIM2 Timer Interrupt

---

## Pin Configuration

| Digit  | GPIO Pin |
| ------ | -------- |
| digit1 | PC7      |
| digit2 | PC8      |
| digit3 | PC9      |
| digit4 | PC10     |

Segment outputs are connected through GPIOC.

---

## Source Code

```c
#define digit1 GPIO_PIN_7
#define digit2 GPIO_PIN_8
#define digit3 GPIO_PIN_9
#define digit4 GPIO_PIN_10

#include <stdint.h>

uint16_t LEDS[]={
0x3F, //0
0x06, //1
0x5B, //2
0x4F, //3
0x66, //4
0x6D, //5
0x7D, //6
0x07, //7
0x7F, //8
0x6F  //9
};

int MSD, MID1, MID2, LSD, m, n;
volatile int Count = 9970;
volatile int flag = 0;

void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
    MSD = Count / 1000;
    m = Count % 1000;

    MID2 = m / 100;
    n = m % 100;

    MID1 = n / 10;
    LSD = n % 10;

    if(flag == 0)
    {
        HAL_GPIO_WritePin(GPIOC, digit4, GPIO_PIN_RESET);
        GPIOC->ODR = LEDS[MSD];
        HAL_GPIO_WritePin(GPIOC, digit1, GPIO_PIN_SET);
        flag++;
    }
    else if(flag == 1)
    {
        HAL_GPIO_WritePin(GPIOC, digit1, GPIO_PIN_RESET);
        GPIOC->ODR = LEDS[MID2];
        HAL_GPIO_WritePin(GPIOC, digit2, GPIO_PIN_SET);
        flag++;
    }
    else if(flag == 2)
    {
        HAL_GPIO_WritePin(GPIOC, digit2, GPIO_PIN_RESET);
        GPIOC->ODR = LEDS[MID1];
        HAL_GPIO_WritePin(GPIOC, digit3, GPIO_PIN_SET);
        flag++;
    }
    else if(flag == 3)
    {
        HAL_GPIO_WritePin(GPIOC, digit3, GPIO_PIN_RESET);
        GPIOC->ODR = LEDS[LSD];
        HAL_GPIO_WritePin(GPIOC, digit4, GPIO_PIN_SET);
        flag = 0;
    }
}

int main(void)
{
    HAL_Init();
    SystemClock_Config();
    MX_GPIO_Init();
    MX_TIM2_Init();

    HAL_GPIO_WritePin(GPIOC, digit1, GPIO_PIN_RESET);
    HAL_GPIO_WritePin(GPIOC, digit2, GPIO_PIN_RESET);
    HAL_GPIO_WritePin(GPIOC, digit3, GPIO_PIN_RESET);
    HAL_GPIO_WritePin(GPIOC, digit4, GPIO_PIN_RESET);

    HAL_TIM_Base_Start_IT(&htim2);

    while(1)
    {
        Count++;

        if(Count > 9999)
            Count = 0;

        HAL_Delay(1000);
    }
}
```

---

## Program Logic

### Main Loop

* Increase Count every 1 second
* Reset to 0000 after 9999

### Timer Interrupt

TIM2 repeatedly scans digits:

1. Show thousand digit
2. Show hundred digit
3. Show ten digit
4. Show one digit

Because scanning is very fast, the display appears fully ON.

---

## Example Output

```text
9970
9971
9972
9973
...
9999
0000
0001
```

---

## CubeMX Timer2 Suggestion

| Setting      | Value    |
| ------------ | -------- |
| Prescaler    | 39999   |
| Counter Mode | down       |
| Period       | 7 |
| Internal Clock Division | Division by 2 |
| auto-reload preload    | Enable   |


Adjust based on your clock source.

---

## Skills Learned

* STM32 GPIO Output
* Timer Interrupt
* 7-Segment Multiplexing
* Embedded Counter Design
