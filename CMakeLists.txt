cmake_minimum_required(VERSION 3.2)
project(modbus)

set (BUILD_DIR ${CMAKE_CURRENT_BINARY_DIR})
set (DL_DIR ${BUILD_DIR}/src)
set (SRC_DIR ${DL_DIR}/modbus)

set (MODBUS_LIB ${BUILD_DIR}/lib/libmodbus.a)
set (MODBUS_INC ${CMAKE_CURRENT_SOURCE_DIR}/src)

set (CONFIGURE ./configure --prefix=${BUILD_DIR}  --enable-static --disable-shared --disable-tests)
if(CMAKE_SYSTEM_PROCESSOR MATCHES "arm|ARM")
   set (CONFIGURE CC=${CMAKE_C_COMPILER} ${CONFIGURE} --host=arm  )
endif()

# autotools (autogen.sh) are supposed to run from source directory,
# so we copy entire source directory into build directory before configure.
# Default ExternalProject_Add() directory structure is as follows:
# DOWNLOAD_DIR = <prefix>/src          <-- DOWNLOAD_COMMAND runs here
# SOURCE_DIR   = <prefix>/src/<name>   <-- CONFIGURE_COMMAND runs here with BUILD_IN_SOURCE option
# INSTALL_DIR  = <prefix>              <-- BUILD_COMMAND creates lib folder here
include(ExternalProject)
ExternalProject_Add(
   modbus
   PREFIX ${BUILD_DIR}
   SOURCE_DIR ${SRC_DIR}
   DOWNLOAD_DIR ${DL_DIR}
   DOWNLOAD_COMMAND cp -r ${CMAKE_CURRENT_SOURCE_DIR}  .
   CONFIGURE_COMMAND ./autogen.sh COMMAND ${CONFIGURE}
   BUILD_COMMAND ${MAKE}
   BUILD_BYPRODUCTS ${MODBUS_LIB}
   BUILD_IN_SOURCE 1
)

set(autogenerated modbus-version.h)
add_custom_target(modbus_autogenerated_headers
      COMMAND ${CMAKE_COMMAND} -E copy ${BUILD_DIR}/include/modbus/${autogenerated} ${MODBUS_INC}/${autogenerated}
      DEPENDS modbus)

add_library(modbus_wrapper STATIC IMPORTED GLOBAL)
add_dependencies(modbus_wrapper modbus modbus_autogenerated_headers)

set_target_properties(modbus_wrapper PROPERTIES IMPORTED_LOCATION ${MODBUS_LIB})
set_target_properties(modbus_wrapper PROPERTIES INTERFACE_INCLUDE_DIRECTORIES ${MODBUS_INC})