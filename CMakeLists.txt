# NOTE for OSX users:
# export MACOSX_DEPLOYMENT_TARGET=10.7 on .bash_profile if you want to compile use 10.7 sdk
# sdk location: /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs
# you can download old sdk from here: https://github.com/JuliaEichler/Mac_OSX_SDKs
# default is 10.7 for binary release.

# CMake arguments for WizNote
# WIZNOTE_USE_QT5   use qt5 to compile project
# APPSTORE_BUILD  build for Apple Appsotre, would not use qt deploy
# XCODEBUILD      adjust project params to suit for xcode
# PLCrashReporter   use PLCrashReporter for mac, need PLCrashReporter framework

if (APPLE)
    cmake_minimum_required(VERSION 2.8.12)
else(APPLE)
    cmake_minimum_required(VERSION 2.8.4)
endif(APPLE)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

# rpath resolve.
#set(CMAKE_BUILD_WITH_INSTALL_RPATH YES)     // would cause wheelEvent problem on centos/fedora when compiled with qt5
if(APPLE)
    set(CMAKE_INSTALL_NAME_DIR "@rpath/PlugIns")
    SET(CMAKE_INSTALL_RPATH "@loader_path/..;@executable_path/..")
else(APPLE)
    set(CMAKE_INSTALL_RPATH "$ORIGIN;$ORIGIN/..;$ORIGIN/../lib/wiznote/plugins")
    set (openssl-linked on)
endif(APPLE)

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING
      "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel."
      FORCE)
endif()

include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX11)
    if(APPLE)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -stdlib=libc++")
    else(APPLE)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
    endif(APPLE)
elseif(COMPILER_SUPPORTS_CXX0X)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
        message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

if(NOT WIZNOTE_USE_QT5)
    set(WIZNOTE_USE_QT5 OFF CACHE BOOL
        "Build WizNote use Qt5, default is off.")
endif(NOT WIZNOTE_USE_QT5)

if(APPLE)
    if(APPSTORE_BUILD)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -gdwarf-2")
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -gdwarf-2")
        add_definitions(-DBUILD4APPSTORE=TRUE)
    endif(APPSTORE_BUILD)
    if (PLCrashReporter)
        add_definitions(-DUsePLCrashReporter=TRUE)
    endif(PLCrashReporter)
    add_definitions(-DUSECOCOATOOLBAR=TRUE)
#    add_definitions(-DUSEWEBENGINE=FALSE)
endif(APPLE)

if(APPLE)
    set(WIZNOTE_ROOT_DIR ${CMAKE_BINARY_DIR}/WizNote.app)
    set(WIZNOTE_CONTENTS_DIR ${WIZNOTE_ROOT_DIR}/Contents)
    set(WIZNOTE_PLUGIN_DIR ${WIZNOTE_ROOT_DIR}/Contents/PlugIns)
    set(WIZNOTE_RESOURCE_DIR ${WIZNOTE_ROOT_DIR}/Contents/Resources)
    set(WIZNOTE_PLUGINSPEC_DIR ${WIZNOTE_ROOT_DIR}/Contents/Resources/plugIns)
else(APPLE)
    set(WIZNOTE_ROOT_DIR ${CMAKE_BINARY_DIR})
    set(WIZNOTE_PLUGIN_DIR ${WIZNOTE_ROOT_DIR}/lib/wiznote/plugins)
    set(WIZNOTE_PLUGINSPEC_DIR ${WIZNOTE_PLUGIN_DIR})
    set(WIZNOTE_RESOURCE_DIR ${WIZNOTE_ROOT_DIR}/share)
endif(APPLE)

# must output to "lib" to make macdeployqt work correctly
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${WIZNOTE_PLUGIN_DIR})


add_subdirectory(lib)
add_subdirectory(src)
