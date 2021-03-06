CMAKE_MINIMUM_REQUIRED(VERSION 2.8)
SET (CMAKE_VERBOSE_MAKEFILE OFF) 

PROJECT (stm32-template)
include (stm32.cmake)

SET(USB_LIB "" CACHE STRING "")

INCLUDE_DIRECTORIES("src/")
AUX_SOURCE_DIRECTORY ("src/" APP_SOURCES)
AUX_SOURCE_DIRECTORY ("${CUBE_ROOT}/Drivers/STM32F4xx_HAL_Driver/Src/" APP_SOURCES)
LIST (APPEND APP_SOURCES "${STARTUP_CODE}")

# TODO Usunąć, bo to tylko dla STM32F4-DISCOVERY
LIST (APPEND APP_SOURCES "${CUBE_ROOT}/Drivers/BSP/STM32F4-Discovery/stm32f4_discovery.c")
INCLUDE_DIRECTORIES("${CUBE_ROOT}/Drivers/BSP/STM32F4-Discovery/")

ADD_DEFINITIONS (-DUSE_USB_FS)
ADD_DEFINITIONS ("-DHSE_VALUE=${CRYSTAL_HZ}")
INCLUDE_DIRECTORIES("${USB_LIB}/Core/Inc")
INCLUDE_DIRECTORIES("${USB_LIB}/Class/CDC/Inc")
LIST (APPEND APP_SOURCES "${USB_LIB}/Core/Src/usbd_core.c")
LIST (APPEND APP_SOURCES "${USB_LIB}/Core/Src/usbd_ctlreq.c")
LIST (APPEND APP_SOURCES "${USB_LIB}/Core/Src/usbd_ioreq.c")
LIST (APPEND APP_SOURCES "${USB_LIB}/Class/CDC/Src/usbd_cdc.c")

ADD_EXECUTABLE(${CMAKE_PROJECT_NAME}.elf ${APP_SOURCES})
#ADD_CUSTOM_TARGET(${CMAKE_PROJECT_NAME}.hex ALL DEPENDS ${CMAKE_PROJECT_NAME}.elf COMMAND ${CMAKE_OBJCOPY} -Oihex ${CMAKE_PROJECT_NAME}.elf ${CMAKE_PROJECT_NAME}.hex)
ADD_CUSTOM_TARGET(${CMAKE_PROJECT_NAME}.bin ALL DEPENDS ${CMAKE_PROJECT_NAME}.elf COMMAND ${CMAKE_OBJCOPY} -Obinary ${CMAKE_PROJECT_NAME}.elf ${CMAKE_PROJECT_NAME}.bin)

FIND_PROGRAM (ST_FLASH st-flash)
ADD_CUSTOM_TARGET("upload" DEPENDS ${CMAKE_PROJECT_NAME}.elf COMMAND ${ST_FLASH} --reset write ${CMAKE_PROJECT_NAME}.bin 0x8000000)
