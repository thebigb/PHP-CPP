cmake_minimum_required(VERSION 2.6)
project(phpcpp)


# Library version
set(PHPCPP_VERSION_MAJOR 1)
set(PHPCPP_VERSION_MINOR 1)
set(PHPCPP_VERSION_PATCH 1)
set(PHPCPP_VERSION_STRING ${PHPCPP_VERSION_MAJOR}.${PHPCPP_VERSION_MINOR}.${PHPCPP_VERSION_PATCH})

# Add -undefined dynamic_lookup flag for OS X
if(APPLE)
    message(STATUS "Adding fix for OS X")
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -undefined dynamic_lookup")
endif()

# Check if compiler supports C++11 and add -std=c++11 flag
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
if(COMPILER_SUPPORTS_CXX11)
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -std=c++11")
    message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has C++11 support. Moving on.")
else()
    message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

# Add find package modules path
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")

# Find php5. Following variables shoud be set now:
#   PHP5_EXECUTABLE     
#   PHP5_INCLUDES_RAW   
find_package(PHP5 REQUIRED)

# Add PHP5 includes to compiler flags 
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${PHP5_INCLUDES_RAW}")

# Build path
set(CMAKE_BINARY_DIR ${CMAKE_SOURCE_DIR}/build)
set(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR})

# Include directory
set(PROJECT_INCLUDE_DIR ${PROJECT_SOURCE_DIR}/include)

# Source directories
set(COMMON_DIR ${PROJECT_SOURCE_DIR}/common)
set(ZEND_DIR ${PROJECT_SOURCE_DIR}/zend)
set(HHVM_DIR ${PROJECT_SOURCE_DIR}/hhvm)

# Include headers directory
include_directories("${PROJECT_INCLUDE_DIR}")

# Headers
set(HEADERS
${PROJECT_INCLUDE_DIR}/argument.h
${PROJECT_INCLUDE_DIR}/array.h
${PROJECT_INCLUDE_DIR}/arrayaccess.h
${PROJECT_INCLUDE_DIR}/base.h
${PROJECT_INCLUDE_DIR}/byref.h
${PROJECT_INCLUDE_DIR}/byval.h
${PROJECT_INCLUDE_DIR}/call.h
${PROJECT_INCLUDE_DIR}/class.h
${PROJECT_INCLUDE_DIR}/classbase.h
${PROJECT_INCLUDE_DIR}/classtype.h
${PROJECT_INCLUDE_DIR}/countable.h
${PROJECT_INCLUDE_DIR}/exception.h
${PROJECT_INCLUDE_DIR}/extension.h
${PROJECT_INCLUDE_DIR}/global.h
${PROJECT_INCLUDE_DIR}/globals.h
${PROJECT_INCLUDE_DIR}/hashmember.h
${PROJECT_INCLUDE_DIR}/hashparent.h
${PROJECT_INCLUDE_DIR}/hiddenpointer.h
${PROJECT_INCLUDE_DIR}/ini.h
${PROJECT_INCLUDE_DIR}/inivalue.h
${PROJECT_INCLUDE_DIR}/interface.h
${PROJECT_INCLUDE_DIR}/iterator.h
${PROJECT_INCLUDE_DIR}/modifiers.h
${PROJECT_INCLUDE_DIR}/namespace.h
${PROJECT_INCLUDE_DIR}/object.h
${PROJECT_INCLUDE_DIR}/parameters.h
${PROJECT_INCLUDE_DIR}/serializable.h
${PROJECT_INCLUDE_DIR}/streams.h
${PROJECT_INCLUDE_DIR}/super.h
${PROJECT_INCLUDE_DIR}/traversable.h
${PROJECT_INCLUDE_DIR}/type.h
${PROJECT_INCLUDE_DIR}/value.h
${PROJECT_INCLUDE_DIR}/valueiterator.h
)

# Common sources
set(COMMON_SOURCES 
${COMMON_DIR}/modifiers.cpp
${COMMON_DIR}/streambuf.cpp
)

# Zend sources
set(ZEND_SOURCES 
${ZEND_DIR}/base.cpp
${ZEND_DIR}/callable.cpp
${ZEND_DIR}/classbase.cpp
${ZEND_DIR}/classimpl.cpp
${ZEND_DIR}/extension.cpp
${ZEND_DIR}/extensionimpl.cpp
${ZEND_DIR}/global.cpp
${ZEND_DIR}/globals.cpp
${ZEND_DIR}/hashmember.cpp
${ZEND_DIR}/ini.cpp
${ZEND_DIR}/inivalue.cpp
${ZEND_DIR}/iteratorimpl.cpp
${ZEND_DIR}/members.cpp
${ZEND_DIR}/namespace.cpp
${ZEND_DIR}/object.cpp
${ZEND_DIR}/streambuf.cpp
${ZEND_DIR}/streams.cpp
${ZEND_DIR}/super.cpp
${ZEND_DIR}/value.cpp
${ZEND_DIR}/valueiterator.cpp
)

# HHVM sources
set(HHVM_SOURCES 
${HHVM_DIR}/base.cpp
${HHVM_DIR}/callable.cpp
)

# Sources for libphpcpp
set(LIBPHPCPP_SOURCES
${COMMON_SOURCES}
${ZEND_SOURCES}
)

# Sources for libhhvmcpp
set(LIBHHVMCPP_SOURCES
${COMMON_SOURCES}
${HHVM_SOURCES}
)

# Add targets
add_library(${PROJECT_NAME} SHARED ${LIBPHPCPP_SOURCES})
add_library(${PROJECT_NAME}static STATIC ${LIBPHPCPP_SOURCES})
set_target_properties(phpcppstatic PROPERTIES OUTPUT_NAME phpcpp)
#add_library(hhvmcpp SHARED ${LIBHHVMCPP_SOURCES})

# Set version and soname
set_target_properties(${PROJECT_NAME} PROPERTIES VERSION ${PHPCPP_VERSION_STRING} SOVERSION ${PHPCPP_VERSION_MAJOR})

# Installation
install(TARGETS phpcpp LIBRARY DESTINATION lib)
install(TARGETS phpcppstatic ARCHIVE DESTINATION lib)
install(FILES phpcpp.h DESTINATION include)
install(FILES ${HEADERS} DESTINATION include/${PROJECT_NAME})
