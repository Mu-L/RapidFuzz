cmake_minimum_required(VERSION 3.15...3.30)

set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(SKBUILD_LINK_LIBRARIES_KEYWORD PRIVATE)
set(THREADS_PREFER_PTHREAD_FLAG ON)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

project(rapidfuzz LANGUAGES C CXX)

include(CMakePrintHelpers)
cmake_print_variables(CMAKE_AR CMAKE_C_COMPILER_AR CMAKE_CXX_COMPILER_AR)
cmake_print_variables(CMAKE_RANLIB CMAKE_C_COMPILER_RANLIB
                      CMAKE_CXX_COMPILER_RANLIB)

if("${CMAKE_C_COMPILER_AR}" STREQUAL "")
  set(CMAKE_C_COMPILER_AR "${CMAKE_AR}")
endif()
if("${CMAKE_CXX_COMPILER_AR}" STREQUAL "")
  set(CMAKE_CXX_COMPILER_AR "${CMAKE_AR}")
endif()
if("${CMAKE_C_COMPILER_RANLIB}" STREQUAL "")
  set(CMAKE_C_COMPILER_RANLIB "${CMAKE_RANLIB}")
endif()
if("${CMAKE_CXX_COMPILER_RANLIB}" STREQUAL "")
  set(CMAKE_CXX_COMPILER_RANLIB "${CMAKE_RANLIB}")
endif()

if(MSVC)
  add_compile_options(/W4 /bigobj /wd4127)

  # NOTE: _DISABLE_CONSTEXPR_MUTEX_CONSTRUCTOR is temporary. When building on VS
  # 2022 17.10 or newer, but using an older runtime, mutexes can crash
  add_compile_options(/D_DISABLE_CONSTEXPR_MUTEX_CONSTRUCTOR)
else()
  add_compile_options(-Wall -Wextra -pedantic -Wno-psabi)
endif()

if(EMSCRIPTEN)
  add_compile_options(-fexceptions)
  add_link_options(-fexceptions)
endif()

if(CMAKE_VERSION VERSION_LESS 3.18)
  find_package(
    Python
    COMPONENTS Interpreter Development
    REQUIRED)
else()
  set(Python_ARTIFACTS_INTERACTIVE TRUE)
  find_package(
    Python
    COMPONENTS Interpreter Development.Module
    REQUIRED)
endif()

if(CMAKE_VERSION VERSION_LESS 3.17)
  execute_process(
    COMMAND
      "${Python_EXECUTABLE}" -c
      "import sysconfig; print(sysconfig.get_config_var('EXT_SUFFIX').split('.')[1])"
    OUTPUT_VARIABLE Python_SOABI
    OUTPUT_STRIP_TRAILING_WHITESPACE COMMAND_ECHO STDOUT)
  message(STATUS "Corrected SOABI: ${Python_SOABI}")
elseif("${Python_INTERPRETER_ID}" STREQUAL "PyPy")
  message(STATUS "PyPy SOABI: ${Python_SOABI}")
  execute_process(
    COMMAND
      "${Python_EXECUTABLE}" -c
      "import sysconfig; print(sysconfig.get_config_var('EXT_SUFFIX').split('.')[1])"
    OUTPUT_VARIABLE Python_SOABI
    OUTPUT_STRIP_TRAILING_WHITESPACE COMMAND_ECHO STDOUT)
  message(STATUS "Corrected SOABI: ${Python_SOABI}")
endif()

include(FetchContent)
include(CheckCPUArch)

check_cpu_arch_x64(RAPIDFUZZ_ARCH_X64)
check_cpu_arch_x86(RAPIDFUZZ_ARCH_X86)

set(RF_BASE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)

find_package(Taskflow 3.10.0 QUIET)
if(NOT Taskflow_FOUND)
  find_package(Taskflow 3.9.0 QUIET)
endif()
if(NOT Taskflow_FOUND)
  find_package(Taskflow 3.8.0 QUIET)
endif()
if(NOT Taskflow_FOUND)
  find_package(Taskflow 3.7.0 QUIET)
endif()
if(NOT Taskflow_FOUND)
  find_package(Taskflow 3.6.0 QUIET)
endif()
if(NOT Taskflow_FOUND)
  find_package(Taskflow 3.5.0 QUIET)
endif()
if(NOT Taskflow_FOUND)
  find_package(Taskflow 3.4.0 QUIET)
endif()
if(NOT Taskflow_FOUND)
  find_package(Taskflow 3.3.0 QUIET)
endif()
if(Taskflow_FOUND)
  message(STATUS "Using system supplied version of Taskflow")
else()
  message(STATUS "Using packaged version of Taskflow")
  set(TF_BUILD_CUDA
      OFF
      CACHE BOOL "Enables build of CUDA code")
  set(TF_BUILD_TESTS
      OFF
      CACHE BOOL "Enables build of tests")
  set(TF_BUILD_EXAMPLES
      OFF
      CACHE BOOL "Enables build of examples")
  add_subdirectory(extern/taskflow EXCLUDE_FROM_ALL)
  add_library(Taskflow::Taskflow ALIAS Taskflow)
endif()

find_package(rapidfuzz 3.3.2 QUIET)
if(rapidfuzz_FOUND)
  message(STATUS "Using system supplied version of rapidfuzz-cpp")
else()
  message(STATUS "Using packaged version of rapidfuzz-cpp")
  add_subdirectory(extern/rapidfuzz-cpp)
endif()

add_subdirectory(src/rapidfuzz)
add_subdirectory(src/rapidfuzz/distance)
