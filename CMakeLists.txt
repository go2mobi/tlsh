# Copyright 2013 Trend Micro Incorporated
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

cmake_minimum_required(VERSION 2.6)

project(TLSH)

set(VERSION_MAJOR 3)
set(VERSION_MINOR 2)
set(VERSION_PATCH 1)

# TLSH uses only half the counting buckets.
# It can use all the buckets now.
set(TLSH_BUCKETS_128 2)
if(TLSH_BUCKETS_128 EQUAL 1)
  set(TLSH_HASH "compact hash")
else()
  set(TLSH_HASH "full hash")
  add_definitions(-DBUCKETS_256)
endif()

# TLSH uses 1 byte checksum. The collision rate is 1 in 24.
# It can use 3 bytes checksum now. That collision rate in 1 in 5800.
set(TLSH_CHECKSUM_1B 2)
if(TLSH_CHECKSUM_1B EQUAL 1)
  set(TLSH_CHECKSUM "1 byte checksum")
else()
  set(TLSH_CHECKSUM "3 bytes checksum")
  add_definitions(-DCHECKSUM_3B)
endif()

# write a file with the VERSION information
file(REMOVE VERSION)
file(WRITE VERSION 
  "TLSH version: ${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH} ${TLSH_HASH}, ${TLSH_CHECKSUM}\n")
  
file(REMOVE include/version.h)
file(WRITE include/version.h
  "/****************************************************\n"
  " * This file is generated by cmake.  Modify the top\n"
  " * level CMakeLists.txt to change the VERSION numbers\n"
  " ****************************************************/\n\n"
  "#define VERSION_MAJOR ${VERSION_MAJOR}\n"
  "#define VERSION_MINOR ${VERSION_MINOR}\n"
  "#define VERSION_PATCH ${VERSION_PATCH}\n"
  "#define TLSH_HASH \"${TLSH_HASH}\"\n"
  "#define TLSH_CHECKSUM \"${TLSH_CHECKSUM}\"\n")

if (CMAKE_BUILD_TYPE STREQUAL Debug)
    set(BUILD_POSTFIX "_d")
else(CMAKE_BUILD_TYPE STREQUAL Debug)
    set(BUILD_POSTFIX "")
    if(CMAKE_COMPILER_IS_GNUCXX)
        set(CMAKE_CXX_FLAGS "-O2")        ## Optimize
    endif()
    if(MSVC)
        set(CMAKE_CXX_FLAGS "/O2")        ## Optimize
    endif()
endif(CMAKE_BUILD_TYPE STREQUAL Debug)

enable_testing()

include_directories(include)
add_subdirectory(src)
add_subdirectory(test)
add_subdirectory(utils)
add_subdirectory(Testing)

set(CPACK_GENERATOR "TGZ")
set(CPACK_SOURCE_GENERATOR "TGZ")
set(CPACK_SOURCE_PACKAGE_FILE_NAME "TLSH-${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")
set(CPACK_PACKAGE_VERSION_MAJOR "${VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${VERSION_PATCH}")
set(CPACK_SOURCE_IGNORE_FILES
"~$"
"^${PROJECT_SOURCE_DIR}/bin/"
"^${PROJECT_SOURCE_DIR}/lib/"
"^${PROJECT_SOURCE_DIR}/build/"
"^${PROJECT_SOURCE_DIR}/py_ext/data/"
"^${PROJECT_SOURCE_DIR}/py_ext/build/"
"^${PROJECT_SOURCE_DIR}/Testing/tmp/"
"^${PROJECT_SOURCE_DIR}/CMakeFiles/"
)
include(CPack)

