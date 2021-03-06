set(PROFILE_LIT_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR})
set(PROFILE_LIT_BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR})

set(PROFILE_TESTSUITES)
set(PROFILE_TEST_DEPS ${SANITIZER_COMMON_LIT_TEST_DEPS})
if(NOT COMPILER_RT_STANDALONE_BUILD)
  list(APPEND PROFILE_TEST_DEPS profile llvm-profdata llvm-cov)
endif()

set(PROFILE_TEST_ARCH ${PROFILE_SUPPORTED_ARCH})
if(APPLE)
  darwin_filter_host_archs(PROFILE_SUPPORTED_ARCH PROFILE_TEST_ARCH)
endif()

foreach(arch ${PROFILE_TEST_ARCH})
  set(PROFILE_TEST_TARGET_ARCH ${arch})
  if(${arch} MATCHES "arm|aarch64")
    # This is only true if we're cross-compiling.
    set(PROFILE_TEST_TARGET_CFLAGS ${COMPILER_RT_TEST_COMPILER_CFLAGS})
  else()
    get_target_flags_for_arch(${arch} PROFILE_TEST_TARGET_CFLAGS)
    string(REPLACE ";" " " PROFILE_TEST_TARGET_CFLAGS "${PROFILE_TEST_TARGET_CFLAGS}")
  endif()
  set(CONFIG_NAME Profile-${arch})
  configure_lit_site_cfg(
    ${CMAKE_CURRENT_SOURCE_DIR}/lit.site.cfg.in
    ${CMAKE_CURRENT_BINARY_DIR}/${CONFIG_NAME}/lit.site.cfg
   )
  list(APPEND PROFILE_TESTSUITES ${CMAKE_CURRENT_BINARY_DIR}/${CONFIG_NAME})
endforeach()

add_lit_testsuite(check-profile "Running the profile tests"
  ${PROFILE_TESTSUITES}
  DEPENDS ${PROFILE_TEST_DEPS})
set_target_properties(check-profile PROPERTIES FOLDER "Profile tests")
