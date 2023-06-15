cmake_minimum_required(VERSION 3.14)

project(fhash
        VERSION 0.2.1
        LANGUAGES Fortran)

option(FHASH_TESTS "fhash: Build unit tests" ${PROJECT_IS_TOP_LEVEL})
option(FHASH_INSTALL "fhash: Install project" ${PROJECT_IS_TOP_LEVEL})
option(FHASH_SHARED_LIBS "fhash: Build as a shared library" ${PROJECT_IS_TOP_LEVEL})
option(FHASH_DOCS "fhash: Build documentations" OFF)
option(FHASH_TEST_COVERAGE "fhash: Test with coverage" OFF)
mark_as_advanced(FHASH_TEST_COVERAGE)

if (NOT DEFINED CMAKE_Fortran_MODULE_DIRECTORY)
    set(CMAKE_Fortran_MODULE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/fortran_modules)
endif ()
set(BUILD_SHARED_LIBS ${FHASH_SHARED_LIBS})

# Define main targets
add_library(fhash_fhash)
add_library(fhash::fhash ALIAS fhash_fhash)

# Actual implementation
add_subdirectory(src)

if (FHASH_INSTALL)
    include(CMakePackageConfigHelpers)

    write_basic_package_version_file(fhashConfigVersion.cmake
            VERSION ${PROJECT_VERSION}
            COMPATIBILITY SameMinorVersion)
    configure_package_config_file(
            cmake/fhashConfig.cmake.in
            fhashConfig.cmake
            INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/fhash)
    install(FILES ${CMAKE_CURRENT_BINARY_DIR}/fhashConfigVersion.cmake
            ${CMAKE_CURRENT_BINARY_DIR}/fhashConfig.cmake
            DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/fhash
            COMPONENT Spglib_Development)

    install(EXPORT fhashTargets
            FILE fhashTargets.cmake
            DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/fhash
            COMPONENT fhash_Development)
    export(EXPORT fhashTargets
            FILE fhashTargets.cmake
            NAMESPACE fhash::)
endif ()

# -------------------
# Build documentation
# -------------------

if (FHASH_DOCS)
    add_subdirectory(docs)
endif ()

if (FHASH_TESTS)
    enable_testing()
    add_subdirectory(test)
endif ()
