# A CMake-template for cross-compiling for STM32 (or other) ARM Cortex-M MCUs.
# Copyright (C) 2016  Andreas Hofmann
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 2 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License along
# with this program; if not, write to the Free Software Foundation, Inc.,
# 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.

cmake_minimum_required(VERSION 2.8.12)

project(STM32-Template C CXX ASM)
set(TARGET stm32)

# General settings {{{

set(CROSS_TARGET_TRIPLET "arm-none-eabi-")
set(BASE_DIR ${CMAKE_CURRENT_LIST_DIR}/)

# Fill in the target CPU here
set(CPU "cortex-m4")

# Fill in the linker script to use here
set(LINKER_SCRIPT "${BASE_DIR}/scripts/linker.ld")

# Fill in the config for openocd to use.
# E.g. for an STM32F4-Discovery dev-board:
set(OPENOCD_CFG "/usr/share/openocd/scripts/board/stm32f4discovery.cfg")

# Fill in the address offset for the device flash for programming.
# Example value for an STM32F4:
set(FLASH_OFFSET "0x08000000")

# Definitions {{{

# We're not using add_definitions() here, since this would break the calls to the assembler.
# Some examples:
set(DEFINITIONS "${DEFINITIONS} -DSTM32F40_41xxx")
set(DEFINITIONS "${DEFINITIONS} -DUSE_STDPERIPH_DRIVER")

#}}}

# Compiler settings {{{

# Prepend the cross-compiler prefix to the used tools.
set(CMAKE_C_COMPILER "${CROSS_TARGET_TRIPLET}gcc")
set(CMAKE_CXX_COMPILER "${CROSS_TARGET_TRIPLET}g++")
set(CMAKE_ASM_COMPILER "${CROSS_TARGET_TRIPLET}as")

# Be verbose when compiling.
set(WARNING_FLAGS "-Wall -Wextra -Wpedantic")

# Flags which are required for gcc, g++ and as.
set(COMMON_COMPILE_FLAGS "-mcpu=${CPU} -mthumb")

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${COMMON_COMPILE_FLAGS} ${WARNING_FLAGS} ${DEFINITIONS}")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${COMMON_COMPILE_FLAGS} ${WARNING_FLAGS} ${DEFINITIONS}")
set(CMAKE_ASM_FLAGS "${CMAKE_ASM_FLAGS} ${COMMON_COMPILE_FLAGS}")

set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -T ${LINKER_SCRIPT}")

# The following is required to get rid of the (not supported) -rdynamic flag
# when linking the final binary.
set(CMAKE_SHARED_LIBRARY_LINK_C_FLAGS "" )
set(CMAKE_SHARED_LIBRARY_LINK_CXX_FLAGS "" )

#}}}

# Source files  {{{

# Fill in your source files here (duh).
set(SOURCES
    "${BASE_DIR}/src/main.c"
)

#}}}

# Header files  {{{

# Fill in your header files here. Not actually required for compiling, but
# some IDEs (e.g. QtCreator) won't list the header files in the project tree
# if they are not explicitly listed.
set(HEADERS
    "${BASE_DIR}include/main.h"
)

#}}}

# Include dirs  {{{

# Fill in the directories of your header files here.
include_directories(
    "${BASE_DIR}/include"
)

#}}}

# Build the image {{{

# Compile the sources to an .elf
add_executable(${TARGET}.elf ${SOURCES} ${HEADERS})

# Create a binary from the elf
add_custom_command(TARGET ${TARGET}.elf POST_BUILD
    COMMAND ${CROSS_TARGET_TRIPLET}objcopy -O binary ${TARGET}.elf ${TARGET}.bin
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
)

#}}}

# Targets for debugging/flashing {{{

# Start debugging with GDB (openocd needs to run in background)
add_custom_target(gdb
    COMMAND ${CROSS_TARGET_TRIPLET}gdb -ex 'target remote localhost:3333' -ex 'monitor reset init' ${TARGET}.elf
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    DEPENDS ${TARGET}.elf
)

# Flash the previously built firmware to the target (openocd needs to run in background)
add_custom_target(flash
    COMMAND ${CROSS_TARGET_TRIPLET}gdb --batch -ex 'target remote localhost:3333' -ex 'monitor reset init' -ex 'monitor program ${TARGET}.bin verify reset ${FLASH_OFFSET}'
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    DEPENDS ${TARGET}.elf
)

# Flash the previously built firmware to the target (openocd needs to run in background)
add_custom_target(openocd
    COMMAND openocd -f ${OPENOCD_CFG}
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    DEPENDS ${TARGET}.elf
)

#}}}

# vim: set ts=4 sw=4 sts=4 et smarttab :
