cmake_minimum_required(VERSION 3.24..4.1.2 FATAL_ERROR)

# ##############################################################################
# Ensure we have access to a suitable version of Cetmodules
include(FetchContent)
FetchContent_Declare(
  cetmodules
  GIT_REPOSITORY https://github.com/FNALssi/cetmodules
  GIT_TAG 4.01.01
  GIT_SHALLOW ON
  FIND_PACKAGE_ARGS 4.01.01
  )
FetchContent_MakeAvailable(cetmodules)
find_package(cetmodules 4.01.01 REQUIRED)
# ##############################################################################

project(spack-at-fnal VERSION 0.00.01 LANGUAGES NONE)

include(CetCMakeEnv)
cet_cmake_env()

add_subdirectory(doc)

cet_cmake_config()
