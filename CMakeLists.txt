cmake_minimum_required(VERSION 3.12)

set(PATCH_VERSION "1" CACHE INTERNAL "Patch version")
set(PROJECT_VESRION 0.0.${PATCH_VERSION})

project(ip_sort VERSION ${PROJECT_VESRION})

option(WITH_GOOGLETEST "Whether to use googletest framework test" ON)

configure_file(version.h.in version.h)

add_executable(ip_sort_cli main.cpp)
add_library(ip_sort lib.cpp)

set_target_properties(ip_sort_cli ip_sort PROPERTIES
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED ON
)

target_include_directories(ip_sort
    PRIVATE "${CMAKE_BINARY_DIR}"
)

target_link_libraries(ip_sort_cli PRIVATE
    ip_sort
)

if(WITH_GOOGLETEST)
    if (MSVC)
        set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
    endif()

    add_subdirectory("googletest")

    add_executable(test_version test_version.cpp)

    set_target_properties(test_version PROPERTIES
        CXX_STANDARD 14
        CXX_STANDARD_REQUIRED ON
    )

    target_include_directories(
        test_version PRIVATE
        "${gtest_SOURCE_DIR}/include"
    )

    target_link_libraries(test_version PRIVATE
        gtest_main
        ip_sort
    )
endif()

if (MSVC)
    target_compile_options(ip_sort_cli PRIVATE
        /W4
    )
    target_compile_options(ip_sort PRIVATE
        /W4
    )
    if(WITH_GOOGLETEST)
        target_compile_options(test_version PRIVATE
            /EHa /W4 /WX
        )
    endif()
else ()
    target_compile_options(ip_sort_cli PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
    target_compile_options(ip_sort PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
    if(WITH_GOOGLETEST)
        target_compile_options(test_version PRIVATE
            -Wall -Wextra -pedantic -Werror
        )
    endif()
endif()

install(TARGETS ip_sort_cli RUNTIME DESTINATION bin)

set(CPACK_GENERATOR DEB)
set(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")
set(CPACK_PACKAGE_CONTACT example@example.com)
include(CPack)

if(WITH_GOOGLETEST)
    enable_testing()
    add_test(test_version test_version)
endif()
