add_executable(tests main.cc)
target_fix_ld(tests)

# link additional test libraries here
target_link_libraries(tests
    PRIVATE
    project_warnings
    project_options

    doctest
)

# do not try to discover tests if dynamic linker is set as it probably wont be
# able to run it and will cause an error
if(NOT (UNIX AND CMAKE_FORCE_LD))
    include(${doctest_SOURCE_DIR}/scripts/cmake/doctest.cmake)
    doctest_discover_tests(tests)
endif()
