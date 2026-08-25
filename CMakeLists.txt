cmake_minimum_required(VERSION 3.20)

if(NOT DEFINED ENV{IDF_PATH})
    message(FATAL_ERROR "IDF_PATH not set — run 'esp' to load the ESP-IDF environment")
endif()

# Download commander source before IDF initializes — download ONLY (don't process
# commander's Pico CMakeLists). SOURCE_SUBDIR=include has no CMakeLists.txt, so
# MakeAvailable populates the source but skips add_subdirectory (non-deprecated Populate).
include(FetchContent)
FetchContent_Declare(commander
    GIT_REPOSITORY https://github.com/gbryant/commander.git
    GIT_TAG        v1.1     # release pin — move with `cmdr pin <tag>`
    SOURCE_SUBDIR  include
)
FetchContent_MakeAvailable(commander)

set(COMMANDER_ROOT ${commander_SOURCE_DIR})
list(APPEND EXTRA_COMPONENT_DIRS ${commander_SOURCE_DIR}/runners/esp32)

include($ENV{IDF_PATH}/tools/cmake/project.cmake)
project(cmdr-solar-monitor)
