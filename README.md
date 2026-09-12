# Automatic-Vehicle-Parking-System-using-STM32-Nucleo-L031K6

## Aim

To design and implement an **Automatic Vehicle Parking System using STM32 Nucleo-L031K6** that detects the occupancy of parking slots, determines the number of available spaces, and indicates when the parking area is full.

---

## Components Required

- STM32 Nucleo-L031K6
- Two pushbuttons to simulate vehicle sensors
- Onboard LED to indicate parking full status
- Wokwi Simulator
- Connecting wires
- Serial Monitor

---

## Theory

An **Automatic Vehicle Parking System** is used to detect the availability of parking spaces and provide information about whether parking slots are **available or occupied**.

In a practical parking system, **IR sensors or ultrasonic sensors** can be used to detect the presence of vehicles in individual parking slots.

In this Wokwi experiment, **pushbuttons are used to simulate the vehicle sensors**.

Two parking slots are considered:

- **Pushbutton 1** represents the sensor for Parking Slot 1.
- **Pushbutton 2** represents the sensor for Parking Slot 2.

When a pushbutton is pressed, the STM32 changes the corresponding parking slot status between **AVAILABLE** and **OCCUPIED**.

The STM32 calculates the number of available parking spaces. When both parking slots are occupied, the parking area is considered **FULL**, and the onboard LED is switched **ON**.

The parking-slot information is also displayed on the **Wokwi Serial Monitor** using UART communication.

---

## program
#include <stdio.h> #include <stdint.h> #include <stm32l0xx_hal.h>

/* Slot 1 sensor pushbutton: D2 / PA10 */ #define SLOT1_PORT GPIOA #define SLOT1_PIN GPIO_PIN_10

/* Slot 2 sensor pushbutton: D3 / PB0 */ #define SLOT2_PORT GPIOB #define SLOT2_PIN GPIO_PIN_0

/* Onboard LED: D13 / PB3 */ #define FULL_LED_PORT GPIOB #define FULL_LED_PIN GPIO_PIN_3

/* USART2 virtual serial pins */ #define VCP_TX_PIN GPIO_PIN_2 #define VCP_RX_PIN GPIO_PIN_15

UART_HandleTypeDef huart2;

void SystemClock_Config(void); static void MX_GPIO_Init(void); static void MX_USART2_UART_Init(void); static void Display_Parking_Status(uint8_t slot1, uint8_t slot2); void Error_Handler(void);

int main(void) { uint8_t slot1Occupied = 0; uint8_t slot2Occupied = 0;

GPIO_PinState previousSlot1Button = GPIO_PIN_SET; GPIO_PinState previousSlot2Button = GPIO_PIN_SET;

GPIO_PinState currentSlot1Button; GPIO_PinState currentSlot2Button;

HAL_Init(); SystemClock_Config();

MX_GPIO_Init(); MX_USART2_UART_Init();

printf("\r\n========================================\r\n"); printf("STM32 Automatic Vehicle Parking System\r\n"); printf("========================================\r\n"); printf("D2 / PA10 : Slot 1 sensor button\r\n"); printf("D3 / PB0 : Slot 2 sensor button\r\n"); printf("D13 / PB3 : Parking Full indicator\r\n\r\n");

printf("Press a slot button to change its status.\r\n");

Display_Parking_Status(slot1Occupied, slot2Occupied);

while (1) { /* * Read both vehicle sensor pushbuttons. * The buttons use internal pull-up resistors: * * Released = GPIO_PIN_SET * Pressed = GPIO_PIN_RESET */ currentSlot1Button = HAL_GPIO_ReadPin(SLOT1_PORT, SLOT1_PIN);

currentSlot2Button =
    HAL_GPIO_ReadPin(SLOT2_PORT, SLOT2_PIN);

/*
 * Detect a new press of the Slot 1 button.
 */
if ((previousSlot1Button == GPIO_PIN_SET) &&
    (currentSlot1Button == GPIO_PIN_RESET))
{
  HAL_Delay(50);

  if (HAL_GPIO_ReadPin(SLOT1_PORT, SLOT1_PIN) ==
      GPIO_PIN_RESET)
  {
    /*
     * Toggle Slot 1 between available and occupied.
     */
    slot1Occupied = !slot1Occupied;

    printf("\r\nSlot 1 sensor activated.\r\n");

    Display_Parking_Status(
        slot1Occupied,
        slot2Occupied);
  }
}

/*
 * Detect a new press of the Slot 2 button.
 */
if ((previousSlot2Button == GPIO_PIN_SET) &&
    (currentSlot2Button == GPIO_PIN_RESET))
{
  HAL_Delay(50);

  if (HAL_GPIO_ReadPin(SLOT2_PORT, SLOT2_PIN) ==
      GPIO_PIN_RESET)
  {
    /*
     * Toggle Slot 2 between available and occupied.
     */
    slot2Occupied = !slot2Occupied;

    printf("\r\nSlot 2 sensor activated.\r\n");

    Display_Parking_Status(
        slot1Occupied,
        slot2Occupied);
  }
}

previousSlot1Button = currentSlot1Button;
previousSlot2Button = currentSlot2Button;

HAL_Delay(20);
} }

/*

Display the status of the parking area and control
the Parking Full indicator LED. */ static void Display_Parking_Status( uint8_t slot1, uint8_t slot2) { uint8_t occupiedSlots; uint8_t availableSlots;
occupiedSlots = slot1 + slot2; availableSlots = 2U - occupiedSlots;

printf("----------------------------------------\r\n");

printf("Slot 1: %s\r\n", slot1 ? "OCCUPIED" : "AVAILABLE");

printf("Slot 2: %s\r\n", slot2 ? "OCCUPIED" : "AVAILABLE");

printf("Available slots: %u\r\n", availableSlots);

/*

Both slots occupied means parking is full. */ if (availableSlots == 0U) { HAL_GPIO_WritePin( FULL_LED_PORT, FULL_LED_PIN, GPIO_PIN_SET);
printf("Parking Status: FULL\r\n");
printf("Entry gate: CLOSED\r\n");
} else { HAL_GPIO_WritePin( FULL_LED_PORT, FULL_LED_PIN, GPIO_PIN_RESET);

printf("Parking Status: SPACE AVAILABLE\r\n");
printf("Entry gate: OPEN\r\n");
}

printf("----------------------------------------\r\n"); }

/*

GPIO initialization. */ static void MX_GPIO_Init(void) { GPIO_InitTypeDef GPIO_InitStruct = {0};
__HAL_RCC_GPIOA_CLK_ENABLE(); __HAL_RCC_GPIOB_CLK_ENABLE();

/*

Configure Slot 1 pushbutton PA10 as an input. */ GPIO_InitStruct.Pin = SLOT1_PIN; GPIO_InitStruct.Mode = GPIO_MODE_INPUT; GPIO_InitStruct.Pull = GPIO_PULLUP; GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
HAL_GPIO_Init(SLOT1_PORT, &GPIO_InitStruct);

/*

Configure Slot 2 pushbutton PB0 as an input. */ GPIO_InitStruct.Pin = SLOT2_PIN; GPIO_InitStruct.Mode = GPIO_MODE_INPUT; GPIO_InitStruct.Pull = GPIO_PULLUP; GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
HAL_GPIO_Init(SLOT2_PORT, &GPIO_InitStruct);

/*

Configure PB3 onboard LED as an output. */ GPIO_InitStruct.Pin = FULL_LED_PIN; GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP; GPIO_InitStruct.Pull = GPIO_NOPULL; GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
HAL_GPIO_Init(FULL_LED_PORT, &GPIO_InitStruct);

/*

Initially switch the Parking Full LED OFF. */ HAL_GPIO_WritePin( FULL_LED_PORT, FULL_LED_PIN, GPIO_PIN_RESET); }
/*

System clock configuration. */ void SystemClock_Config(void) { RCC_OscInitTypeDef RCC_OscInitStruct = {0}; RCC_ClkInitTypeDef RCC_ClkInitStruct = {0}; RCC_PeriphCLKInitTypeDef PeriphClkInit = {0};
__HAL_PWR_VOLTAGESCALING_CONFIG( PWR_REGULATOR_VOLTAGE_SCALE1);

RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSI;

RCC_OscInitStruct.HSIState = RCC_HSI_ON;

RCC_OscInitStruct.HSICalibrationValue = RCC_HSICALIBRATION_DEFAULT;

RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON; RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSI; RCC_OscInitStruct.PLL.PLLMUL = RCC_PLLMUL_4; RCC_OscInitStruct.PLL.PLLDIV = RCC_PLLDIV_2;

if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK) { Error_Handler(); }

RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK | RCC_CLOCKTYPE_SYSCLK | RCC_CLOCKTYPE_PCLK1 | RCC_CLOCKTYPE_PCLK2;

RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_PLLCLK;

RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;

RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV1;

RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV1;

if (HAL_RCC_ClockConfig( &RCC_ClkInitStruct, FLASH_LATENCY_1) != HAL_OK) { Error_Handler(); }

PeriphClkInit.PeriphClockSelection = RCC_PERIPHCLK_USART2;

PeriphClkInit.Usart2ClockSelection = RCC_USART2CLKSOURCE_PCLK1;

if (HAL_RCCEx_PeriphCLKConfig( &PeriphClkInit) != HAL_OK) { Error_Handler(); } }

/*

USART2 initialization. */ static void MX_USART2_UART_Init(void) { GPIO_InitTypeDef GPIO_InitStruct = {0};
__HAL_RCC_GPIOA_CLK_ENABLE(); __HAL_RCC_USART2_CLK_ENABLE();

/*

PA2 -> USART2_TX
PA15 -> USART2_RX */ GPIO_InitStruct.Pin = VCP_TX_PIN | VCP_RX_PIN;
GPIO_InitStruct.Mode = GPIO_MODE_AF_PP; GPIO_InitStruct.Pull = GPIO_NOPULL;

GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_VERY_HIGH;

GPIO_InitStruct.Alternate = GPIO_AF4_USART2;

HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);

huart2.Instance = USART2; huart2.Init.BaudRate = 115200; huart2.Init.WordLength = UART_WORDLENGTH_8B; huart2.Init.StopBits = UART_STOPBITS_1; huart2.Init.Parity = UART_PARITY_NONE; huart2.Init.Mode = UART_MODE_TX_RX; huart2.Init.HwFlowCtl = UART_HWCONTROL_NONE; huart2.Init.OverSampling = UART_OVERSAMPLING_16;

huart2.Init.OneBitSampling = UART_ONE_BIT_SAMPLE_DISABLE;

huart2.AdvancedInit.AdvFeatureInit = UART_ADVFEATURE_NO_INIT;

if (HAL_UART_Init(&huart2) != HAL_OK) { Error_Handler(); } }

/*

Error handler. */ void Error_Handler(void) { HAL_GPIO_WritePin( FULL_LED_PORT, FULL_LED_PIN, GPIO_PIN_RESET);
while (1) { } }

/*

Redirect printf() output to USART2. */ #define STDOUT_FILENO 1 #define STDERR_FILENO 2
int _write(int file, uint8_t *ptr, int len) { if ((file == STDOUT_FILENO) || (file == STDERR_FILENO)) { HAL_UART_Transmit( &huart2, ptr, len, HAL_MAX_DELAY);

return len;
}

return -1; }

## Pin Configuration

| Component | STM32 Pin | Function |
|---|---|---|
| Slot 1 Pushbutton | PA12 | Digital Input |
| Slot 2 Pushbutton | PB0 | Digital Input |
| Parking Full LED | PB3 | Digital Output |
| USART2 TX | PA2 | Serial Data Transmission |
| USART2 RX | PA15 | Serial Data Reception |
| Pushbutton Common | GND | Ground |

---

## Block Diagram

~~~text
     Slot 1                 Slot 2
   Pushbutton             Pushbutton
 (Vehicle Sensor)       (Vehicle Sensor)
       |                      |
       v                      v
     PA12                    PB0
       |                      |
       +----------+-----------+
                  |
                  v
       +-------------------------+
       | STM32 Nucleo-L031K6     |
       |                         |
       | Read Slot Status        |
       |          |              |
       |          v              |
       | Calculate Available     |
       | Parking Spaces          |
       +-----------+-------------+
                   |
          +--------+--------+
          |                 |
          v                 v
       PB3 LED           USART2
          |                 |
          v                 v
     Parking Full      Serial Monitor
      Indicator        Parking Status
~~~

---

## Parking Status Conditions

| Slot 1 | Slot 2 | Available Slots | Parking Status | LED |
|---|---|---:|---|---|
| Available | Available | 2 | Space Available | OFF |
| Occupied | Available | 1 | Space Available | OFF |
| Available | Occupied | 1 | Space Available | OFF |
| Occupied | Occupied | 0 | Parking Full | ON |

---

## Algorithm

1. Start the program.
2. Initialize the STM32 HAL library.
3. Configure the system clock.
4. Configure **PA12** as the Slot 1 sensor input.
5. Configure **PB0** as the Slot 2 sensor input.
6. Configure **PB3** as the Parking Full LED output.
7. Enable internal pull-up resistors for the pushbutton inputs.
8. Initialize USART2 for serial communication.
9. Set both parking slots initially as **AVAILABLE**.
10. Read the status of the Slot 1 and Slot 2 pushbuttons.
11. When Pushbutton 1 is pressed, change the status of Slot 1.
12. When Pushbutton 2 is pressed, change the status of Slot 2.
13. Calculate the number of available parking slots.
14. If both slots are occupied, switch the Parking Full LED **ON**.
15. If at least one parking slot is available, keep the LED **OFF**.
16. Display the slot status and number of available spaces on the Serial Monitor.
17. Repeat the process continuously.

---

## Circuit Connections

### Slot 1 Pushbutton

| Pushbutton Terminal | STM32 Connection |
|---|---|
| Terminal 1 | PA12 |
| Terminal 2 | GND |

### Slot 2 Pushbutton

| Pushbutton Terminal | STM32 Connection |
|---|---|
| Terminal 1 | PB0 |
| Terminal 2 | GND |

### Parking Full Indicator

| Indicator | STM32 Connection |
|---|---|
| Onboard LED | PB3 |

The program uses **internal pull-up resistors** for the pushbutton inputs. Therefore, external pull-up resistors are not required.

---

## Circuit Diagram

~~~text
        Slot 1 Pushbutton
      (Vehicle Sensor 1)

        +-------------+
PA12 ---|             |
        |   BUTTON    |
GND  ---|             |
        +-------------+


        Slot 2 Pushbutton
      (Vehicle Sensor 2)

        +-------------+
PB0  ---|             |
        |   BUTTON    |
GND  ---|             |
        +-------------+


       +---------------------------+
       | STM32 Nucleo-L031K6       |
       |                           |
       | PA12 <- Slot 1 Sensor     |
       | PB0  <- Slot 2 Sensor     |
       |                           |
       | PB3  -> Parking Full LED  |
       |                           |
       | PA2  -> USART2 TX         |
       | PA15 -> USART2 RX         |
       +-------------+-------------+
                     |
                     |
                   USART2
                     |
                     v
              Serial Monitor
~~~

---

## Pushbutton Logic

The pushbuttons are configured using the STM32 **internal pull-up resistors**.

Therefore:

- **Button Released → Logic HIGH (1)**
- **Button Pressed → Logic LOW (0)**

Initially:

~~~text
Slot 1 = AVAILABLE
Slot 2 = AVAILABLE
~~~

When the Slot 1 pushbutton is pressed:

~~~text
Slot 1 = OCCUPIED
Slot 2 = AVAILABLE

Available Slots = 1
~~~

When the Slot 2 pushbutton is also pressed:

~~~text
Slot 1 = OCCUPIED
Slot 2 = OCCUPIED

Available Slots = 0
Parking Status = FULL
LED = ON
~~~

Pressing the corresponding pushbutton again changes the slot status back to **AVAILABLE**, simulating a vehicle leaving the parking slot.

---

## Why Pushbuttons Are Used

In an actual Automatic Vehicle Parking System, vehicle detection is normally performed using sensors such as:

- IR sensors
- Ultrasonic sensors
- Proximity sensors

For this Wokwi experiment, pushbuttons are used to **simulate vehicle detection**.

A button press represents a change in the parking-slot condition.

~~~text
Wokwi Simulation            Actual System

Pushbutton             ->   IR/Ultrasonic Sensor

Button Press           ->   Vehicle Detected

Button Press Again     ->   Vehicle Leaves

PB3 LED                ->   Parking Full Indicator
~~~

This makes it easy to understand and test the parking-control logic without requiring an actual vehicle sensor.

---

## Procedure

1. Open the **Wokwi Simulator**.
2. Select the **STM32 Nucleo-L031K6** board.
3. Add two pushbuttons.
4. Connect one terminal of Pushbutton 1 to **PA12**.
5. Connect the other terminal of Pushbutton 1 to **GND**.
6. Connect one terminal of Pushbutton 2 to **PB0**.
7. Connect the other terminal of Pushbutton 2 to **GND**.
8. Use the onboard LED connected to **PB3** as the Parking Full indicator.
9. Enter the STM32 HAL program for the Automatic Vehicle Parking System.
10. Compile the program.
11. Start the Wokwi simulation.
12. Open the **Serial Monitor**.
13. Press Pushbutton 1 to simulate a vehicle occupying Slot 1.
14. Press Pushbutton 2 to simulate a vehicle occupying Slot 2.
15. Observe the parking-slot status on the Serial Monitor.
16. Verify that the onboard LED turns **ON when both parking slots are occupied**.
17. Press a slot button again to simulate the vehicle leaving the parking space.

---

## Expected Output

### Initially – Both Slots Available

~~~text
Slot 1: AVAILABLE
Slot 2: AVAILABLE
Available Slots: 2
Parking Status: SPACE AVAILABLE
Entry Gate: OPEN
~~~

### Vehicle in Slot 1

~~~text
Slot 1: OCCUPIED
Slot 2: AVAILABLE
Available Slots: 1
Parking Status: SPACE AVAILABLE
Entry Gate: OPEN
~~~

### Both Slots Occupied

~~~text
Slot 1: OCCUPIED
Slot 2: OCCUPIED
Available Slots: 0
Parking Status: FULL
Entry Gate: CLOSED
~~~

The onboard LED connected to **PB3** turns **ON** when the parking area is full.

---

## Working

The two pushbuttons simulate vehicle-detection sensors for two parking slots. The STM32 continuously monitors the pushbuttons through the GPIO input pins **PA12** and **PB0**.

When a pushbutton is pressed, the program changes the status of the corresponding parking slot between **AVAILABLE** and **OCCUPIED**.

The STM32 then calculates the available parking spaces using:

`Available Slots = Total Slots - Occupied Slots`

For this experiment, the total number of parking slots is **2**.

If both parking slots are occupied, the number of available slots becomes zero. The STM32 switches the **PB3 onboard LED ON** to indicate **Parking Full**.

If one or more parking slots are available, the LED remains **OFF**.

The status of each parking slot, number of available spaces, parking condition, and entry-gate status are transmitted through **USART2** and displayed on the **Wokwi Serial Monitor**.

---

## Applications

- Shopping mall parking systems
- Hospital parking areas
- College and university campuses
- Office parking facilities
- Airport parking systems
- Smart city parking systems
- Residential parking management

---

## Result

Thus, the **Automatic Vehicle Parking System using STM32 Nucleo-L031K6** was designed and implemented successfully. The system detected the simulated occupancy of parking slots, calculated the number of available spaces, indicated the **Parking Full** condition using the onboard LED, and displayed the parking status on the **Wokwi Serial Monitor**.
