#
# @file CMakeLists.txt
# @author Alfonso Castillo Orozco
# @brief This is the main CMake file for Building the MCU Traffic Light project.
# @version 1.0
# @date 2026-02-23
# 
# @copyright Copyright (c) 2026
# 
#

cmake_minimum_required(VERSION 3.28)

# Programmer type for "flash" target (in case of needing it)
set(PROG_TYPE usbasp)

# Variables: AVR Chip
set(MCU attiny4313)
set(F_CPU 1000000UL)
add_compile_definitions(-DF_CPU=${F_CPU})

#====================== Compiler Setup =====================#

# Command names
set(AVRC     avr-gcc)
set(OBJCOPY  avr-objcopy)
set(AVRDUDE  avrdude)

# Compiler flags
set(CSTANDARD     "-std=gnu99")
set(CWARN         "-Wall")
set(CDEBUG        "-g")
set(COPT          "-Os")
set(CMCU          "-mmcu=${MCU}")
set(CDEFS         "-DF_CPU=${F_CPU}")

set(CFLAGS        "${CSTANDARD} ${CWARN} ${CDEBUG} ${COPT} ${CMCU} ${CDEFS}")

# Setting Compiler for C
set(CMAKE_SYSTEM_NAME  Generic)
set(CMAKE_C_COMPILER   ${AVRC})
set(CMAKE_C_FLAGS      "${CFLAGS}")

#====================== Project Setup ======================#

# Project
project(MCU_Traffic_Light C)

# Product filename
set(PRODUCT_NAME main)

# Important project paths
set(INC_PATH     "include")
set(SRC_PATH     "src")

# Files to be compiled
file(GLOB SRC_FILES "${SRC_PATH}/*.c"  "${SRC_PATH}/app/*.c")

# Project setup
include_directories("${INC_PATH}/app")
add_executable(${PRODUCT_NAME} ${SRC_FILES})
set_target_properties(${PRODUCT_NAME} PROPERTIES OUTPUT_NAME "${PRODUCT_NAME}.elf")

#====================== Building Targets ===================#

# Compiling targets: Compiling with Debugging capabilities: main.elf -> main.hex
add_custom_target(compile ALL ${AVRC} "${PRODUCT_NAME}.elf" DEPENDS ${PRODUCT_NAME})
add_custom_target(hex     ALL ${OBJCOPY} -j .text -j .data -O ihex "${PRODUCT_NAME}.elf" "${PRODUCT_NAME}.hex" DEPENDS compile)

# Flashing target: In case of working with a physical MCU (this project is used with SimulIDE)
add_custom_target(flash   ${AVRDUDE} -p ${MCU} -c ${PROG_TYPE} -U flash:w:${PRODUCT_NAME}.hex:i -F -P usb DEPENDS hex)

# Cleaning target: .elf and .hex to be deleted
set_directory_properties(PROPERTIES ADDITIONAL_CLEAN_FILES "a.out;${PRODUCT_NAME}.elf;${PRODUCT_NAME}.hex")
