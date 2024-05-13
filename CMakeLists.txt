###
# Cmake build script for urglib
# Put me to the source directory of urglib.
# 
# urglib用Cmakeビルドスクリプト
# urglibのソースディレクトリに置いてください．
# 
# Date: 2024.05.13
# Author: SUZUKI Ryota
# Copyright 2024 SUZUKI Ryota. All rights researved.
# This file is released under the MIT license.
# https://opensource.org/license/mit
###

cmake_minimum_required(VERSION 2.8.12)

project(urglib)

set_property(GLOBAL PROPERTY USE_FOLDERS ON)

if(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
    set(CMAKE_INSTALL_PREFIX ${CMAKE_CURRENT_SOURCE_DIR}/installed CACHE PATH "Directory for installation" FORCE)
endif()
set(CMAKE_DEBUG_POSTFIX _d)


set(BUILD_SHARED_LIBS OFF CACHE BOOL "Build shared library instead of static library (not available on MSVC)")
set(BUILD_SAMPLES_C OFF CACHE BOOL "Build C examples")
set(BUILD_SAMPLES_CPP OFF CACHE BOOL "Build CPP examples")

if(MSVC AND BUILD_SHARED_LIBS)
    message(SEND_ERROR "Shared library is not available on MSVC.")
    set(BUILD_SHARED_LIBS OFF CACHE BOOL "Build shared library instead of static library (not available on MSVC)" FORCE)
endif()

add_library(urg_obj OBJECT
    src/urg_connection.c
    src/urg_debug.c
    src/urg_ring_buffer.c
    src/urg_sensor.c
    src/urg_serial.c
    #src/urg_serial_windows.c
    #src/urg_serial_linux.c
    src/urg_serial_utils.c
    #src/urg_serial_utils_linux.c
    #src/urg_serial_utils_windows.c
    src/urg_tcpclient.c
    src/urg_utils.c
)
target_include_directories(urg_obj
    PRIVATE src
    #PUBLIC include/c
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/c>
        $<INSTALL_INTERFACE:include/urglib/c>
)
if(MSVC)
    target_link_libraries(urg_obj
        INTERFACE ws2_32
        INTERFACE SetupAPI
    )
endif()

add_library(urg STATIC $<TARGET_OBJECTS:urg_obj>)
target_include_directories(urg
    PRIVATE src
    #PUBLIC include/c
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/c>
        $<INSTALL_INTERFACE:include/urglib/c>
)
if(MSVC)
    target_link_libraries(urg
        INTERFACE ws2_32
        INTERFACE SetupAPI
    )
endif()

add_library(urg_cpp STATIC
    $<TARGET_OBJECTS:urg_obj>
    src/Urg_driver.cpp
    src/ticks.cpp
)
target_link_libraries(urg_cpp
    PRIVATE urg
)
target_include_directories(urg_cpp
    PRIVATE src
    PRIVATE include/c
    #PUBLIC include/cpp
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/cpp>
        $<INSTALL_INTERFACE:include/urglib/cpp>
)

set_target_properties(urg_obj PROPERTIES FOLDER "Libraries")
set_target_properties(urg PROPERTIES FOLDER "Libraries")
set_target_properties(urg_cpp PROPERTIES FOLDER "Libraries")

if(BUILD_SAMPLES_C)
    add_executable(calculate_xy
        samples/c/calculate_xy.c
        samples/c/open_urg_sensor.c
    )
    add_executable(find_port
        samples/c/find_port.c
        samples/c/open_urg_sensor.c
    )
    add_executable(get_distance
        samples/c/get_distance.c
        samples/c/open_urg_sensor.c
    )
    add_executable(get_distance_intensity
        samples/c/get_distance_intensity.c
        samples/c/open_urg_sensor.c
    )
    add_executable(get_multiecho
        samples/c/get_multiecho.c
        samples/c/open_urg_sensor.c
    )
    add_executable(get_multiecho_intensity
        samples/c/get_multiecho_intensity.c
        samples/c/open_urg_sensor.c
    )
    add_executable(sensor_parameter
        samples/c/sensor_parameter.c
        samples/c/open_urg_sensor.c
    )
    add_executable(sync_time_stamp
        samples/c/sync_time_stamp.c
        samples/c/open_urg_sensor.c
    )
    
    target_link_libraries(calculate_xy PRIVATE urg)
    target_link_libraries(find_port PRIVATE urg)
    target_link_libraries(get_distance PRIVATE urg)
    target_link_libraries(get_distance_intensity PRIVATE urg)
    target_link_libraries(get_multiecho PRIVATE urg)
    target_link_libraries(get_multiecho_intensity PRIVATE urg)
    target_link_libraries(sensor_parameter PRIVATE urg)
    target_link_libraries(sync_time_stamp PRIVATE urg)

    set_target_properties(calculate_xy PROPERTIES FOLDER "Samples_C")
    set_target_properties(find_port PROPERTIES FOLDER "Samples_C")
    set_target_properties(get_distance PROPERTIES FOLDER "Samples_C")
    set_target_properties(get_distance_intensity PROPERTIES FOLDER "Samples_C")
    set_target_properties(get_multiecho PROPERTIES FOLDER "Samples_C")
    set_target_properties(get_multiecho_intensity  PROPERTIES FOLDER "Samples_C")
    set_target_properties(sensor_parameter PROPERTIES FOLDER "Samples_C")
    set_target_properties(sync_time_stamp PROPERTIES FOLDER "Samples_C")
endif()
if(BUILD_SAMPLES_CPP)
    add_executable(get_distance_handshake_cpp
        samples/cpp/get_distance_handshake.cpp
        samples/cpp/Connection_information.cpp
    )
    add_executable(get_distance_intensity_cpp
        samples/cpp/get_distance_intensity.cpp
        samples/cpp/Connection_information.cpp
    )
    add_executable(get_distance_cpp
        samples/cpp/get_distance.cpp
        samples/cpp/Connection_information.cpp
    )
    add_executable(get_multiecho_intensity_cpp
        samples/cpp/get_multiecho_intensity.cpp
        samples/cpp/Connection_information.cpp
    )
    add_executable(get_multiecho_cpp
        samples/cpp/get_multiecho.cpp
        samples/cpp/Connection_information.cpp
    )
    add_executable(sensor_parameter_cpp
        samples/cpp/sensor_parameter.cpp
        samples/cpp/Connection_information.cpp
    )
    add_executable(sync_time_stamp_cpp
        samples/cpp/sync_time_stamp.cpp
        samples/cpp/Connection_information.cpp
    )

    target_link_libraries(get_distance_handshake_cpp PRIVATE urg_cpp)
    target_link_libraries(get_distance_intensity_cpp PRIVATE urg_cpp)
    target_link_libraries(get_distance_cpp PRIVATE urg_cpp)
    target_link_libraries(get_multiecho_intensity_cpp PRIVATE urg_cpp)
    target_link_libraries(get_multiecho_cpp PRIVATE urg_cpp)
    target_link_libraries(sensor_parameter_cpp PRIVATE urg_cpp)
    target_link_libraries(sync_time_stamp_cpp PRIVATE urg_cpp)

    set_target_properties(get_distance_handshake_cpp PROPERTIES FOLDER "Samples_CPP")
    set_target_properties(get_distance_intensity_cpp PROPERTIES FOLDER "Samples_CPP")
    set_target_properties(get_distance_cpp PROPERTIES FOLDER "Samples_CPP")
    set_target_properties(get_multiecho_intensity_cpp PROPERTIES FOLDER "Samples_CPP")
    set_target_properties(get_multiecho_cpp PROPERTIES FOLDER "Samples_CPP")
    set_target_properties(sensor_parameter_cpp PROPERTIES FOLDER "Samples_CPP")
    set_target_properties(sync_time_stamp_cpp PROPERTIES FOLDER "Samples_CPP")
endif()

install(TARGETS urg
    EXPORT urg-export
    LIBRARY DESTINATION lib
)
install(EXPORT urg-export
    FILE urg-config.cmake
    DESTINATION cmake
    EXPORT_LINK_INTERFACE_LIBRARIES
)

install(TARGETS urg_cpp
    EXPORT urg_cpp-export
    LIBRARY DESTINATION lib
)
install(EXPORT urg_cpp-export
    FILE urg_cpp-config.cmake
    DESTINATION cmake
    EXPORT_LINK_INTERFACE_LIBRARIES
)

install(DIRECTORY
    ${CMAKE_CURRENT_SOURCE_DIR}/include/
    DESTINATION include/urglib
)
