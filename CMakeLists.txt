cmake_minimum_required(VERSION 2.8.12)
project(incremental_pidgeon_hole_principle)

# You can use the cmake builtin 'cmake -LH' to list all options.

# Options
set(USER_CXX_FLAGS "" CACHE STRING "Use this variable to provide additional\
flags to the compiler.")
set(IPASIR_DIR "${CMAKE_SOURCE_DIR}/external/lib/ipasir" CACHE STRING
"The directory which contains one or more implemntations to the ipasir \
interface. The libs should have the format 'libipasir[solver-name].a'")

# Basic Settings
set(CMAKE_CXX_FLAGS "-std=c++14 -O3 -Wall -Wextra -Wpedantic -pthread -g ${USER_CXX_FLAGS}")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -static-libstdc++")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin)
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/external/cmake")

include_directories(${CMAKE_SOURCE_DIR}/src)
include_directories(${CMAKE_SOURCE_DIR}/external/include)

set(SRC_FILES
		src/carj/carj.cpp
		src/incphp.cpp
	)

set(UNIT_TEST_FILES
		test/TestBasic.cpp
	)

add_library(ipasir_wrapper external/include/ipasir/ipasir_cpp.cpp)
add_library(all_sources ${SRC_FILES})

# Include google test, our testing framework
include(gtest)

# === Target: unitTest ===
add_executable(unitTest ${UNIT_TEST_FILES})
target_link_libraries(unitTest
	all_sources
	ipasir_wrapper
	gtest_main
	gmock_main
)
set_target_properties(unitTest PROPERTIES
	COTIRE_PREFIX_HEADER_IGNORE_PATH "${CMAKE_SOURCE_DIR}"
		_PREFIX_HEADER_INCLUDE_PATH "${CMAKE_SOURCE_DIR}/libs")

# === Target: runtest ===
add_custom_target(runTest
	COMMAND ./unitTest --gtest_color=yes
	WORKING_DIRECTORY ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}
	DEPENDS unitTest)

# === Target: incphp-[solver_name] ===

# Creates executables for each aviable solver [solver-name]

# This folder should contain the library of solvers to link aigainst. Each
# incremental solver library should be named libipasir[solver-name].a
file(GLOB ipasir_libs RELATIVE ${IPASIR_DIR}
	${IPASIR_DIR}/*.a
	)

FOREACH(ipasir_lib ${ipasir_libs})
	string(REGEX REPLACE "libipasir(.*).a" "\\1" libname ${ipasir_lib})
	add_executable(incphp-${libname} src/main.cpp)
	target_link_libraries(incphp-${libname}
		all_sources
		ipasir_wrapper
		${IPASIR_DIR}/${ipasir_lib}
		)
ENDFOREACH()

# === Target: core ===

# Dummy target which builds all targets but only for one solver
add_custom_target(core DEPENDS unitTest runTest)
if (TARGET incphp-cominisatps2sun_nopre)
	add_dependencies(core incphp-cominisatps2sun_nopre)

	# Make sure the tool is build, even if tests fail.
	add_dependencies(runTest incphp-cominisatps2sun_nopre)
endif()