add_library(fhash_TestLite)
add_library(fhash_tests)
add_executable(fhash_main_tests main.f90)

target_sources(fhash_TestLite PRIVATE
        TestLite.f90
        TestLite_suite.f90
        TestLite_error.f90
        )

target_sources(fhash_tests PRIVATE
        test_container.f90
        test_fnv.f90
        test_key.f90
        test_sll.f90
        test_tbl.f90
        test_tbl_iter.f90
        )


target_link_libraries(fhash_tests PRIVATE fhash_fhash fhash_TestLite)
target_link_libraries(fhash_main_tests PRIVATE fhash_fhash fhash_TestLite fhash_tests)

add_test(main fhash_main_tests)

add_custom_target(all_tests ALL
  DEPENDS fhash_main_tests)

add_custom_command(
     TARGET all_tests
     COMMENT "Run tests"
     POST_BUILD
     WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
     COMMAND ${CMAKE_CTEST_COMMAND} --output-on-failure -C $<CONFIGURATION>
)
