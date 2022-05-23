# Makefile - STM32Cube SDK
#
# Copyright (c) 2016 Linaro Limited
#
# SPDX-License-Identifier: Apache-2.0

#add_library(STM32CUBE INTERFACE)

zephyr_library()

if (CONFIG_USE_UPSTREAM_CUBE)
  if (NOT EXISTS $ENV{STM32CUBEF7_BASE})
    message(FATAL_ERROR
      "Cannot find STM32CubeF7 directory; please set the STM32CUBEF7_BASE env-var")
  endif()
  set(hal_inc $ENV{STM32CUBEF7_BASE}/Drivers/STM32F7xx_HAL_Driver)
  set(cmsis_inc $ENV{STM32CUBEF7_BASE}/Drivers/CMSIS/Device/ST/STM32F7xx/Include)
  set(bsp_inc $ENV{STM32CUBEF7_BASE}/Projects/STM32F723E-Discovery/Examples/BSP/Inc/)
endif()

string(TOUPPER ${CONFIG_SOC} _STM32CUBE_CPU)
string(REPLACE "X" "x" STM32CUBE_CPU ${_STM32CUBE_CPU})

# STM32Cub uses the CPU name to expose SOC-specific attributes of a specific
# peripheral.
# It also requires USE_HAL_DRIVER to be define in order to benefit from
# STM32Cube HAL and LL APIs
zephyr_compile_definitions(
  -D${STM32CUBE_CPU}
  -DUSE_HAL_DRIVER
  -DUSE_FULL_LL_DRIVER
  )

set(stm_socs
  stm32f0x
  stm32f1x
  stm32f2x
  stm32f3x
  stm32f4x
  stm32f7x
  stm32g0x
  stm32g4x
  stm32h7x
  stm32l0x
  stm32l1x
  stm32l4x
  stm32l5x
  stm32mp1x
  stm32u5x
  stm32wbx
  stm32wlx
  )

if(CONFIG_CPU_CORTEX_M4)
  zephyr_compile_definitions( -DCORE_CM4 )
elseif(CONFIG_CPU_CORTEX_M7)
  zephyr_compile_definitions( -DCORE_CM7 )
endif()

if(CONFIG_CLOCK_STM32_HSE_CLOCK)
  zephyr_compile_definitions( -DHSE_VALUE=${CONFIG_CLOCK_STM32_HSE_CLOCK} )
endif()

if(CONFIG_USE_UPSTREAM_CUBE)
  zephyr_include_directories(${hal_inc}/Inc)
  zephyr_include_directories(${hal_inc}/Inc/Legacy)
  zephyr_include_directories(${cmsis_inc})
  zephyr_include_directories(stm32f7xx/drivers/include/upstream) # for stm32f7xx_hal_conf.h
  add_subdirectory(stm32f7xx)
else()
  foreach(stm_soc ${stm_socs})
    string(TOUPPER ${stm_soc} soc_to_upper)
    if(CONFIG_SOC_SERIES_${soc_to_upper})
      zephyr_include_directories(
	${stm_soc}x/soc
	${stm_soc}x/drivers/include
	${stm_soc}x/drivers/include/Legacy
      )

      add_subdirectory(${stm_soc}x)
    endif()
  endforeach()
endif()
  
if(CONFIG_USE_STM32_ASSERT)
	zephyr_compile_definitions( -DUSE_FULL_ASSERT )
endif()

zephyr_include_directories(common_ll/include)
