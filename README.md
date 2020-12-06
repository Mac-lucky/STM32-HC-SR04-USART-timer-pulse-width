# STM32-HC-SR04-USART
 STM32F303RE connected to HC-SR04 sensor measuring pulse width with timer written in Attolic TrueSTUDIO for STM32

# Installation

First we need to prepare our board in STM32CubeMX. It should work for every STM32.

Enable the TIM1 as you can see on the picture below and set Channel1 to Input capture direct mode and enable TIM1 capture compare interrupt in NVIC settings tab.

![alt text](https://github.com/Mac-lucky/STM32-HC-SR04-USART-timer-pulse-width/blob/main/images/interr.png?raw=true)

Then go to Parameter settings tab and set Prescaler to 72-1 in my case, because I have 72MHz processor, and that sets the clock of the timer to 1MHz used later on for a delay() function.

![alt text](https://github.com/Mac-lucky/STM32-HC-SR04-USART-timer-pulse-width/blob/main/images/prescaler.png?raw=true)

If the clock is not already set to 72MHz go to the clock configuration tab and set PLLCLK to your max clock frequency of your board. APB1 and APB2 timer and peripherial clocks (next to mouse on picture below) should be set to 72MHz.

![alt text](https://github.com/Mac-lucky/STM32-HC-SR04-USART-timer-pulse-width/blob/main/images/clock.png?raw=true)

Then go back to pinout configuration to connectivity list and click on USART2 and set the mode to asynchronous. Baud Rate is your personal choice. 

![alt text](https://github.com/Mac-lucky/STM32-HC-SR04-USART-timer-pulse-width/blob/main/images/usart.png?raw=true)

Now click on a GPIO port and set it to output in my case it was PA8 (D7 pin on board).

This is how it should look after all changes.

![alt text](https://github.com/Mac-lucky/STM32-HC-SR04-USART-timer-pulse-width/blob/main/images/inout.png?raw=true)

GENERATE THE PROJECT (choose your IDE)

All comments explain what happens in the code.

# Usage

![alt text](https://github.com/Mac-lucky/STM32-HC-SR04-USART-timer-pulse-width/blob/main/images/pinout.png?raw=true)
Connect your pins to the board and check your pinout. 

If you have other board than mine copy the parts of code:

* Include needed libraries
```C
#include "main.h"
#include <string.h>    //library to send string to usart
#include <stdio.h>
```

* Write callback function and delay function

```C
/* USER CODE BEGIN 0 */
void delay (uint16_t time)					//delay function in us
{
	__HAL_TIM_SET_COUNTER(&htim1, 0);
	while (__HAL_TIM_GET_COUNTER (&htim1) < time);	 //
}


uint32_t firstValue = 0;
uint32_t secondValue = 0;
uint32_t sub = 0;			//subtract value
uint8_t capturedOne = 0;
uint8_t distance  = 0;
char uartBuf[100];			//used as buffer for the string to be send to PC

#define TRIG_PIN GPIO_PIN_8
#define TRIG_PORT GPIOA

void HAL_TIM_IC_CaptureCallback(TIM_HandleTypeDef *htim)
{
	if (htim->Channel == HAL_TIM_ACTIVE_CHANNEL_1)
	{
		if (capturedOne==0) // check if first value is captured
		{
			firstValue = HAL_TIM_ReadCapturedValue(htim, TIM_CHANNEL_1);    //read the value
			capturedOne = 1;  									// change the value of the variable
			__HAL_TIM_SET_CAPTUREPOLARITY(htim, TIM_CHANNEL_1, TIM_INPUTCHANNELPOLARITY_FALLING);  		//if the value is captured change the polarity to falling edge
		}

		else if (capturedOne==1)   // if the first value is captured
		{
			secondValue = HAL_TIM_ReadCapturedValue(htim, TIM_CHANNEL_1);  // read second value
			__HAL_TIM_SET_COUNTER(htim, 0);  							// reset the counter

			sub = secondValue-firstValue;				//subtraction of the two values

			distance = sub * .034/2;			//time in us * speed of sound in air in cm / 2 (the sound needs to go back to the sensor)
			capturedOne = 0; 					// set the capture value to 0

			__HAL_TIM_SET_CAPTUREPOLARITY(htim, TIM_CHANNEL_1, TIM_INPUTCHANNELPOLARITY_RISING);  //change the polarity back to rising edge
			__HAL_TIM_DISABLE_IT(&htim1, TIM_IT_CC1);											//disable the TIM1 interrupt
		}
	}
}

void HCSR04_Read (void)
{
	HAL_GPIO_WritePin(TRIG_PORT, TRIG_PIN, GPIO_PIN_SET);  // TRIG set to HIGH
	delay(10);  // wait for 10 us
	HAL_GPIO_WritePin(TRIG_PORT, TRIG_PIN, GPIO_PIN_RESET);  // TRIG set to LOW

	__HAL_TIM_ENABLE_IT(&htim1, TIM_IT_CC1);				//Enable timer interrupt
}

/* USER CODE END 0 */
```
* Write the loop code to read the width of the pulse convert to centimeters and send to PC via Usart

```C
/* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1)
  {
	  HCSR04_Read();				//read the value
	  sprintf(uartBuf, "Distance (cm)  = %.1d\r\n", distance);			//convert to string
	  		HAL_UART_Transmit(&huart2, (uint8_t *)uartBuf, strlen(uartBuf), 100);		//transfer the string to PC // (interface, address, buffer size, timeout)
	  		HAL_Delay(1000);		//delay 1s
  }
  /* USER CODE END 3 */
}
```

It should work now
