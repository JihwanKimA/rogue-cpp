# CMakeList.txt : CMake project for rogue-cpp, include source and define
# project specific logic here.
#
cmake_minimum_required (VERSION 3.8)

project ("rogue-cpp")
 
file(GLOB rogue_src
    "*.h"
    "*.c"
)

if(WIN32)
    set(CURSES_BACKEND "pdcurses")
else()
    set(CURSES_BACKEND "ncurses")
endif()

if(CURSES_BACKEND STREQUAL "pdcurses")
    find_path(PDCURSES_INCLUDES curses.h)
    find_library(PDCURSES_LIBRARIES pdcurses REQUIRED)

    # generate config.h, it is only way to make it work with non-intrusive way
    #    copied from extern.h - WIN32
    option(HAVE_CURSES_H "have curses.h" ON)
    #    pdcurses from vcpkg don't have it, and it seems unncessary
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

    add_definitions(-D_CRT_SECURE_NO_WARNINGS)
elseif(CURSES_BACKEND STREQUAL "ncurses")
    find_package(Curses REQUIRED)
    include_directories(${CURSES_INCLUDE_DIR})
    include_directories("${CURSES_INCLUDE_DIR}/ncurses")

    #add_compile_definitions(NCURSES_INTERNALS)
    add_compile_definitions(HAVE_NCURSES_TERM_H)
else()
    message(FATAL_ERROR "Unknown curses backend: '${CURSES_BACKEND}'")
endif()

# Add source to this project's executable.
add_executable (rogue-cpp ${rogue_src})

if(CURSES_BACKEND STREQUAL "pdcurses")
    target_include_directories(rogue-cpp PRIVATE ${PDCURSES_INCLUDES})
    target_link_libraries(rogue-cpp PRIVATE ${PDCURSES_LIBRARIES})
elseif(CURSES_BACKEND STREQUAL "ncurses")
    target_link_libraries(rogue-cpp PRIVATE ${CURSES_LIBRARIES})
else()
    message(FATAL_ERROR "Unknown curses backend: '${CURSES_BACKEND}'")
endif()

# TODO: Add tests and install targets if needed.
