#  Ultrasonic Distance Measurement with PWM Feedback (STM32)

This project implements a **custom driver for the HC-SR04 ultrasonic distance sensor** on an **STM32 microcontroller**, using **direct HAL configuration** and **timers for precise microsecond timing**.  
The system measures distance and dynamically adjusts a **PWM signal** based on the measured value.

>  Developed in C using STM32 HAL libraries (CubeIDE environment).  
> Microcontroller family: STM32F0 / STM32F4 (compatible with HAL).  



##  Project Overview

The application measures distance using an **HC-SR04 ultrasonic sensor** connected to the STM32 board.  
The **TRIG** pin sends a 10 µs pulse to initiate measurement, while the **ECHO** pin measures the duration of the reflected signal using a timer-based delay.  
The time difference is converted into **distance (in cm)**, and a **PWM output** (from TIM1) is modulated according to the measured distance.

The PWM value is inversely proportional to distance: pwm = 65535 / distance



This allows the PWM signal to change dynamically depending on the proximity of an object.



##  Hardware Setup

| Component | Function | Connection |
|------------|-----------|------------|
| **STM32 MCU** | Main controller | Any STM32 board supported by CubeIDE |
| **HC-SR04 Sensor** | Distance measurement | TRIG → GPIO Output (PB), ECHO → GPIO Input (PB) |
| **Timer 3 (TIM3)** | Microsecond timing | Used for precise echo measurement |
| **Timer 1 (TIM1)** | PWM generation | Controls PWM output based on distance |
| **Power Supply** | 5V for sensor | VCC (5V), GND |



##  Software Description

###  Main Components
- **GPIO**: Configured for TRIG (output) and ECHO (input with pulldown)
- **TIM3**: Used for microsecond timing (Prescaler = 48 → 1 µs tick)
- **TIM1**: Used for PWM output (Period = 65535)
- **Delay_us()**: Custom delay function based on TIM3 counter



###  Measurement Procedure
1. Set **TRIG** high for 10 µs.  
2. Wait for **ECHO** pin to go high.  
3. Record start time (`val1`) using TIM3 counter.  
4. Wait for **ECHO** pin to go low.  
5. Record end time (`val2`).  
6. Compute distance: distance = (val2 - val1) * (0.034 / 2) (0.034 cm/µs is the speed of sound divided by 2 for round-trip distance)
7. Update PWM duty cycle proportionally: pwm = 65535 / distance




### 🕹️ Example Output Flow (pseudocode)
```c
HAL_GPIO_WritePin(GPIOB, TRIG_Pin, GPIO_PIN_SET);
Delay_us(10);
HAL_GPIO_WritePin(GPIOB, TRIG_Pin, GPIO_PIN_RESET);

while (!(HAL_GPIO_ReadPin(GPIOB, ECHO_Pin)));
val1 = __HAL_TIM_GET_COUNTER(&htim3);

while (HAL_GPIO_ReadPin(GPIOB, ECHO_Pin));
val2 = __HAL_TIM_GET_COUNTER(&htim3);

distance = (val2 - val1) * (0.034 / 2);
pwm = 65535 / distance;
__HAL_TIM_SET_COMPARE(&htim1, TIM_CHANNEL_1, pwm);
```

## Key Features

✅ Custom-built ultrasonic sensor driver using STM32 HAL
✅ High-resolution microsecond timing with Timer 3
✅ PWM output based on measured distance
✅ Fully functional without external libraries
✅ Suitable for real-time distance-based control systems


## Configuration Summary
Peripheral	Purpose	Key Settings
TIM3	Microsecond counter	Prescaler = 48, Period = 65535
TIM1	PWM generator	Channel 1, Period = 65535
GPIOB	TRIG / ECHO pins	TRIG = Output, ECHO = Input (Pulldown)


## Dependencies

STM32Cube HAL Drivers
STM32CubeIDE 1.xx
STM32 HAL_TIM and HAL_GPIO modules

## Testing

The driver was tested successfully with an HC-SR04 sensor at distances between 2 cm and 200 cm.
PWM duty cycle varied linearly with the detected distance.
Accuracy was stable within ±1 cm under normal conditions.
