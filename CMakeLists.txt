# Base EFM32 CMake file
#
# This can be used as is as a project base, or by adding the efm32-base
# repository as a submodule to another project, copying this CMakeLists file
# to the top level directory, and updating the BASE_LOCATION variable to reflect this
# change
#
## Copyright (c) 2016 Ryan Kurte
# This file is covered under the MIT license available at: https://opensource.org/licenses/MIT

###### Project Environment #####

# Set minimum CMake version
cmake_minimum_required(VERSION 2.8.4)

# Optional verbose mode, uncomment or pass in -DCMAKE_VERBOSE_MAKEFILE=ON
# set(CMAKE_VERBOSE_MAKEFILE ON)

set(BASE_LOCATION efm32-base)

# Set the compiler (must be prior to project setup)
include(${BASE_LOCATION}/toolchain/arm-gcc.cmake)

##### Project Setup #####

# Configure project and languages
project(AudioMoth-Project C CXX ASM)

# Set device
if (NOT DEVICE)
    set(DEVICE EFM32WG380F256)
endif ()

# Set build
if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE DEBUG)
endif ()

##### Modules #####

# Libraries can be added to the LIBS variable
# or manually included here.

set(OPTIONAL_DEBUG_SYMBOLS "-DARM_MATH_CM4=1 -mfpu=fpv4-sp-d16 -mfloat-abi=hard")

# Add base libs (emlib, CMSIS, device files)
include(${BASE_LOCATION}/toolchain/efm32-base.cmake)

add_subdirectory(${BASE_LOCATION}/cmsis)
add_subdirectory(${BASE_LOCATION}/emlib)
add_subdirectory(${BASE_LOCATION}/device)
add_subdirectory(${BASE_LOCATION}/protocol)
add_subdirectory(${BASE_LOCATION}/hardware)

add_subdirectory(fatfs)

include(./usb.cmake)

# This is normally set in efm32-base.cmake, but libraries may modify it so set
# it after libraries/subdirectories have been added
set(CMAKE_EXE_LINKER_FLAGS "${COMMON_DEFINITIONS} -Xlinker -T${LINKER_SCRIPT} -Wl,-Map=${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_PROJECT_NAME}.map -Wl,--gc-sections -Wl,-v")

##### Files #####

# Add project headers
include_directories(inc)

# Generate executable and link
add_executable(${PROJECT_NAME}
        src/audioMoth.c
        src/main.c)

set(FLASH_ORIGIN 0x4000)
set(FLASH_LENGTH 0x3C000)
efm32_configure_linker_addresses(${PROJECT_NAME})

target_link_libraries(${PROJECT_NAME} ${LIBS} emlib cmsis device fatfs)

# Link optional libraries if available
if (${HAS_HARDWARE})
    target_link_libraries(${PROJECT_NAME} hardware)
endif ()

if (${HAS_PROTOCOL})
    target_link_libraries(${PROJECT_NAME} protocol)
endif ()

##### Post build #####

# Add post build commands
include(${BASE_LOCATION}/toolchain/post-build.cmake)

# Add JLink commands
include(${BASE_LOCATION}/toolchain/jlink.cmake)

##### CMake debug prints #####
if (CMAKE_VERBOSE_MAKEFILE)
    print_debug_info()
endif()
