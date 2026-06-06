cmake_minimum_required(VERSION 3.20)

if(NOT DEFINED ENV{IDF_PATH})
    message(FATAL_ERROR "IDF_PATH not set — run 'esp' to load the ESP-IDF environment")
endif()

# Download commander source before IDF initializes (FetchContent_Populate
# does download only — does not process commander's CMakeLists.txt).
include(FetchContent)
FetchContent_Declare(commander
    GIT_REPOSITORY https://github.com/gbryant/commander.git
    GIT_TAG        main
)
FetchContent_Populate(commander)

set(COMMANDER_ROOT ${commander_SOURCE_DIR})
list(APPEND EXTRA_COMPONENT_DIRS ${commander_SOURCE_DIR}/runners/esp32)

include($ENV{IDF_PATH}/tools/cmake/project.cmake)
project(cmdr-solar-monitor)
