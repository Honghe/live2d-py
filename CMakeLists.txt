cmake_minimum_required(VERSION 3.16)

project(LAppModelWrapper)

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
  execute_process(
      COMMAND chcp 65001
  )
endif()

# set(CMAKE_BUILD_TYPE "Release")

message("${CMAKE_BUILD_TYPE}")

set(OUTPUT_LIB_NAME LAppModelWrapper)
set(PROJECT_ROOT ${CMAKE_CURRENT_SOURCE_DIR})

# prevent python root dir being shadowed by registry
set(Python3_FIND_REGISTRY "NEVER")

if(DEFINED PYTHON_INSTALLATION_PATH)
    message("Found PYTHON_INSTALLATION_PATH in environment variables")
    set(CMAKE_PREFIX_PATH ${PYTHON_INSTALLATION_PATH})
else()
    # 手动设置 python 路径
    message("Not found PYTHON_INSTALLATION_PATH in environment variables. \nUse default path.")
    set(CMAKE_PREFIX_PATH D:/Python/x64/3.10.0)
endif()

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
  set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
  set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
  set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/static)
  set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /Zi")
  set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} /Zi")
  add_compile_options("/utf-8" "/wd4018" "/wd4244" "/wd4996")
  add_link_options("/NODEFAULTLIB:LIBCMT")
  if (CMAKE_CL_64)
    add_link_options("/BASE:0x800000000")
  endif()
else()
  set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib/${CMAKE_BUILD_TYPE})
  set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE})
  set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/static/${CMAKE_BUILD_TYPE})
endif()  

find_package(Python3 REQUIRED COMPONENTS Development.SABIModule)

add_subdirectory(Main)

# 创建Python扩展模块
add_library(LAppModelWrapper SHARED LAppModelWrapper.cpp)
target_link_libraries(LAppModelWrapper PRIVATE Main Python3::SABIModule)

# Configure for Linux / Windows
if(CMAKE_SYSTEM_NAME MATCHES "Linux")
  set(MODULE_NAME lib${OUTPUT_LIB_NAME}.so)
  set(OUTPUT_NAME live2d.so)
elseif(CMAKE_SYSTEM_NAME MATCHES "Windows")
  set(MODULE_NAME ${OUTPUT_LIB_NAME}.dll)
  set(OUTPUT_NAME live2d.pyd)
elseif(CMAKE_SYSTEM_NAME MATCHES "Darwin")
  set(MODULE_NAME lib${OUTPUT_LIB_NAME}.so)
  set(OUTPUT_NAME live2d.so)
endif()

# Post build commands
add_custom_command(
  TARGET ${OUTPUT_LIB_NAME}
  POST_BUILD
  COMMAND
    ${CMAKE_COMMAND} -E
      copy $<TARGET_FILE_DIR:${OUTPUT_LIB_NAME}>/${MODULE_NAME} ${CMAKE_CURRENT_SOURCE_DIR}/package/live2d/v3/${OUTPUT_NAME}
)
