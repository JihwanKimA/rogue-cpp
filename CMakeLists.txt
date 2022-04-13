# CMakeList.txt : CMake project for rogue-cpp, include source and define
# project specific logic here.
#
cmake_minimum_required (VERSION 3.8)

project ("rogue-cpp")

file(GLOB rogue_src
    "*.h"
    "*.c"
)

# Find curses library and includes based on platform
if(NOT MINGW AND CMAKE_SYSTEM_NAME STREQUAL "Windows" AND
    NOT (CMAKE_SYSTEM_NAME STREQUAL "WindowsStore" OR CMAKE_SYSTEM_NAME STREQUAL "WindowsPhone")) #uwp
    find_path(PDCURSES_INCLUDES curses.h)
    find_library(PDCURSES_LIBRARIES pdcurses REQUIRED)
    set(CURSES_BACKEND "pdcurses")
elseif(MINGW OR NOT CMAKE_SYSTEM_NAME STREQUAL "Windows")
    set(CURSES_NEED_NCURSES TRUE)
    find_package(Curses REQUIRED)
    #It seems that FindCurses can't find debug version cause name has suffix
    find_library(NCURSES_LIBRARIES_DEBUG NAMES ncurses_g)
    set(CURSES_BACKEND "ncurses")
else()
    message(FATAL_ERROR "There are no supported curses implmentation!")
endif()

# Do backend specific
if(CURSES_BACKEND STREQUAL "pdcurses")
    # Generate config.h, it is a only non-intrusive way to make it work 
    # below options are copied from extern.h - WIN32
    option(HAVE_CURSES_H "have curses.h" ON)
    # pdcurses from vcpkg don't have it, and it seems pdcurses don't use it at all
    #option(HAVE_TERM_H "Enable Foo" OFF)
    option(HAVE__SPAWNL "have spawnl" ON)
    option(HAVE_SYS_TYPES_H "have sys_types.h" ON)
    option(HAVE_PROCESS_H "have process.h" ON)
    option(HAVE_ERASECHAR "have erasechar" ON)
    option(HAVE_KILLCHAR "have killchar" ON)
    configure_file(config_cmake.h.in config.h @ONLY)
    include_directories(${CMAKE_CURRENT_BINARY_DIR})

    add_compile_definitions(HAVE_CONFIG_H)
    add_compile_definitions(PDCURSES)
endif()

# Do platform specific
if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
    add_compile_definitions(_CRT_SECURE_NO_WARNINGS)
endif()

# Add compile flags
# It seems flags can be modified through passing variables to configure.
# Before building sophiscated configuration mechanism like original had, we just enable important features for test.
add_compile_definitions(ALLSCORES)
add_compile_definitions(MASTER)
add_compile_definitions(SCOREFILE="rogue.scr")
add_compile_definitions(LOCKFILE="rogue.lck")

# Add source to this project's executable
add_executable (rogue-cpp ${rogue_src})

# Link and include curses
if(CURSES_BACKEND STREQUAL "pdcurses")
    target_include_directories(rogue-cpp PRIVATE ${PDCURSES_INCLUDES})
    target_link_libraries(rogue-cpp PRIVATE ${PDCURSES_LIBRARIES})
elseif(CURSES_BACKEND STREQUAL "ncurses")
    target_include_directories(rogue-cpp PRIVATE ${CURSES_INCLUDE_DIRS})
    # To expose curses.h without ncurses dir
    target_include_directories(rogue-cpp PRIVATE "${CURSES_INCLUDE_DIRS}/ncurses")
    target_link_libraries(rogue-cpp PRIVATE debug ${NCURSES_LIBRARIES_DEBUG} optimized ${CURSES_LIBRARIES})
else()
    message(FATAL_ERROR "Unknown curses backend: '${CURSES_BACKEND}'")
endif()

# TODO: Add tests and install targets if needed.
