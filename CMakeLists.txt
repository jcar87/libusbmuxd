cmake_minimum_required(VERSION 3.16)

project(libusbmuxd LANGUAGES C)

set(CMAKE_PREFIX_PATH "/home/julius/.conan/data/libplist/2.2.0/luis/test/package/abd3ca9581f5ec3d6672fa2ee8818b1f09dbb082/")
find_package(libplist CONFIG REQUIRED)
find_package(Threads REQUIRED)

#######################################################
###  libinternalcommon - internal only
#######################################################

add_library(internalcommon STATIC)

target_sources(internalcommon PRIVATE
    common/collection.c
    common/collection.h
    common/socket.c
    common/socket.h
    common/thread.h
    common/thread.c
)

target_include_directories(internalcommon
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/common>
)

set_target_properties(internalcommon PROPERTIES 
    POSITION_INDEPENDENT_CODE ON
)


#######################################################
###  libusbmuxd
#######################################################

add_library(usbmuxd SHARED)

target_sources(usbmuxd PRIVATE
src/libusbmuxd.c)

target_include_directories(usbmuxd
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include/>
)

target_compile_definitions(usbmuxd PRIVATE HAVE_STPNCPY PACKAGE_STRING="libusbmuxd 2.0.2")

target_link_libraries(usbmuxd PRIVATE plist-2.0::plist internalcommon)

set_target_properties(usbmuxd PROPERTIES 
    VERSION 6.0.0
    SOVERSION 6)

#######################################################
###  iproxy tool
#######################################################

add_executable(iproxy
    tools/iproxy.c)

target_link_libraries(iproxy PRIVATE usbmuxd internalcommon Threads::Threads)

target_compile_definitions(iproxy PRIVATE PACKAGE_VERSION="" PACKAGE_URL="" PACKAGE_BUGREPORT="")


#######################################################
###  INSTALL usbmuxd
#######################################################

install(TARGETS usbmuxd
        EXPORT libusbmuxd-targets
        LIBRARY DESTINATION lib)
    
install(EXPORT libusbmuxd-targets
        NAMESPACE libusbmuxd::
        FILE libusbmuxd-targets.cmake
        DESTINATION lib/cmake/libusbmuxd)

install(
    DIRECTORY ${CMAKE_SOURCE_DIR}/include
    DESTINATION ./
    FILES_MATCHING PATTERN "*.h"
)

include(CMakePackageConfigHelpers)

# generate the config file that is includes the exports
configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/libusbmuxd-config.cmake"
  INSTALL_DESTINATION "lib/cmake/libusbmuxd"
  NO_SET_AND_CHECK_MACRO
  NO_CHECK_REQUIRED_COMPONENTS_MACRO
  )

install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/libusbmuxd-config.cmake
    DESTINATION lib/cmake/libusbmuxd
)