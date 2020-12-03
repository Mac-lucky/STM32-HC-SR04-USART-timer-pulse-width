# STM32-HC-SR04-USART
 STM32F303RE connected to HC-SR04 sensor measuring pulse width with timer written in Attolic TrueSTUDIO for STM32

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

If you have other board than mine copy the parts of code you don't have:

Line 159 to 169
Line 64 to 117

It should work now
