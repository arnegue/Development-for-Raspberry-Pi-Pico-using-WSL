cmake_minimum_required(VERSION 3.19)
include(pico-sdk/pico_sdk_init.cmake)

project(Development_for_Raspberry_Pi_Pico_using_WSL)

pico_sdk_init()

add_executable(pico_example main.cpp)

target_link_libraries(pico_example pico_stdlib)

pico_add_extra_outputs(pico_example)