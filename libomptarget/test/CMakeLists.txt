# CMakeLists.txt file for unit testing OpenMP offloading runtime library.

if(LIBOMPTARGET_CMAKE_BUILD_TYPE MATCHES debug)
  set(LIBOMPTARGET_DEBUG True)
else()
  set(LIBOMPTARGET_DEBUG False)
endif()

if(${OPENMP_STANDALONE_BUILD})
  if (NOT(${OPENMP_TEST_C_COMPILER} MATCHES "clang" AND ${OPENMP_TEST_CXX_COMPILER} MATCHES "clang"))
    libomptarget_say("Can only test with Clang compiler!")
    libomptarget_warning_say("The check-libomptarget target will not be available!")
    return()
  endif()

  execute_process(
    COMMAND ${OPENMP_TEST_C_COMPILER} --version
    OUTPUT_VARIABLE TEST_COMPILER_VERSION)
  string(REGEX MATCH "version ([0-9.]+)" TEST_COMPILER_VERSION ${TEST_COMPILER_VERSION})
  if (NOT(TEST_COMPILER_VERSION))
    libomptarget_say("Unable to determine Clang compiler version!")
    libomptarget_warning_say("The check-libomptarget target will not be available!")
    return()
  endif()
  set(TEST_COMPILER_VERSION ${CMAKE_MATCH_1})
  if (TEST_COMPILER_VERSION VERSION_LESS 6.0.0)
    libomptarget_say("Clang compiler version does not implement all codegen, please update to 6.0.0!")
    libomptarget_warning_say("The check-libomptarget target will not be available!")
    return()
  endif()
endif()

add_openmp_testsuite(check-libomptarget "Running libomptarget tests" ${CMAKE_CURRENT_BINARY_DIR} DEPENDS omptarget omp)

if(${OPENMP_STANDALONE_BUILD})
  set(LIBOMPTARGET_OPENMP_HEADER_FOLDER "${CMAKE_CURRENT_BINARY_DIR}/../../runtime/src" CACHE STRING
    "Path to folder containing omp.h")
  set(LIBOMPTARGET_OPENMP_HOST_RTL_FOLDER "${CMAKE_CURRENT_BINARY_DIR}/../../runtime/src" CACHE STRING
    "Path to folder containing libomp.so")
else()
  set(LIBOMPTARGET_OPENMP_HEADER_FOLDER "${LIBOMPTARGET_BINARY_DIR}/../runtime/src")
endif()

# Configure the lit.site.cfg.in file
set(AUTO_GEN_COMMENT "## Autogenerated by libomptarget configuration.\n# Do not edit!")
configure_file(lit.site.cfg.in lit.site.cfg @ONLY)
