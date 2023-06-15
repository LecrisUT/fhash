target_sources(fhash PRIVATE
        fhash.f90
        fhash_sll.f90
        fhash_data_container.f90
        fhash_tbl.f90
        fhash_tbl_iter.f90
        fhash_fnv.f90
        )


target_include_directories(fhash PUBLIC
        $<BUILD_INTERFACE:${CMAKE_Fortran_MODULE_DIRECTORY}>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)
if (FHASH_INSTALL)
    install(TARGETS fhash
            EXPORT fhashTargets
            LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT fhash_Runtime NAMELINK_COMPONENT fhash_Development
            ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT fhash_Development
            )
    install(FILES ${CMAKE_Fortran_MODULE_DIRECTORY}/fhash.mod
            DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
            COMPONENT fhash_Development)
endif ()

if (FHASH_TEST_COVERAGE)
    target_compile_options(fhash PRIVATE --coverage -ftest-forall-temp)
endif ()

add_subdirectory(fhash_key)
