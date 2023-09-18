cmake_minimum_required(VERSION 3.16)
set(CMAKE_TOOLCHAIN_FILE ${CMAKE_CURRENT_SOURCE_DIR}/stm32-cmake/cmake/stm32_gcc.cmake)

set(CMAKE_CXX_COMPILER /opt/gcc-arm-none-eabi-10.3-2021.10/bin/arm-none-eabi-g++)
set(CMAKE_C_COMPILER /opt/gcc-arm-none-eabi-10.3-2021.10/bin/arm-none-eabi-gcc)
set(CMAKE_ASM_COMPILER /opt/gcc-arm-none-eabi-10.3-2021.10/bin/arm-none-eabi-gcc)
set(CMAKE_OBJCOPY /opt/gcc-arm-none-eabi-10.3-2021.10/bin/arm-none-eabi-objcopy)
set(CMAKE_OBJDUMP /opt/gcc-arm-none-eabi-10.3-2021.10/bin/arm-none-eabi-objdump)
set(CMAKE_DEBUGER /opt/gcc-arm-none-eabi-10.3-2021.10/bin/arm-none-eabi-gdb)
set(CMAKE_CPPFILT /opt/gcc-arm-none-eabi-10.3-2021.10/bin/arm-none-eabi-c++filt)
set(CMAKE_SIZE /opt/gcc-arm-none-eabi-10.3-2021.10/bin/arm-none-eabi-size)

include_directories(/usr/include)

project(stm32_gcc_cmake CXX C ASM)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_INCLUDE_CURRENT_DIR TRUE)

set(STM32_CUBE_F1_PATH STM32CubeF1)
find_package(CMSIS COMPONENTS STM32F103RC REQUIRED)
find_package(HAL COMPONENTS STM32F1 LL_RCC LL_GPIO REQUIRED)

add_library(ustm32
    src/library.h
    src/library.c
    src/library.cpp)
target_link_libraries(ustm32
    HAL::STM32::F1::LL_RCC
    HAL::STM32::F1::LL_GPIO
    CMSIS::STM32::F103RC

    STM32::NoSys)

add_executable(${PROJECT_NAME} app/main.cpp)
target_link_libraries(${PROJECT_NAME} ustm32)
stm32_print_size_of_target(${PROJECT_NAME})

add_custom_command(
    TARGET  ${PROJECT_NAME}
    COMMAND ${CMAKE_OBJCOPY} -O ihex ${PROJECT_NAME}.elf ${PROJECT_NAME}.hex
    COMMENT "Building ${PROJECT_NAME}.hex")

add_custom_command(
    TARGET  ${PROJECT_NAME}
    POST_BUILD COMMAND ${CMAKE_OBJCOPY} -O binary ${PROJECT_NAME}.elf ${PROJECT_NAME}.bin
    COMMENT "Building ${PROJECT_NAME}.bin")

add_custom_command(
    TARGET  ${PROJECT_NAME}
    POST_BUILD COMMAND ${CMAKE_SIZE}  --format=berkeley ${PROJECT_NAME}.elf ${PROJECT_NAME}.hex
    COMMENT "Invoking: Cross ARM GNU Print Size")
