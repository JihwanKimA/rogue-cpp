# CMakeList.txt : CMake project for rogue-cpp, include source and define
# project specific logic here.
#
cmake_minimum_required (VERSION 3.8)

project ("rogue-cpp")
 
file(GLOB rogue_src
    "*.h"
    "*.c"
)

find_package( Curses REQUIRED )
include_directories( ${CURSES_INCLUDE_DIR} )

add_compile_definitions(NCURSES_INTERNALS)

# Add source to this project's executable.
add_executable (rogue-cpp ${rogue_src})

target_link_libraries(rogue-cpp ${CURSES_LIBRARIES})

# TODO: Add tests and install targets if needed.
