cmake_minimum_required (VERSION 3.15)
if (POLICY CMP0048)
  cmake_policy(SET CMP0048 NEW)
endif (POLICY CMP0048)
project (dirent VERSION 1.23.2 LANGUAGES C CXX)

# Initialize C and C++ compilers
enable_language (C CXX)

# Compile in debug mode by default
if (NOT CMAKE_BUILD_TYPE)
    set (CMAKE_BUILD_TYPE Debug
        CACHE STRING
        "Type of build: None Debug Release RelWithDebInfo MinSizeRel."
        FORCE
    )
endif (NOT CMAKE_BUILD_TYPE)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/dirent-config-version.cmake
  VERSION ${dirent_VERSION}
  COMPATIBILITY AnyNewerVersion
)

# Use the include directory only on Windows
if (WIN32)
    include_directories (${CMAKE_SOURCE_DIR}/include)
endif (WIN32)

# Install dirent.h
if (WIN32)
    install (FILES include/dirent.h DESTINATION include)
else (WIN32)
    cmake_policy(SET CMP0037 OLD) # Supress warnings about fake install
    add_custom_target(install) # Fake install target
endif (WIN32)

# Add distclean target
add_custom_target (distclean
    COMMAND ${CMAKE_BUILD_TOOL} clean
    COMMAND ${CMAKE_COMMAND} -P ${CMAKE_SOURCE_DIR}/distclean.cmake
)

install(
  FILES
    cmake/dirent-config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/dirent-config-version.cmake
  DESTINATION
    lib/cmake/dirent
)

# Build example programs
add_executable (find examples/find.c)
add_executable (ls examples/ls.c)
add_executable (locate examples/locate.c)
add_executable (updatedb examples/updatedb.c)
add_executable (scandir examples/scandir.c)

# Build test programs
include (CTest)
add_custom_target (check COMMAND ${CMAKE_CTEST_COMMAND} --output-on-failure -C ${CMAKE_CFG_INTDIR})
function (add_test_executable TEST_NAME)
    add_executable (${TEST_NAME} EXCLUDE_FROM_ALL ${ARGN})
    add_test (NAME ${TEST_NAME} WORKING_DIRECTORY ${CMAKE_SOURCE_DIR} COMMAND $<TARGET_FILE:${TEST_NAME}>)
    add_dependencies (check ${TEST_NAME})
endfunction (add_test_executable)

add_test_executable (t-compile tests/t-compile.c)
add_test_executable (t-dirent tests/t-dirent.c)
add_test_executable (t-scandir tests/t-scandir.c)
add_test_executable (t-cplusplus tests/t-cplusplus.cpp)

