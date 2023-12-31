# Copyright by The HDF Group.
# All rights reserved.
#
# This file is part of HDF5.  The full HDF5 copyright notice, including
# terms governing use, modification, and redistribution, is contained in
# the COPYING file, which can be found at the root of the source code
# distribution tree, or in https://www.hdfgroup.org/licenses.
# If you do not have access to either file, you may request a copy from
# help@hdfgroup.org.
#

cmake_minimum_required (VERSION 3.18)
project (HDF5_TEST_PAR_API C)

#------------------------------------------------------------------------------
# Define for API tests
#------------------------------------------------------------------------------

set (HDF5_API_TESTS
  attribute
  dataset
  datatype
  file
  group
  link
  misc
  object
)

if (HDF5_TEST_API_ENABLE_ASYNC)
  set (HDF5_API_TESTS
    ${HDF5_API_TESTS}
    async
  )
endif ()

# Ported HDF5 tests
set (HDF5_API_PAR_TESTS_EXTRA
  t_bigio
  t_pshutdown
  t_shapesame
  testphdf5
)

# List of files generated by the HDF5 API tests which
# should be cleaned up in case the test failed to remove
# them
set (HDF5_API_PAR_TESTS_FILES
  H5_api_test_parallel.h5
  H5_api_async_test_parallel.h5
  H5_api_async_test_parallel_0.h5
  H5_api_async_test_parallel_1.h5
  H5_api_async_test_parallel_2.h5
  H5_api_async_test_parallel_3.h5
  H5_api_async_test_parallel_4.h5
  test_file_parallel.h5
  split_comm_file.h5
)

#-----------------------------------------------------------------------------
# Build the main API test executable
#-----------------------------------------------------------------------------
foreach (api_test ${HDF5_API_TESTS})
  set (HDF5_API_PAR_TEST_SRCS
    ${HDF5_API_PAR_TEST_SRCS}
    ${CMAKE_CURRENT_SOURCE_DIR}/H5_api_${api_test}_test_parallel.c
  )
endforeach ()

set (HDF5_API_PAR_TEST_SRCS
  ${HDF5_API_PAR_TEST_SRCS}
  ${CMAKE_CURRENT_SOURCE_DIR}/H5_api_test_parallel.c
  ${HDF5_TEST_API_SRC_DIR}/H5_api_test_util.c
)

add_executable (h5_api_test_parallel ${HDF5_API_PAR_TEST_SRCS})
target_include_directories (
  h5_api_test_parallel
  PRIVATE
    "${HDF5_SRC_INCLUDE_DIRS}"
    "${HDF5_TEST_PAR_DIR}"
    "${HDF5_TEST_API_SRC_DIR}"
    "${HDF5_TEST_API_PAR_SRC_DIR}"
    "${HDF5_SRC_BINARY_DIR}"
    "${HDF5_TEST_BINARY_DIR}"
    "${HDF5_TEST_API_SRC_DIR}"
    "$<$<BOOL:${HDF5_ENABLE_PARALLEL}>:${MPI_C_INCLUDE_DIRS}>"
)
target_compile_options (
  h5_api_test_parallel
  PRIVATE
    "${HDF5_CMAKE_C_FLAGS}"
)
target_compile_definitions (
  h5_api_test_parallel
  PRIVATE
    "$<$<CONFIG:Developer>:${HDF5_DEVELOPER_DEFS}>"
)
if (NOT BUILD_SHARED_LIBS)
  TARGET_C_PROPERTIES (h5_api_test_parallel STATIC)
  target_link_libraries (
    h5_api_test_parallel
    PRIVATE
      ${HDF5_TEST_LIB_TARGET}
      ${HDF5_LIB_TARGET}
      "$<$<BOOL:${HDF5_ENABLE_PARALLEL}>:MPI::MPI_C>"
  )
else ()
  TARGET_C_PROPERTIES (h5_api_test_parallel SHARED)
  target_link_libraries (
    h5_api_test_parallel
    PRIVATE
      ${HDF5_TEST_LIBSH_TARGET}
      ${HDF5_LIBSH_TARGET}
      "$<$<BOOL:${HDF5_ENABLE_PARALLEL}>:MPI::MPI_C>"
  )
endif ()
set_target_properties (
  h5_api_test_parallel
  PROPERTIES
    FOLDER test/par/API
)
# Add Target to clang-format
if (HDF5_ENABLE_FORMATTERS)
  clang_format (HDF5_TEST_h5_api_test_parallel_FORMAT h5_api_test_parallel)
endif ()

if (HDF5_TEST_API_INSTALL)
  install (
    TARGETS
      h5_api_test_parallel
    EXPORT
      ${HDF5_EXPORTED_TARGETS}
    DESTINATION
      ${HDF5_INSTALL_BIN_DIR}
    PERMISSIONS
      OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
    COMPONENT
      tests
  )
endif ()

#-----------------------------------------------------------------------------
# Build the ported HDF5 test executables
#-----------------------------------------------------------------------------
foreach (api_test_extra ${HDF5_API_PAR_TESTS_EXTRA})
  unset (HDF5_API_PAR_TEST_EXTRA_SRCS)

  set (HDF5_API_PAR_TEST_EXTRA_SRCS
    ${HDF5_API_PAR_TEST_EXTRA_SRCS}
    ${CMAKE_CURRENT_SOURCE_DIR}/${api_test_extra}.c
  )

  if (${api_test_extra} STREQUAL "testphdf5")
    set (HDF5_API_PAR_TEST_EXTRA_SRCS
      ${HDF5_API_PAR_TEST_EXTRA_SRCS}
      ${CMAKE_CURRENT_SOURCE_DIR}/t_ph5basic.c
      ${CMAKE_CURRENT_SOURCE_DIR}/t_file.c
      ${CMAKE_CURRENT_SOURCE_DIR}/t_dset.c
      ${CMAKE_CURRENT_SOURCE_DIR}/t_mdset.c
      ${CMAKE_CURRENT_SOURCE_DIR}/t_coll_chunk.c
      ${CMAKE_CURRENT_SOURCE_DIR}/t_span_tree.c
      ${CMAKE_CURRENT_SOURCE_DIR}/t_prop.c
      ${CMAKE_CURRENT_SOURCE_DIR}/t_file_image.c
      ${CMAKE_CURRENT_SOURCE_DIR}/t_coll_md_read.c
      ${CMAKE_CURRENT_SOURCE_DIR}/t_chunk_alloc.c
      ${CMAKE_CURRENT_SOURCE_DIR}/t_filter_read.c
    )
  endif ()

  add_executable (h5_api_test_parallel_${api_test_extra} ${HDF5_API_PAR_TEST_EXTRA_SRCS})
  target_include_directories (
    h5_api_test_parallel_${api_test_extra}
    PRIVATE
      "${HDF5_SRC_INCLUDE_DIRS}"
      "${HDF5_TEST_PAR_DIR}"
      "${HDF5_TEST_API_SRC_DIR}"
      "${HDF5_TEST_API_PAR_SRC_DIR}"
      "${HDF5_SRC_BINARY_DIR}"
      "${HDF5_TEST_BINARY_DIR}"
      "$<$<BOOL:${HDF5_ENABLE_PARALLEL}>:${MPI_C_INCLUDE_DIRS}>"
  )
  target_compile_options (
    h5_api_test_parallel_${api_test_extra}
    PRIVATE
      "${HDF5_CMAKE_C_FLAGS}"
  )
  target_compile_definitions (
    h5_api_test_parallel_${api_test_extra}
    PRIVATE
      "$<$<CONFIG:Developer>:${HDF5_DEVELOPER_DEFS}>"
  )
  if (NOT BUILD_SHARED_LIBS)
    TARGET_C_PROPERTIES (h5_api_test_parallel_${api_test_extra} STATIC)
    target_link_libraries (
      h5_api_test_parallel_${api_test_extra}
      PRIVATE
        ${HDF5_TEST_LIB_TARGET}
        ${HDF5_LIB_TARGET}
        "$<$<BOOL:${HDF5_ENABLE_PARALLEL}>:MPI::MPI_C>"
    )
  else ()
    TARGET_C_PROPERTIES (h5_api_test_parallel_${api_test_extra} SHARED)
    target_link_libraries (
      h5_api_test_parallel_${api_test_extra}
      PRIVATE
        ${HDF5_TEST_LIBSH_TARGET}
        ${HDF5_LIBSH_TARGET}
        "$<$<BOOL:${HDF5_ENABLE_PARALLEL}>:MPI::MPI_C>"
    )
  endif ()
  set_target_properties (
    h5_api_test_parallel_${api_test_extra}
    PROPERTIES
      FOLDER test/par/API
  )
  # Add Target to clang-format
  if (HDF5_ENABLE_FORMATTERS)
    clang_format (HDF5_TEST_h5_api_test_parallel_${api_test_extra}_FORMAT h5_api_test_parallel_${api_test_extra})
  endif ()

  if (HDF5_TEST_API_INSTALL)
    install (
      TARGETS
        h5_api_test_parallel_${api_test_extra}
      EXPORT
        ${HDF5_EXPORTED_TARGETS}
      DESTINATION
        ${HDF5_INSTALL_BIN_DIR}
      PERMISSIONS
        OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
      COMPONENT
        tests
    )
  endif ()
endforeach ()

#-----------------------------------------------------------------------------
# Add tests if HDF5 parallel testing is enabled
#-----------------------------------------------------------------------------
if (HDF5_TEST_PARALLEL)
  if (HDF5_TEST_API_ENABLE_DRIVER)
    if ("${HDF5_TEST_API_SERVER}" STREQUAL "")
      message (FATAL_ERROR "Please set HDF5_TEST_API_SERVER to point to a server executable for the test driver program.")
    endif ()

    # Driver options
    if (HDF5_TEST_API_SERVER_ALLOW_ERRORS)
      set (HDF5_TEST_API_DRIVER_EXTRA_FLAGS --allow-server-errors)
    endif ()
    if (HDF5_TEST_API_CLIENT_HELPER)
      set (HDF5_TEST_API_DRIVER_EXTRA_FLAGS ${HDF5_TEST_API_DRIVER_EXTRA_FLAGS}
        --client-helper ${HDF5_TEST_API_CLIENT_HELPER}
      )
    endif ()
    if (HDF5_TEST_API_CLIENT_INIT)
      set (HDF5_TEST_API_DRIVER_EXTRA_FLAGS ${HDF5_TEST_API_DRIVER_EXTRA_FLAGS}
        --client-init ${HDF5_TEST_API_CLIENT_INIT}
      )
    endif ()

    set (last_api_test "")
    foreach (api_test ${HDF5_API_TESTS})
      add_test (
        NAME "h5_api_test_parallel_${api_test}"
        COMMAND $<TARGET_FILE:h5_api_test_driver>
        --server ${HDF5_TEST_API_SERVER}
        --client $<TARGET_FILE:h5_api_test_parallel> "${api_test}"
        --serial
        ${HDF5_TEST_API_DRIVER_EXTRA_FLAGS}
      )

      set_tests_properties ("h5_api_test_parallel_${api_test}" PROPERTIES DEPENDS "${last_api_test}")

      set (last_api_test "h5_api_test_parallel_${api_test}")
    endforeach ()

    foreach (hdf5_test ${HDF5_API_PAR_TESTS_EXTRA})
      add_test (
        NAME "h5_api_test_parallel_${hdf5_test}"
        COMMAND $<TARGET_FILE:h5_api_test_driver>
        --server ${HDF5_TEST_API_SERVER}
        --client $<TARGET_FILE:h5_api_test_parallel_${hdf5_test}>
        --serial
        ${HDF5_TEST_API_DRIVER_EXTRA_FLAGS}
      )
    endforeach ()

    # Hook external tests to same test suite
    foreach (ext_api_test ${HDF5_API_EXT_PARALLEL_TESTS})
      add_test (
        NAME "h5_api_ext_test_parallel_${ext_api_test}"
        COMMAND $<TARGET_FILE:h5_api_test_driver>
        --server ${HDF5_TEST_API_SERVER}
        --client $<TARGET_FILE:${ext_api_test}>
        --serial
        ${HDF5_TEST_API_DRIVER_EXTRA_FLAGS}
      )
    endforeach ()

    # Add tests for each external VOL connector that was built
    foreach (external_vol_tgt ${HDF5_EXTERNAL_VOL_TARGETS})
      # Determine whether connector should be tested with parallel tests
      get_target_property (vol_test_parallel "${external_vol_tgt}" HDF5_VOL_TEST_PARALLEL)
      if (${vol_test_parallel})
        # Determine environment variables that need to be set for testing
        set (vol_test_env "")
        set (vol_plugin_paths "${CMAKE_BINARY_DIR}/${HDF5_INSTALL_BIN_DIR}")

        get_target_property (vol_test_string "${external_vol_tgt}" HDF5_VOL_NAME)
        list (APPEND vol_test_env "HDF5_VOL_CONNECTOR=${vol_test_string}")

        get_target_property (vol_lib_targets "${external_vol_tgt}" HDF5_VOL_TARGETS)
        foreach (lib_target ${vol_lib_targets})
          get_target_property (lib_target_output_dir "${lib_target}" LIBRARY_OUTPUT_DIRECTORY)
          if (NOT "${lib_target_output_dir}" STREQUAL "lib_target_output_dir-NOTFOUND"
              AND NOT "${lib_target_output_dir}" STREQUAL ""
              AND NOT "${lib_target_output_dir}" STREQUAL "${CMAKE_BINARY_DIR}/${HDF5_INSTALL_BIN_DIR}")
            set (vol_plugin_paths "${vol_plugin_paths}${CMAKE_SEP}${lib_target_output_dir}")
          endif ()
        endforeach ()

        list (APPEND vol_test_env "HDF5_PLUGIN_PATH=${vol_plugin_paths}")

        # Add main API tests
        set (last_api_test "")
        foreach (api_test ${HDF5_API_TESTS})
          add_test (
            NAME "${external_vol_tgt}-h5_api_test_parallel_${api_test}"
            COMMAND $<TARGET_FILE:h5_api_test_driver>
            --server ${HDF5_TEST_API_SERVER}
            --client $<TARGET_FILE:h5_api_test_parallel> "${api_test}"
            --serial
            ${HDF5_TEST_API_DRIVER_EXTRA_FLAGS}
          )
          set_tests_properties (
            "${external_vol_tgt}-h5_api_test_parallel_${api_test}"
            PROPERTIES
              ENVIRONMENT
                "${vol_test_env}"
              WORKING_DIRECTORY
                "${HDF5_TEST_BINARY_DIR}/${external_vol_tgt}"
              DEPENDS
                "${last_api_test}"
          )

          set (last_api_test "${external_vol_tgt}-h5_api_test_parallel_${api_test}")
        endforeach ()

        # Add any extra HDF5 tests
        foreach (hdf5_test ${HDF5_API_PAR_TESTS_EXTRA})
          add_test (
            NAME "${external_vol_tgt}-h5_api_test_parallel_${hdf5_test}"
            COMMAND $<TARGET_FILE:h5_api_test_driver>
            --server ${HDF5_TEST_API_SERVER}
            --client $<TARGET_FILE:h5_api_test_parallel_${hdf5_test}>
            --serial
            ${HDF5_TEST_API_DRIVER_EXTRA_FLAGS}
          )
          set_tests_properties (
            "${external_vol_tgt}-h5_api_test_parallel_${hdf5_test}"
            PROPERTIES
              ENVIRONMENT
                "${vol_test_env}"
              WORKING_DIRECTORY
                "${HDF5_TEST_BINARY_DIR}/${external_vol_tgt}"
          )
        endforeach ()

        # Hook external tests to same test suite
        foreach (ext_api_test ${HDF5_API_EXT_PARALLEL_TESTS})
          add_test (
            NAME "${external_vol_tgt}-h5_api_ext_test_parallel_${ext_api_test}"
            COMMAND $<TARGET_FILE:h5_api_test_driver>
            --server ${HDF5_TEST_API_SERVER}
            --client $<TARGET_FILE:${ext_api_test}>
            --serial
            ${HDF5_TEST_API_DRIVER_EXTRA_FLAGS}
          )
          set_tests_properties (
            "${external_vol_tgt}-h5_api_ext_test_parallel_${ext_api_test}"
            PROPERTIES
              ENVIRONMENT
                "${vol_test_env}"
              WORKING_DIRECTORY
                "${HDF5_TEST_BINARY_DIR}/${external_vol_tgt}"
          )
        endforeach ()
      endif ()
    endforeach ()
  else ()
    set (last_api_test "")
    foreach (api_test ${HDF5_API_TESTS})
      add_test (
        NAME "h5_api_test_parallel_${api_test}"
        COMMAND ${MPIEXEC} ${MPIEXEC_NUMPROC_FLAG} ${MPIEXEC_MAX_NUMPROCS}
          ${MPIEXEC_PREFLAGS} $<TARGET_FILE:h5_api_test_parallel> "${api_test}"
          ${MPIEXEC_POSTFLAGS}
      )

      set_tests_properties ("h5_api_test_parallel_${api_test}" PROPERTIES DEPENDS "${last_api_test}")

      set (last_api_test "h5_api_test_parallel_${api_test}")
    endforeach ()

    foreach (hdf5_test ${HDF5_API_PAR_TESTS_EXTRA})
      add_test (
        NAME "h5_api_test_parallel_${hdf5_test}"
        COMMAND ${MPIEXEC} ${MPIEXEC_NUMPROC_FLAG} ${MPIEXEC_MAX_NUMPROCS}
          ${MPIEXEC_PREFLAGS} $<TARGET_FILE:h5_api_test_parallel_${hdf5_test}>
          ${MPIEXEC_POSTFLAGS}
      )
    endforeach ()

    # Add tests for each external VOL connector that was built
    foreach (external_vol_tgt ${HDF5_EXTERNAL_VOL_TARGETS})
      # Determine whether connector should be tested with parallel tests
      get_target_property (vol_test_parallel "${external_vol_tgt}" HDF5_VOL_TEST_PARALLEL)
      if (${vol_test_parallel})
        # Determine environment variables that need to be set for testing
        set (vol_test_env "")
        set (vol_plugin_paths "${CMAKE_BINARY_DIR}/${HDF5_INSTALL_BIN_DIR}")

        get_target_property (vol_test_string "${external_vol_tgt}" HDF5_VOL_NAME)
        list (APPEND vol_test_env "HDF5_VOL_CONNECTOR=${vol_test_string}")

        get_target_property (vol_lib_targets "${external_vol_tgt}" HDF5_VOL_TARGETS)
        foreach (lib_target ${vol_lib_targets})
          get_target_property (lib_target_output_dir "${lib_target}" LIBRARY_OUTPUT_DIRECTORY)
          if (NOT "${lib_target_output_dir}" STREQUAL "lib_target_output_dir-NOTFOUND"
              AND NOT "${lib_target_output_dir}" STREQUAL ""
              AND NOT "${lib_target_output_dir}" STREQUAL "${CMAKE_BINARY_DIR}/${HDF5_INSTALL_BIN_DIR}")
            set (vol_plugin_paths "${vol_plugin_paths}${CMAKE_SEP}${lib_target_output_dir}")
          endif ()
        endforeach ()

        list (APPEND vol_test_env "HDF5_PLUGIN_PATH=${vol_plugin_paths}")

        # Add main API tests
        set (last_api_test "")
        foreach (api_test ${HDF5_API_TESTS})
          add_test (
            NAME "${external_vol_tgt}-h5_api_test_parallel_${api_test}"
            COMMAND ${MPIEXEC} ${MPIEXEC_NUMPROC_FLAG} ${MPIEXEC_MAX_NUMPROCS}
              ${MPIEXEC_PREFLAGS} $<TARGET_FILE:h5_api_test_parallel> "${api_test}"
              ${MPIEXEC_POSTFLAGS}
          )
          set_tests_properties (
            "${external_vol_tgt}-h5_api_test_parallel_${api_test}"
            PROPERTIES
              ENVIRONMENT
                "${vol_test_env}"
              WORKING_DIRECTORY
                "${HDF5_TEST_BINARY_DIR}/${external_vol_tgt}"
              DEPENDS
                "${last_api_test}"
          )

          set (last_api_test "${external_vol_tgt}-h5_api_test_parallel_${api_test}")
        endforeach ()

        # Add any extra HDF5 tests
        foreach (hdf5_test ${HDF5_API_PAR_TESTS_EXTRA})
          add_test (
            NAME "${external_vol_tgt}-h5_api_test_parallel_${hdf5_test}"
            COMMAND ${MPIEXEC} ${MPIEXEC_NUMPROC_FLAG} ${MPIEXEC_MAX_NUMPROCS}
              ${MPIEXEC_PREFLAGS} $<TARGET_FILE:h5_api_test_parallel_${hdf5_test}>
              ${MPIEXEC_POSTFLAGS}
          )
          set_tests_properties (
            "${external_vol_tgt}-h5_api_test_parallel_${hdf5_test}"
            PROPERTIES
              ENVIRONMENT
                "${vol_test_env}"
              WORKING_DIRECTORY
                "${HDF5_TEST_BINARY_DIR}/${external_vol_tgt}"
          )
        endforeach ()
      endif ()
    endforeach ()
  endif ()
endif ()
