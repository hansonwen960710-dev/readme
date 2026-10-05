# readme
/* USER CODE BEGIN Header */
/**
  ******************************************************************************
  * @file           : main.c
  * @brief          : Lab3 FSM + Warning Auto Return Challenge
  ******************************************************************************
  */
/* USER CODE END Header */
/* Includes ------------------------------------------------------------------*/
#include "main.h"

/* Private includes ----------------------------------------------------------*/
/* USER CODE BEGIN Includes */

/* USER CODE END Includes */

/* Private typedef -----------------------------------------------------------*/
/* USER CODE BEGIN PTD */

/* USER CODE END PTD */

/* Private define ------------------------------------------------------------*/
/* USER CODE BEGIN PD */

/* USER CODE END PD */

/* Private macro -------------------------------------------------------------*/
/* USER CODE BEGIN PM */

/* USER CODE END PM */

/* Private variables ---------------------------------------------------------*/

/* USER CODE BEGIN PV */

/* =========================================================
   State
   0 = NORMAL
   1 = WARNING
   2 = EMERGENCY
   ========================================================= */
#define STATE_NORMAL      0
#define STATE_WARNING     1
#define STATE_EMERGENCY   2

int state = STATE_NORMAL;


/* =========================================================
   Timing
   now      = 現在時間
   previous = LED 上次 Toggle 時間
   ========================================================= */
uint32_t now = 0;
uint32_t previous = 0;


/* =========================================================
   WARNING Timeout
   記錄進入 WARNING 的時間
   ========================================================= */
uint32_t warning_start = 0;


/* =========================================================
   STOP Event Flag
   PC13 EXTI 設定
   ========================================================= */
volatile int STOP = 0;


/* =========================================================
   MODE Event
   ========================================================= */
int MODE_event = 0;


/* =========================================================
   MODE Button
   PC9 Pull-up

   沒按 = HIGH
   按下 = LOW
   ========================================================= */
GPIO_PinState mode_previous = GPIO_PIN_SET;
GPIO_PinState mode_current = GPIO_PIN_SET;


/* USER CODE END PV */

/* Private function prototypes -----------------------------------------------*/
void SystemClock_Config(void);
static void MX_GPIO_Init(void);
/* USER CODE BEGIN PFP */

/* USER CODE END PFP */

/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */

/* USER CODE END 0 */

/**
  * @brief  The application entry point.
  * @retval int
  */
int main(void)
{

  /* USER CODE BEGIN 1 */

  /* USER CODE END 1 */

  /* MCU Configuration--------------------------------------------------------*/

  /* Reset of all peripherals, Initializes the Flash interface and the Systick. */
  HAL_Init();

  /* USER CODE BEGIN Init */

  /* USER CODE END Init */

  /* Configure the system clock */
  SystemClock_Config();

  /* USER CODE BEGIN SysInit */

  /* USER CODE END SysInit */

  /* Initialize all configured peripherals */
  MX_GPIO_Init();
  /* USER CODE BEGIN 2 */

    /* =====================================================
       Initial State = NORMAL
       ===================================================== */

    state = STATE_NORMAL;

    previous = HAL_GetTick();


    /* NORMAL 初始輸出 */

    /* Green ON */
    HAL_GPIO_WritePin(
        GPIOC,
        GPIO_PIN_8,
        GPIO_PIN_SET
    );

    /* Yellow OFF */
    HAL_GPIO_WritePin(
        GPIOC,
        GPIO_PIN_5,
        GPIO_PIN_RESET
    );

    /* Red OFF */
    HAL_GPIO_WritePin(
        GPIOC,
        GPIO_PIN_6,
        GPIO_PIN_RESET
    );


  /* USER CODE END 2 */

  /* Infinite loop */
  /* USER CODE BEGIN WHILE */

    while (1)
    {

        /* =====================================================
           STEP 1
           Read Events
           ===================================================== */


        /* 每一圈先清除 MODE Event */
        MODE_event = 0;


        /* -----------------------------------------------------
           MODE Button
           PC9
           Pull-up
           Active Low
           ----------------------------------------------------- */

        mode_current =
            HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_9);


        /* -----------------------------------------------------
           偵測按下瞬間

           前一次 HIGH
           現在   LOW

           HIGH → LOW
           代表剛按下
           ----------------------------------------------------- */

        if ((mode_previous == GPIO_PIN_SET) &&
            (mode_current == GPIO_PIN_RESET))
        {

            /* Debounce */
            HAL_Delay(50);


            /* 50 ms 後再次確認 */
            if (HAL_GPIO_ReadPin(
                    GPIOC,
                    GPIO_PIN_9
                ) == GPIO_PIN_RESET)
            {
                MODE_event = 1;
            }
        }


        /* 儲存目前 MODE 狀態 */
        mode_previous = mode_current;


        /* 讀取目前時間 */
        now = HAL_GetTick();



        /* =====================================================
           STEP 2
           STATE TRANSITION
           ===================================================== */


        /* -----------------------------------------------------
           NORMAL + MODE
           → WARNING
           ----------------------------------------------------- */

        if ((state == STATE_NORMAL) &&
            (MODE_event == 1))
        {

            state = STATE_WARNING;


            /* LED Timing 重新開始 */
            previous = now;


            /* ★ 記錄進入 WARNING 的時間 */
            warning_start = now;


            /* WARNING 初始輸出 */

            /* Green OFF */
            HAL_GPIO_WritePin(
                GPIOC,
                GPIO_PIN_8,
                GPIO_PIN_RESET
            );


            /* Yellow ON */
            HAL_GPIO_WritePin(
                GPIOC,
                GPIO_PIN_5,
                GPIO_PIN_SET
            );


            /* Red OFF */
            HAL_GPIO_WritePin(
                GPIOC,
                GPIO_PIN_6,
                GPIO_PIN_RESET
            );
        }


        /* -----------------------------------------------------
           WARNING + MODE
           → NORMAL
           ----------------------------------------------------- */

        else if ((state == STATE_WARNING) &&
                 (MODE_event == 1))
        {

            state = STATE_NORMAL;


            /* NORMAL Timing 重新開始 */
            previous = now;


            /* NORMAL 初始輸出 */

            /* Green ON */
            HAL_GPIO_WritePin(
                GPIOC,
                GPIO_PIN_8,
                GPIO_PIN_SET
            );


            /* Yellow OFF */
            HAL_GPIO_WritePin(
                GPIOC,
                GPIO_PIN_5,
                GPIO_PIN_RESET
            );


            /* Red OFF */
            HAL_GPIO_WritePin(
                GPIOC,
                GPIO_PIN_6,
                GPIO_PIN_RESET
            );
        }


        /* -----------------------------------------------------
           EMERGENCY + MODE

           MODE 無效
           保持 EMERGENCY
           ----------------------------------------------------- */

        else if ((state == STATE_EMERGENCY) &&
                 (MODE_event == 1))
        {

            state = STATE_EMERGENCY;
        }



        /* =====================================================
           ★ CHALLENGE
           WARNING 5 秒自動返回 NORMAL
           ===================================================== */

        if (state == STATE_WARNING)
        {

            /* 進 WARNING 已滿 5000 ms */
            if ((now - warning_start) >= 5000)
            {

                state = STATE_NORMAL;


                /* NORMAL Timing 重新開始 */
                previous = now;


                /* NORMAL 初始輸出 */

                /* Green ON */
                HAL_GPIO_WritePin(
                    GPIOC,
                    GPIO_PIN_8,
                    GPIO_PIN_SET
                );


                /* Yellow OFF */
                HAL_GPIO_WritePin(
                    GPIOC,
                    GPIO_PIN_5,
                    GPIO_PIN_RESET
                );


                /* Red OFF */
                HAL_GPIO_WritePin(
                    GPIOC,
                    GPIO_PIN_6,
                    GPIO_PIN_RESET
                );
            }
        }



        /* =====================================================
           STOP EVENT
           任意 State → EMERGENCY

           STOP 優先處理
           ===================================================== */

        if (STOP == 1)
        {

            state = STATE_EMERGENCY;


            /* 清除 STOP Event */
            STOP = 0;


            /* EMERGENCY Output */

            /* Green OFF */
            HAL_GPIO_WritePin(
                GPIOC,
                GPIO_PIN_8,
                GPIO_PIN_RESET
            );


            /* Yellow OFF */
            HAL_GPIO_WritePin(
                GPIOC,
                GPIO_PIN_5,
                GPIO_PIN_RESET
            );


            /* Red ON */
            HAL_GPIO_WritePin(
                GPIOC,
                GPIO_PIN_6,
                GPIO_PIN_SET
            );
        }



        /* =====================================================
           STEP 3
           STATE BEHAVIOR
           ===================================================== */


        /* -----------------------------------------------------
           NORMAL

           Green LED = 1 Hz

           Toggle every 500 ms

           ON  500 ms
           OFF 500 ms

           Total = 1000 ms = 1 Hz
           ----------------------------------------------------- */

        if (state == STATE_NORMAL)
        {

            if ((now - previous) >= 500)
            {

                previous = now;


                HAL_GPIO_TogglePin(
                    GPIOC,
                    GPIO_PIN_8
                );
            }
        }


        /* -----------------------------------------------------
           WARNING

           Yellow LED = 2 Hz

           Toggle every 250 ms

           ON  250 ms
           OFF 250 ms

           Total = 500 ms = 2 Hz
           ----------------------------------------------------- */

        else if (state == STATE_WARNING)
        {

            if ((now - previous) >= 250)
            {

                previous = now;


                HAL_GPIO_TogglePin(
                    GPIOC,
                    GPIO_PIN_5
                );
            }
        }


        /* -----------------------------------------------------
           EMERGENCY

           Red LED always ON
           ----------------------------------------------------- */

        else if (state == STATE_EMERGENCY)
        {

            /* Green OFF */
            HAL_GPIO_WritePin(
                GPIOC,
                GPIO_PIN_8,
                GPIO_PIN_RESET
            );


            /* Yellow OFF */
            HAL_GPIO_WritePin(
                GPIOC,
                GPIO_PIN_5,
                GPIO_PIN_RESET
            );


            /* Red ON */
            HAL_GPIO_WritePin(
                GPIOC,
                GPIO_PIN_6,
                GPIO_PIN_SET
            );
        }
    }

    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
  }
  /* USER CODE END 3 */
}

/**
  * @brief System Clock Configuration
  * @retval None
  */
void SystemClock_Config(void)
{
  RCC_OscInitTypeDef RCC_OscInitStruct = {0};
  RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};

  /** Configure the main internal regulator output voltage
  */
  HAL_PWREx_ControlVoltageScaling(PWR_REGULATOR_VOLTAGE_SCALE1_BOOST);

  /** Initializes the RCC Oscillators according to the specified parameters
  * in the RCC_OscInitTypeDef structure.
  */
  RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSI;
  RCC_OscInitStruct.HSIState = RCC_HSI_ON;
  RCC_OscInitStruct.HSICalibrationValue = RCC_HSICALIBRATION_DEFAULT;
  RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
  RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSI;
  RCC_OscInitStruct.PLL.PLLM = RCC_PLLM_DIV4;
  RCC_OscInitStruct.PLL.PLLN = 85;
  RCC_OscInitStruct.PLL.PLLP = RCC_PLLP_DIV2;
  RCC_OscInitStruct.PLL.PLLQ = RCC_PLLQ_DIV2;
  RCC_OscInitStruct.PLL.PLLR = RCC_PLLR_DIV2;
  if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
  {
    Error_Handler();
  }

  /** Initializes the CPU, AHB and APB buses clocks
  */
  RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK|RCC_CLOCKTYPE_SYSCLK
                              |RCC_CLOCKTYPE_PCLK1|RCC_CLOCKTYPE_PCLK2;
  RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_PLLCLK;
  RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
  RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV1;
  RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV1;

  if (HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_4) != HAL_OK)
  {
    Error_Handler();
  }
}

/**
  * @brief GPIO Initialization Function
  * @param None
  * @retval None
  */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};
  /* USER CODE BEGIN MX_GPIO_Init_1 */

  /* USER CODE END MX_GPIO_Init_1 */

  /* GPIO Ports Clock Enable */
  __HAL_RCC_GPIOC_CLK_ENABLE();
  __HAL_RCC_GPIOF_CLK_ENABLE();
  __HAL_RCC_GPIOA_CLK_ENABLE();
  __HAL_RCC_GPIOB_CLK_ENABLE();

  /*Configure GPIO pin Output Level */
  HAL_GPIO_WritePin(GPIOC, GPIO_PIN_5|GPIO_PIN_6|GPIO_PIN_8, GPIO_PIN_RESET);

  /*Configure GPIO pin : PC13 */
  GPIO_InitStruct.Pin = GPIO_PIN_13;
  GPIO_InitStruct.Mode = GPIO_MODE_IT_RISING;
  GPIO_InitStruct.Pull = GPIO_PULLDOWN;
  HAL_GPIO_Init(GPIOC, &GPIO_InitStruct);

  /*Configure GPIO pins : LPUART1_TX_Pin LPUART1_RX_Pin */
  GPIO_InitStruct.Pin = LPUART1_TX_Pin|LPUART1_RX_Pin;
  GPIO_InitStruct.Mode = GPIO_MODE_AF_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  GPIO_InitStruct.Alternate = GPIO_AF12_LPUART1;
  HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);

  /*Configure GPIO pins : PC5 PC6 PC8 */
  GPIO_InitStruct.Pin = GPIO_PIN_5|GPIO_PIN_6|GPIO_PIN_8;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(GPIOC, &GPIO_InitStruct);

  /*Configure GPIO pin : PC9 */
  GPIO_InitStruct.Pin = GPIO_PIN_9;
  GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
  GPIO_InitStruct.Pull = GPIO_PULLUP;
  HAL_GPIO_Init(GPIOC, &GPIO_InitStruct);

  /* EXTI interrupt init*/
  HAL_NVIC_SetPriority(EXTI15_10_IRQn, 0, 0);
  HAL_NVIC_EnableIRQ(EXTI15_10_IRQn);

  /* USER CODE BEGIN MX_GPIO_Init_2 */

  /* USER CODE END MX_GPIO_Init_2 */
}

/* USER CODE BEGIN 4 */


/**
  * @brief GPIO EXTI Callback
  */
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{

    /* 確認 Interrupt 來自 PC13 */

    if (GPIO_Pin == GPIO_PIN_13)
    {

        /* 只設定 Event Flag */
        STOP = 1;
    }
}


/* USER CODE END 4 */

/**
  * @brief  This function is executed in case of error occurrence.
  * @retval None
  */
void Error_Handler(void)
{
  /* USER CODE BEGIN Error_Handler_Debug */
  /* User can add his own implementation to report the HAL error return state */
  __disable_irq();
  while (1)
  {
  }
  /* USER CODE END Error_Handler_Debug */
}
#ifdef USE_FULL_ASSERT
/**
  * @brief  Reports the name of the source file and the source line number
  *         where the assert_param error has occurred.
  * @param  file: pointer to the source file name
  * @param  line: assert_param error line source number
  * @retval None
  */
void assert_failed(uint8_t *file, uint32_t line)
{
  /* USER CODE BEGIN 6 */
  /* User can add his own implementation to report the file name and line number,
     ex: printf("Wrong parameters value: file %s on line %d\r\n", file, line) */
  /* USER CODE END 6 */
}
#endif /* USE_FULL_ASSERT */

