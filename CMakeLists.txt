cmake_minimum_required(VERSION 3.15)

file(READ "${CMAKE_SOURCE_DIR}/Source/pdxinfo" PDXINFO_CONTENTS)
string(REGEX MATCH "name[ \t]*=[ \t]*([^\n\r]+)" _ ${PDXINFO_CONTENTS})

set(PLAYDATE_GAME_NAME "${CMAKE_MATCH_1}")
set(PLAYDATE_GAME_DEVICE "${PLAYDATE_GAME_NAME}_DEVICE")

project(${PLAYDATE_GAME_NAME} C ASM)
set(CMAKE_C_STANDARD 11)

message(STATUS "Executable: ${PLAYDATE_GAME_NAME}")

# Set configuration types (useful for Xcode or MSVC)
set(CMAKE_CONFIGURATION_TYPES "Debug;Release")
set(CMAKE_XCODE_GENERATE_SCHEME TRUE)

# Determine SDK path
if (DEFINED ENV{PLAYDATE_SDK_PATH} AND NOT "$ENV{PLAYDATE_SDK_PATH}" STREQUAL "")
	file(TO_CMAKE_PATH "$ENV{PLAYDATE_SDK_PATH}" SDK)
else()
	execute_process(
		COMMAND bash -c "egrep '^\s*SDKRoot' $HOME/.Playdate/config | head -n 1 | cut -c9-"
		OUTPUT_VARIABLE SDK
		OUTPUT_STRIP_TRAILING_WHITESPACE
	)
endif()

if (NOT EXISTS "${SDK}")
	message(FATAL_ERROR "SDK Path not found; set ENV value PLAYDATE_SDK_PATH")
endif()

# Collect all .c source files
file(GLOB SOURCES "Source/*.c")

# Optional toolchain flag
set(TOOLCHAIN "" CACHE STRING "Toolchain to use (e.g., armgcc)")

if (TOOLCHAIN STREQUAL "armgcc")
	add_executable(${PLAYDATE_GAME_DEVICE} ${SOURCES})
else()
	add_library(${PLAYDATE_GAME_NAME} SHARED ${SOURCES})
endif()

# Include SDK helper
include("${SDK}/C_API/buildsupport/playdate_game.cmake")