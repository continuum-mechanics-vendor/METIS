cmake_minimum_required(VERSION 3.26)

#[==============================================================================================[
#                                   Basic project definition                                   #
]==============================================================================================]

list(APPEND CMAKE_MESSAGE_CONTEXT METIS)
project(METIS
  VERSION 5.3.0
  DESCRIPTION "Serial Graph Partitioning and Fill-reducing Matrix Ordering"
  HOMEPAGE_URL https://github.com/KarypisLab/METIS
  LANGUAGES C
)

# Specify C standard
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS ON)

if (NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif ()

#[==============================================================================================[
#                                            Options                                            #
]==============================================================================================]

include(CMakeDependentOption)
include(FeatureSummary)

option(METIS_TESTS "METIS: Build unit tests" OFF)
option(METIS_SHARED_LIBS "METIS: Build as a shared library" ${CMAKE_BUILD_SHARED})
option(METIS_INSTALL "METIS: Install project" ON)
option(METIS_OpenMP "METIS: Enable OpenMP support" OFF)
add_feature_info(METIS_OpenMP METIS_OpenMP "OpenMP support")
option(METIS_PROGRAMS "METIS: Build executable programs" OFF)
add_feature_info(METIS_PROGRAMS METIS_PROGRAMS "Build executable programs")
option(METIS_IDX64 "METIS: Build with 64 bit ints" OFF)
add_feature_info(METIS_IDX64 METIS_IDX64 "Build with 64 bit ints")
option(METIS_REAL64 "METIS: Build with 64 bit floats" ON)
add_feature_info(METIS_REAL64 METIS_REAL64 "Build with 64 bit floats")

#[==============================================================================================[
#                                     Project configuration                                     #
]==============================================================================================]

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)

# Add some helper functions
include(METIS)

# Include basic tools
include(FetchContent)
if (METIS_INSTALL)
  include(CMakePackageConfigHelpers)
  if (UNIX)
    include(GNUInstallDirs)
  endif ()
endif ()

# Define basic parameters
set(BUILD_SHARED_LIBS ${METIS_SHARED_LIBS})


#[==============================================================================================[
#                                       External packages                                       #
]==============================================================================================]

set(gklib_find_package_args CONFIG)
if (METIS_OpenMP)
  if (NOT COMPONENTS IN_LIST gklib_find_package_args)
    list(APPEND gklib_find_package_args COMPONENTS)
  endif ()
  list(APPEND gklib_find_package_args OpenMP)
endif ()

find_package(GKlib)
# FetchContent_Declare(GKlib
#   GIT_REPOSITORY https://github.com/LecrisUT/GKlib
#   GIT_TAG cmake/modernization
#   FIND_PACKAGE_ARGS ${gklib_find_package_args}
# )
# METIS_FetchContent_MakeAvailable(GKlib)

include(METIS_PackagesInfo)

feature_summary(
  FILENAME ${CMAKE_CURRENT_BINARY_DIR}/METIS.info
  VAR METIS_Info
  DESCRIPTION "METIS supported libraries"
  FATAL_ON_MISSING_REQUIRED_PACKAGES
  WHAT ALL
)
message(STATUS ${METIS_Info})

#[==============================================================================================[
#                                        Main definition                                        #
]==============================================================================================]

# Main project
add_library(METIS_METIS)
set_target_properties(METIS_METIS PROPERTIES
  VERSION ${PROJECT_VERSION}
  SOVERSION ${PROJECT_VERSION_MAJOR}
  EXPORT_NAME METIS
  OUTPUT_NAME metis
)
add_library(METIS::METIS ALIAS METIS_METIS)

# Main definitions inside src
add_subdirectory(include)
add_subdirectory(src)

# Testing
if (METIS_TESTS)
  enable_testing()
  add_subdirectory(test)
endif ()

#[==============================================================================================[
#                                       Install or Export
#]==============================================================================================]

# Installation
if (METIS_INSTALL)
  # pkg-config files
  configure_file(cmake/metis.pc.in metis.pc @ONLY)
  install(FILES ${CMAKE_CURRENT_BINARY_DIR}/metis.pc
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig
    COMPONENT METIS_Development
  )

  # cmake export files
  write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/METISConfigVersion.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
  )
  configure_package_config_file(
    cmake/METISConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/METISConfig.cmake
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/METIS
  )
  install(FILES ${CMAKE_CURRENT_BINARY_DIR}/METISConfigVersion.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/METISConfig.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/METIS
    COMPONENT METIS_Development
  )
  export(EXPORT METISTargets
    FILE METISTargets.cmake
    NAMESPACE METIS::
  )
  install(EXPORT METISTargets
    FILE METISTargets.cmake
    NAMESPACE METIS::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/METIS
    COMPONENT METIS_Development
  )
endif ()
