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

project(Osi LANGUAGES C CXX VERSION 0.108.7)

set(CMAKE_CXX_STANDARD 14)

include(GNUInstallDirs)
set(INSTALL_INCLUDE_DIR ${CMAKE_INSTALL_INCLUDEDIR}/Coin)
set(INSTALL_CMAKE_DIR ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME})

find_package(CoinUtils REQUIRED)
find_package(Mosek)

set(OSI_SOURCES
    src/Osi/OsiAuxInfo.cpp
    src/Osi/OsiBranchingObject.cpp
    src/Osi/OsiChooseVariable.cpp
    src/Osi/OsiColCut.cpp
    src/Osi/OsiCut.cpp
    src/Osi/OsiCuts.cpp
    src/Osi/OsiFeatures.cpp
    src/Osi/OsiNames.cpp
    src/Osi/OsiPresolve.cpp
    src/Osi/OsiRowCut.cpp
    src/Osi/OsiRowCutDebugger.cpp
    src/Osi/OsiSolverBranch.cpp
    src/Osi/OsiSolverInterface.cpp
)

set(OSI_HEADERS
    src/Osi/config_osi_default.h
    src/Osi/OsiAuxInfo.hpp
    src/Osi/OsiBranchingObject.hpp
    src/Osi/OsiChooseVariable.hpp
    src/Osi/OsiColCut.hpp
    src/Osi/OsiCollections.hpp
    src/Osi/OsiConfig.h
    src/Osi/OsiCut.hpp
    src/Osi/OsiCuts.hpp
    src/Osi/OsiFeatures.hpp
    src/Osi/OsiPresolve.hpp
    src/Osi/OsiRowCutDebugger.hpp
    src/Osi/OsiRowCut.hpp
    src/Osi/OsiSolverBranch.hpp
    src/Osi/OsiSolverInterface.hpp
    src/Osi/OsiSolverParameters.hpp
)

add_library(${PROJECT_NAME} ${OSI_SOURCES})

set(MSVC_OPTIONS /bigobj /MP /wd4996 /wd4819 /wd4251 /wd4267 /wd4244 /wd4275)
set(APPLE_OPTIONS
    -Wno-inconsistent-missing-override
    -Wno-unused-command-line-argument
    -Wno-unused-result
    -Wno-exceptions
)

if(MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE ${MSVC_OPTIONS})
    target_compile_definitions(${PROJECT_NAME} PRIVATE _CRT_SECURE_NO_WARNINGS)
endif()

if(APPLE)
    target_compile_options(${PROJECT_NAME} PRIVATE ${APPLE_OPTIONS})
    set(CMAKE_OSX_DEPLOYMENT_TARGET "10.9"
        CACHE STRING "Minimum OS X deployment version")
endif()

target_include_directories(${PROJECT_NAME} PUBLIC $<INSTALL_INTERFACE:${INSTALL_INCLUDE_DIR}> $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/Osi>)

target_compile_definitions(${PROJECT_NAME} PRIVATE HAVE_CMATH)

target_link_libraries(${PROJECT_NAME} PRIVATE Coin::CoinUtils)

install(FILES ${OSI_HEADERS} DESTINATION ${INSTALL_INCLUDE_DIR})

set(INSTALL_TARGETS ${PROJECT_NAME})

if (${MOSEK_FOUND})
    set(OSIMSK_SOURCES
        src/OsiMsk/OsiMskSolverInterface.cpp
    )

    set(OSIMSK_HEADERS
        src/OsiMsk/OsiMskSolverInterface.hpp
        src/OsiMsk/OsiMskConfig.h
    )

    add_library(OsiMsk ${OSIMSK_SOURCES})

    if(MSVC)
        target_compile_options(OsiMsk PRIVATE ${MSVC_OPTIONS})
        target_compile_definitions(OsiMsk PRIVATE _CRT_SECURE_NO_WARNINGS)
    endif()

    if(APPLE)
        target_compile_options(OsiMsk PRIVATE ${APPLE_OPTIONS})
    endif()

    target_include_directories(OsiMsk PUBLIC $<INSTALL_INTERFACE:${INSTALL_INCLUDE_DIR}> $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/OsiMsk> ${MOSEK_INCLUDE})

    target_compile_definitions(OsiMsk PRIVATE HAVE_CMATH)

    target_link_libraries(OsiMsk PRIVATE Coin::CoinUtils ${MOSEK_LIB})

    install(FILES ${OSIMSK_HEADERS} DESTINATION ${INSTALL_INCLUDE_DIR})

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
