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
  $<INSTALL_INTERFACE:${Fortran_MODULE_DIRECTORY}>)
if (FHASH_INSTALL)
    install(TARGETS fhash EXPORT fhash
            LIBRARY DESTINATION lib ${CMAKE_INSTALL_LIBDIR})
endif ()

if (FHASH_TEST_COVERAGE)
    target_compile_options(fhash PRIVATE --coverage -ftest-forall-temp)
endif ()

add_subdirectory(fhash_key)
