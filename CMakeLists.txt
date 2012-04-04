# Copyright 2012 Intel Corporation
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

project(waffle C)
set(waffle_version 0.0)
cmake_minimum_required(VERSION 2.8)

# ------------------------------------------------------------------------------
# Find Libraries
# ------------------------------------------------------------------------------

function(waffle_find_library var name)
    find_library("${var}" "${name}")

    if("${var}")
        message(STATUS "Library ${name} found: ${${var}}")
    else("${var}")
        message(STATUS "Library ${name} not found")
    endif("${var}")
endfunction(waffle_find_library)

if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    waffle_find_library(waffle_EGL_library EGL)
    waffle_find_library(waffle_X11-xcb_library X11-xcb)
endif(CMAKE_SYSTEM_NAME STREQUAL "Linux")

# ------------------------------------------------------------------------------
# Waffle Options
# ------------------------------------------------------------------------------

# All options here are prefixed with `waffle_`. Most options are cache
# variables set by the user.

# ----------------------------------------------
# Set waffle options
# ----------------------------------------------

option(waffle_build_tests "Build tests" ON)
option(waffle_build_examples "Build examples" ON)

#
# Set waffle_default_platform.
#

if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    set(waffle_default_platform_choices "x11_egl (just one choice for now)")
    set(waffle_default_platform_regex "x11_egl")
    set(waffle_default_platform "NOTFOUND"
        CACHE STRING "One of ${waffle_default_platform_choices}")
endif(CMAKE_SYSTEM_NAME STREQUAL "Linux")

#
# Set waffle_default_gl_api.
#

set(waffle_default_gl_api_choices "gl, gles1, gles2")
set(waffle_default_gl_api_regex "gl|gles1|gles2")
set(waffle_default_gl_api "NOTFOUND"
    CACHE STRING "One of ${waffle_default_gl_api_choices}")

#
# Set waffle_has_{platform}.
#

if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    # Set waffle_has_x11_egl.
    set(waffle_has_x11_egl_default OFF)
    if(waffle_EGL_library AND waffle_X11-xcb_library)
        set(waffle_has_x11_egl_default ON)
    endif(waffle_EGL_library AND waffle_X11-xcb_library)
    option(waffle_has_x11_egl
           "Build support for X11/EGL"
           ${waffle_has_x11_egl_default})
endif(CMAKE_SYSTEM_NAME STREQUAL "Linux")

#
# Set internal option waffle_has_egl.
#

if(waffle_has_x11_egl)
    set(waffle_has_egl TRUE)
else(waffle_has_x11_egl)
    set(waffle_has_egl FALSE)
endif(waffle_has_x11_egl)

#
# Set internal option waffle_has_x11.
#

if(waffle_has_x11_egl)
    set(waffle_has_x11 TRUE)
else(waffle_has_x11_egl)
    set(waffle_has_x11 FALSE)
endif(waffle_has_x11_egl)

# ----------------------------------------------
# Validate waffle options
# ----------------------------------------------

#
# Validate waffle_default_platform.
#

if(NOT DEFINED waffle_default_platform)
    message(FATAL_ERROR "Option waffle_default_platform is not defined. Must be one of ${waffle_default_platform_choices}.")
endif(NOT DEFINED waffle_default_platform)

if(NOT waffle_default_platform MATCHES ${waffle_default_platform_regex})
    message(FATAL_ERROR "Option waffle_default_platform has bad value '${waffle_default_platform}'. Must be one of ${waffle_default_platform_choices}.")
endif(NOT waffle_default_platform MATCHES ${waffle_default_platform_regex})

#
# Validate waffle_default_gl_api.
#

if(NOT DEFINED waffle_default_gl_api)
    message(FATAL_ERROR "Option waffle_default_gl_api is not defined. Must be one of ${waffle_default_gl_api_choices}.")
endif(NOT DEFINED waffle_default_gl_api)

if(NOT waffle_default_gl_api MATCHES ${waffle_default_gl_api_regex})
    message(FATAL_ERROR "Option waffle_default_gl_api has bad value '${waffle_default_gl_api}'. Must be one of ${waffle_default_gl_api_choices}.")
endif(NOT waffle_default_gl_api MATCHES ${waffle_default_gl_api_regex})

# ------------------------------------------------------------------------------
# Compiler Flags
# ------------------------------------------------------------------------------

# FIXME: Only enable c99 if compiler supports it.
set(CMAKE_C_FLAGS "--std=c99 -Wall -Werror")

set(CMAKE_C_FLAGS_DEBUG "-g3 -O0 -DDEBUG")

# Produce enough debug info for generating backtraces, but not
# single-stepping.
set(CMAKE_C_FLAGS_RELEASE "-g1 -02 -DNDEBUG")

#
# Define WAFFLE_PLATFORM_DEFAULT.
#

if(waffle_default_platform STREQUAL "glx")
    add_definitions(-DWAFFLE_PLATFORM_DEFAULT=WAFFLE_PLATFORM_GLX)
elseif(waffle_default_platform STREQUAL "wayland")
    add_definitions(-DWAFFLE_PLATFORM_DEFAULT=WAFFLE_PLATFORM_WAYLAND)
elseif(waffle_default_platform STREQUAL "x11_egl")
    add_definitions(-DWAFFLE_PLATFORM_DEFAULT=WAFFLE_PLATFORM_X11_EGL)
endif(waffle_default_platform STREQUAL "glx")

#
# Define macro WAFFLE_GL_API_DEFAULT.
#

if(waffle_default_gl_api STREQUAL "gl")
    add_definitions(-DWAFFLE_GL_API_DEFAULT=WAFFLE_GL)
elseif(waffle_default_gl_api STREQUAL "gles1")
    add_definitions(-DWAFFLE_GL_API_DEFAULT=WAFFLE_GLES1)
elseif(waffle_default_gl_api STREQUAL "gles2")
    add_definitions(-DWAFFLE_GL_API_DEFAULT=WAFFLE_GLES2)
endif(waffle_default_gl_api STREQUAL "gl")

#
# Define macros WAFFLE_HAS_{PLATFORM}.
#
# Android is not in this list because it uses a separate build system.
#

if(waffle_has_x11_egl)
    add_definitions(-DWAFFLE_HAS_X11_EGL)
endif(waffle_has_x11_egl)

# ------------------------------------------------------------------------------
# Target: check
# ------------------------------------------------------------------------------

if(waffle_build_tests)
    # Subdirectories should add dependencies.
    add_custom_target(check)
endif(waffle_build_tests)

# ------------------------------------------------------------------------------
# Add subdirectories
# ------------------------------------------------------------------------------

include_directories(include src)

add_subdirectory(src)

if(waffle_build_tests)
    add_subdirectory(tests)
endif(waffle_build_tests)

if(waffle_build_examples)
    add_subdirectory(examples)
endif(waffle_build_examples)

# ------------------------------------------------------------------------------
# Print summary
# ------------------------------------------------------------------------------

# CMake is annoyingly silent compared to autoconf. The user wants to know what
# was configured how.

message("-----------------------------------------------")
message("")
message("Waffle configuration summary")
message("")
message("Supported platforms: ")
if(waffle_has_x11_egl)
    message("    x11_egl")
endif(waffle_has_x11_egl)
message("")
message("Libraries:")
if(DEFINED waffle_EGL_library)
    message("    EGL: ${waffle_EGL_library}")
endif(DEFINED waffle_EGL_library)
if(DEFINED waffle_X11-xcb_library)
    message("    X11-xcb: ${waffle_X11-xcb_library}")
endif(DEFINED waffle_X11-xcb_library)
message("")
message("Defaults:")
message("    default platform: ${waffle_default_platform}")
message("    default GL API: ${waffle_default_gl_api}")
message("")
message("Build type:")
message("    ${CMAKE_BUILD_TYPE}")
message("")
message("Tools:")
message("    cc: ${CMAKE_C_COMPILER}")
message("    CFLAGS_base: ${CMAKE_C_FLAGS}")
message("    CFLAGS_debug: ${CMAKE_C_FLAGS_DEBUG}")
message("    CFLAGS_release: ${CMAKE_C_FLAGS_RELEASE}")
message("-----------------------------------------------")