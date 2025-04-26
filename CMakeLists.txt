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

set(SDL_STATIC ON CACHE INTERNAL "") # build sdl statically by default
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/vendor/sdl EXCLUDE_FROM_ALL)
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/vendor/sdl_image EXCLUDE_FROM_ALL)
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/vendor/sdl_ttf EXCLUDE_FROM_ALL)

set(BUILD_TESTS OFF CACHE INTERNAL "")
set(BUILD_EXAMPLES OFF CACHE INTERNAL "")
set(BUILD_CLI OFF CACHE INTERNAL "")
set(BUILD_CLAR OFF CACHE INTERNAL "")
set(BUILD_SHARED_LIBS OFF CACHE INTERNAL "") # build libgit as static library
set(LINK_WITH_STATIC_LIBRARIES ON CACHE INTERNAL "") # link with static libraries
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/vendor/libgit2 EXCLUDE_FROM_ALL)

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

target_include_directories(${EXECUTABLE_NAME} PUBLIC ${clay_SOURCE_DIR})

target_compile_definitions(${EXECUTABLE_NAME} PRIVATE SDL_MAIN_USE_CALLBACKS)

target_link_libraries(${EXECUTABLE_NAME}
    PRIVATE
    SDL3::SDL3-static
    SDL3_image::SDL3_image-static
    SDL3_ttf::SDL3_ttf-static
    libgit2 llhttp ntlmclient xdiff util libgit2package
)

