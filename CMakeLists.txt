cmake_minimum_required(VERSION 3.27)

# set the output directory for built objects.
# This makes sure that the dynamic library goes into the build directory automatically.
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/$<CONFIGURATION>")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/$<CONFIGURATION>")

# prevent installing to system directories.
set(CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}" CACHE INTERNAL "")

set(CMAKE_BUILD_WITH_INSTALL_RPATH ON CACHE INTERNAL "")
set(CMAKE_INSTALL_RPATH "\$ORIGIN" CACHE INTERNAL "")

project(chaotic-mind VERSION 0.1 LANGUAGES CXX C)

if(MSVC)
    if(NOT CMAKE_GENERATOR STREQUAL "Ninja")
        # parallelize each target, unless Ninja is the generator
        add_definitions(/MP)
    endif()
endif()

include(FetchContent)
set(FETCHCONTENT_QUIET FALSE)
FetchContent_Declare(
    SDL
    SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/vendor/SDL3"
    GIT_REPOSITORY "https://github.com/libsdl-org/SDL.git"
    GIT_TAG 877399b2b2cf21e67554ed9046410f268ce1d1b2 # release-3.2.10
    GIT_SHALLOW TRUE
    GIT_PROGRESS TRUE
)
FetchContent_MakeAvailable(SDL)
set_property(DIRECTORY "${sdl_SOURCE_DIR}" PROPERTY EXCLUDE_FROM_ALL TRUE)

FetchContent_Declare(
    SDL_ttf
    SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/vendor/SDL3_ttf"
    GIT_REPOSITORY "https://github.com/libsdl-org/SDL_ttf.git"
    GIT_TAG a1ce3670aec736ecbf0936c43f2f0cc53aa61e5b # release-3.2.2
    GIT_SHALLOW TRUE
    GIT_PROGRESS TRUE
)
FetchContent_MakeAvailable(SDL_ttf)
set_property(DIRECTORY "${sdl_ttf_SOURCE_DIR}" PROPERTY EXCLUDE_FROM_ALL TRUE)

FetchContent_Declare(
    SDL_image
    SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/vendor/SDL3_image"
    GIT_REPOSITORY "https://github.com/libsdl-org/SDL_image.git"
    GIT_TAG 11154afb7855293159588b245b446a4ef09e574f # release-3.2.4
    GIT_SHALLOW TRUE
    GIT_PROGRESS TRUE
)
FetchContent_MakeAvailable(SDL_image)
set_property(DIRECTORY "${SDL_image_SOURCE_DIR}" PROPERTY EXCLUDE_FROM_ALL TRUE)

FetchContent_Declare(
    libgit2
    SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/vendor/libgit2"
    GIT_REPOSITORY "https://github.com/libgit2/libgit2.git"
    GIT_TAG v1.9.0
    GIT_SHALLOW TRUE
    GIT_PROGRESS TRUE
)
set(BUILD_TESTS OFF CACHE INTERNAL "")
set(BUILD_CLI OFF CACHE INTERNAL "")
FetchContent_MakeAvailable(libgit2)
set_property(DIRECTORY "${libgit2_SOURCE_DIR}" PROPERTY EXCLUDE_FROM_ALL TRUE)

FetchContent_Declare(
    clay
    SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/vendor/clay"
    GIT_REPOSITORY "https://github.com/nicbarker/clay.git"
    GIT_TAG 6d23a35d1596f139998b2994a63a1d5a982e713b # v0.13
    GIT_SHALLOW TRUE
    GIT_PROGRESS TRUE
    SOURCE_SUBDIR null # point to non-existing subdir so cmake is not configured
)
FetchContent_MakeAvailable(clay)
set_property(DIRECTORY "${clay_SOURCE_DIR}" PROPERTY EXCLUDE_FROM_ALL TRUE)

set(EXECUTABLE_NAME ${PROJECT_NAME})

if(ANDROID)
    set(EXECUTABLE_NAME main)
    add_library(${EXECUTABLE_NAME} SHARED)
else()
    add_executable(${EXECUTABLE_NAME})
endif()

target_sources(${EXECUTABLE_NAME}
    PRIVATE
    src/main.cpp
)

target_compile_features(${EXECUTABLE_NAME} PUBLIC cxx_std_20)

target_include_directories(${EXECUTABLE_NAME} PUBLIC ${SDL3_INCLUDE_DIRS} ${SDL3_image_INCLUDE_DIRS} ${SDL3_ttf_INCLUDE_DIRS})

target_compile_definitions(${EXECUTABLE_NAME} PRIVATE SDL_MAIN_USE_CALLBACKS)

target_link_libraries(${EXECUTABLE_NAME}
    PRIVATE
    SDL3::SDL3
    SDL3_image::SDL3_image
    SDL3_ttf::SDL3_ttf
)

