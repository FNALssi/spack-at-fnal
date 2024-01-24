cmake_minimum_required(VERSION 3.18.2 FATAL_ERROR)

find_package(cetmodules 3.23.00 REQUIRED)
project(spack-at-fnal VERSION 0.00.01 LANGUAGES)

include(CetCMakeEnv)
cet_cmake_env()

add_subdirectory(doc)

cet_cmake_config()
