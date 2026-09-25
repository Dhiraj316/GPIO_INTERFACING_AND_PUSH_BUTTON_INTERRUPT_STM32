# GPIO_INTERFACING_AND_PUSH_BUTTON_INTERRUPT_STM32
an interactive GPIO application developed for the STM32F407VETx microcontroller. The project demonstrates how to use External Interrupts (EXTI) to read push-button inputs and dynamically alter the blink rate of an onboard LED. It utilizes a non-blocking delay approach (`HAL_GetTick()`) alongside hardware interrupts.

## Hardware & Programmer Study
* **Board:** STM32F407 Development Board
* **Inputs/Outputs:** 
  * Onboard/External Push Button (Configured to `PA0` / `EXTI0`)
  * Onboard/External LED (Configured to `PA6(INBUILT)`)
* **Programmer:** ST-LINK V2

### 1. Project Creation
### 2. IOC Pin Configuration
1. Open the `.ioc` Device Configuration Tool.
2. Go to **System Core > SYS** and set **Debug** to **Serial Wire**.
3. **LED Setup:** Left-click your LED pin (e.g., `PA6`) and select **GPIO_Output**.
4. **Button Setup (EXTI):** Left-click your Button pin (e.g., `PA0`) and select **GPIO_EXTI0**.
5. Go to **System Core > GPIO**. 
   * Click on your Button pin (`PA0`).
   * Set **GPIO mode** to **External Interrupt Mode with Rising edge trigger detection** (if active HIGH) or **Falling edge** (if active LOW).
   * Set **GPIO Pull-up/Pull-down** to **Pull-down** (if active HIGH) or **Pull-up** (if active LOW).
6. Go to **System Core > NVIC**.
   * Check the **Enabled** box next to **EXTI line0 interrupt** to enable the interrupt in the Nested Vectored Interrupt Controller.
7. Save (`Ctrl+S`) and click **Yes** to generate the initialization code.

### 3. Application Code
Open `Core/Src/main.c`. We will declare our tracking variables, add the interrupt callback function to handle the button press, and put the non-blocking LED toggle in the main loop.

Locate `/* USER CODE BEGIN 0 */` and add the state variables:

uint32_t delay_options[] = {100, 500, 1000}; // Fast, Medium, Slow delays in ms
volatile uint8_t current_delay_idx = 1;      // Start at Medium (500ms)
uint32_t previous_blink_time = 0;
volatile uint32_t last_interrupt_time = 0;   // For debouncing


#THEN 
/* USER CODE BEGIN 4 */
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
    // Check if the interrupt came from our button pin
    if(GPIO_Pin == GPIO_PIN_0)
    {
        uint32_t current_time = HAL_GetTick();
        // 300ms debounce timer to prevent multiple triggers from physical switch bounce
        if (current_time - last_interrupt_time > 300) 
        {
            current_delay_idx++; // Move to next speed
            
            if (current_delay_idx > 2) {
                current_delay_idx = 0; // Wrap back to fast speed
            }
            last_interrupt_time = current_time;
        }
    }
}
/* USER CODE END 4 */


#THEN IN WHILE 

while (1)
  {
      // Non-blocking LED Toggle based on the dynamically updated delay array
      if (HAL_GetTick() - previous_blink_time >= delay_options[current_delay_idx]) 
      {
          HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_6);
          previous_blink_time = HAL_GetTick(); // Reset LED timer
      }
  }    


 ### 4. Build and Flash
Click the Build (Hammer) icon and verify the .elf binary is generated with zero errors.
Connect your ST-LINK and target board via USB.
Click the Run (Play) icon, leave the ST-LINK debug probe defaults, and click OK.
The LED will blink at 500ms intervals. Press the button to trigger the interrupt and instantly cycle the speed between Fast (100ms), Medium (500ms), and Slow (1000ms).

### I uploaded the zip file of this project in case there is any issue. just extract the zip file and import in the stmcube IDE for the reference.

