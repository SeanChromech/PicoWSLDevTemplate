cmake_minimum_required(VERSION 3.12)
set(ENV{PICO_SDK_PATH} ../pico-sdk)
include(pico-sdk/external/pico_sdk_import.cmake)

project(template C CXX ASM)
set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)

# Initialize the SDK
pico_sdk_init()

add_compile_options(-Wall
    -Wno-format
    -Wno-unused-function
    )

add_executable(template main.c)

target_link_libraries(template pico_stdlib)
pico_add_extra_outputs(template)
