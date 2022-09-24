# Copyright (c) Microsoft Corporation
#
# All rights reserved.
#
# MIT License
#
# Permission is hereby granted, free of charge, to any person obtaining a copy of
# this software and associated documentation files (the "Software"), to deal in
# the Software without restriction, including without limitation the rights to
# use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies
# of the Software, and to permit persons to whom the Software is furnished to do
# so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED *AS IS*, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.

cmake_minimum_required(VERSION 3.11)

project(Osi LANGUAGES C CXX)

set(PROJECT_VERSION 0.107.9)
set(PROJECT_VERSION_MAJOR 0)
set(PROJECT_VERSION_MINOR 107)
set(PROJECT_VERSION_PATCH 9)

set(CMAKE_CXX_STANDARD 14)

include(GNUInstallDirs)
set(INSTALL_INCLUDE_DIR ${CMAKE_INSTALL_INCLUDEDIR}/Coin)
set(INSTALL_CMAKE_DIR ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME})

if(MSVC)
  set(
    CMAKE_CXX_FLAGS
    "${CMAKE_CXX_FLAGS} /bigobj /MP /wd4996 /wd4819 /wd4251 /wd4267 /wd4244 /wd4275"
    )
endif()

if(APPLE)
  set(
    CMAKE_CXX_FLAGS
    "${CMAKE_CXX_FLAGS} -Wno-inconsistent-missing-override -Wno-unused-command-line-argument -Wno-unused-result -Wno-exceptions"
    )
  set(CMAKE_OSX_DEPLOYMENT_TARGET "10.9"
      CACHE STRING "Minimum OS X deployment version")
endif()

find_package(CoinUtils REQUIRED)
find_package(Mosek)

file(GLOB OSI_SOURCES src/Osi/*.hpp src/Osi/*.cpp)

add_library(${PROJECT_NAME} ${OSI_SOURCES})

target_include_directories(${PROJECT_NAME} PUBLIC $<INSTALL_INTERFACE:${INSTALL_INCLUDE_DIR}> $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/Osi>)

if(MSVC)
  target_compile_definitions(${PROJECT_NAME} PRIVATE _CRT_SECURE_NO_WARNINGS)
endif()
target_compile_definitions(${PROJECT_NAME} PRIVATE HAVE_CMATH)

target_link_libraries(${PROJECT_NAME} PRIVATE Coin::CoinUtils)

install(DIRECTORY src/Osi/
        DESTINATION ${INSTALL_INCLUDE_DIR}
        FILES_MATCHING
        PATTERN "*.h"
        PATTERN "*.hpp"
        PATTERN "config_default.h" EXCLUDE)

set(INSTALL_TARGETS ${PROJECT_NAME})

if (${MOSEK_FOUND})
    file(GLOB OSIMSK_SOURCES src/OsiMsk/*.hpp src/OsiMsk/*.cpp)

    add_library(OsiMsk ${OSIMSK_SOURCES})
    add_library(Coin::OsiMsk ALIAS OsiMsk)

    target_include_directories(OsiMsk PUBLIC $<INSTALL_INTERFACE:${INSTALL_INCLUDE_DIR}> $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/OsiMsk> ${MOSEK_INCLUDE})

    if(MSVC)
        target_compile_definitions(OsiMsk PRIVATE _CRT_SECURE_NO_WARNINGS)
    endif()
    target_compile_definitions(OsiMsk PRIVATE HAVE_CMATH)

    target_link_libraries(OsiMsk PRIVATE Coin::CoinUtils ${MOSEK_LIB})

    install(DIRECTORY src/OsiMsk/
            DESTINATION ${INSTALL_INCLUDE_DIR}
            FILES_MATCHING
            PATTERN "*.h"
            PATTERN "*.hpp")

    list(APPEND INSTALL_TARGETS OsiMsk)
endif()

install(TARGETS ${INSTALL_TARGETS}
        EXPORT "${PROJECT_NAME}Targets"
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
        RUNTIME DESTINATION bin)

include(CMakePackageConfigHelpers)

set(version_config "${CMAKE_CURRENT_BINARY_DIR}/temp/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${CMAKE_CURRENT_BINARY_DIR}/temp/${PROJECT_NAME}Config.cmake")
set(namespace "Coin::")

write_basic_package_version_file("${version_config}"
                                 COMPATIBILITY SameMajorVersion)

configure_package_config_file("Config.cmake.in"
                              "${project_config}"
                              INSTALL_DESTINATION
                              "${INSTALL_CMAKE_DIR}")

install(FILES "${project_config}" "${version_config}"
        DESTINATION "${INSTALL_CMAKE_DIR}")

install(EXPORT "${PROJECT_NAME}Targets"
        NAMESPACE "${namespace}"
        DESTINATION "${INSTALL_CMAKE_DIR}")
