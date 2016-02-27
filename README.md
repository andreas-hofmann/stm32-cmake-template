# stm32-cmake-template
A CMake template file for cross-compiling to an STM32 (or other ARM Cortex MCUs)

This CMakeLists.txt shall serve as starting point when setting up a build
environment for a Cortex-M CPU. It also adds some convenient targets for
flashing the firmware and debugging.

It is set up that either C, C++ or startup-code in assembly can be compiled by
just setting up the correct CPU target, adding the corresponding source files
and a linker script.
