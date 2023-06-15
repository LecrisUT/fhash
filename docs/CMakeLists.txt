find_program(FORD ford)
find_package(PythonInterp)

if (NOT PYTHON_VERSION_STRING VERSION_LESS 3.5)
    set(MARKDOWN_SUPPORTED_PYTHON true)
endif ()

if (NOT MARKDOWN_SUPPORTED_PYTHON)
    message(WARNING "Python version ${PYTHON_VERSION_STRING} is not supported by Python-Markdown, not attempting to build documentation.")
endif ()

if (FORD AND MARKDOWN_SUPPORTED_PYTHON)

    set(DOC_ROOT "${PROJECT_BINARY_DIR}/doc")
    set(DOC_DIR "${DOC_ROOT}/fhash")
    set(PROJ_DIR "${PROJECT_SOURCE_DIR}")
    set(FORD_PROJECT_FILE "${CMAKE_CURRENT_SOURCE_DIR}/ford.md")
    set(FORD_PROJECT_EXCLUDE_DIRS "${MPI_Fortran_INCLUDE_DIRS};${PROJECT_BINARY_DIR};${PROJ_DIR}/build;${PROJ_DIR}/tests;${PROJ_DIR}/sphinx")
    set(FORD_PROJECT_EXCLUDE_FILES "random.f90;random_integer.f90")
    set(MACRO_FLAG "")
    string(REPLACE ";" "\ninclude:" FORD_PROJECT_INCLUDES "${PROJ_DIR}/src")
    string(REPLACE ";" "\nexclude_dir:" FORD_PROJECT_EXCLUDE_DIRS "${FORD_PROJECT_EXCLUDE_DIRS}")
    string(REPLACE ";" "\nexclude:" FORD_PROJECT_EXCLUDE_FILES "${FORD_PROJECT_EXCLUDE_FILES}")

    # Pick the preprocessor to use based on the Fortran compiler
    if ("${CMAKE_Fortran_COMPILER_ID}" MATCHES "Intel")
        set(FPP "fpp\n")
    else ()
        set(FPP "gfortran -E\n") # default to gfortran -E for gfortran and unsupported compilers
    endif ()
    file(WRITE "${PROJECT_BINARY_DIR}/.PREPROCESSOR" "${FPP}")

    # Dynamically generate the FORD outputs list
    message(STATUS "Dynamically computing FORD output information...")
    if (NOT (DEFINED FORD_OUTPUTS_CACHED))
        message(STATUS "Running FORD to dynamically compute documentation outputs, this could take a while...")
        execute_process(COMMAND ${CMAKE_COMMAND} -E remove_directory ${DOC_DIR}
                COMMAND ${CMAKE_COMMAND} -E make_directory ${DOC_DIR}
                COMMAND "${FORD}" --debug -q ${MACRO_FLAG} -d "${PROJ_DIR}" -o "${DOC_DIR}" -p "${PAGE_DIR}" "${FORD_PROJECT_FILE}" OUTPUT_QUIET)
    else ()
        message(STATUS "Re-using cached FORD outputs, rather than regenerating them")
    endif ()

    # Compile a list of FORD output files
    file(GLOB_RECURSE FORD_OUTPUTS
            "${DOC_DIR}/*.*")
    file(GLOB_RECURSE FORD_CLEAN_OUTPUTS
            "${DOC_DIR}/*.*")

    # Cache FORD outputs
    if ((DEFINED FORD_OUTPUTS) AND (NOT ("${FORD_OUTPUTS}" STREQUAL "")))
        message(STATUS "Caching FORD outputs")
        set(FORD_OUTPUTS_CACHED "${FORD_OUTPUTS}"
                CACHE STRING "variable containing FORD outputs to prevent rebuilding FORD docs" FORCE)
    endif ()
    message(STATUS "Done dynamically computing FORD outputs.")

    foreach (DOC_SRC_FILE ${PROJECT_SOURCE_DIR}/ford.md ${FHASH_SOURCES})
        list(APPEND FORD_DEPENDS "${DOC_SRC_FILE}")
    endforeach ()

    # Re-build FORD output if needed
    add_custom_command(OUTPUT ${FORD_OUTPUTS_CACHED}
            COMMAND "${FORD}" --debug ${MACRO_FLAG} -d "${PROJ_DIR}" -o "${DOC_DIR}" "${FORD_PROJECT_FILE}"
            MAIN_DEPENDENCY "${FORD_PROJECT_FILE}"
            DEPENDS ${FORD_DEPENDS}
            COMMENT "Building HTML documentation for ${PROJECT_NAME} using FORD")

    add_custom_target(documentation ALL
            DEPENDS ${FORD_OUTPUTS_CACHED})

    # Install documentation
    set(INSTALL_API_DOCUMENTATION TRUE
            CACHE BOOL "Install FORD generated documentation?")
    if (INSTALL_API_DOCUMENTATION)
        install(DIRECTORY "${DOC_ROOT}/" DESTINATION share/doc)
    else ()

        # FORD Not found
        message(WARNING
                "FORD not found. Please set the CMake cache variable FORD to point to the installed FORD executable, and reconfigure or disable building the documentation. FORD can be installed from PYPI with `sudo pip install FORD` or from <https://github.com/cmacmackin/ford> If you do not wish to install FORD and build the JSON-Fortran documentation, then please set the CMake cache variable SKIP_DOC_GEN to TRUE.")

    endif ()
endif ()
