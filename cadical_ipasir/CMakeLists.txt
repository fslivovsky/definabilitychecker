cmake_minimum_required(VERSION 3.12)
project(cadical_bindings)

# Run the configure script
execute_process(COMMAND ./configure WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/cadical)

add_custom_target(
  cadical_make
  COMMAND make
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/cadical
)

add_library(cadical_external SHARED IMPORTED )
set_target_properties(cadical_external PROPERTIES IMPORTED_LOCATION ${CMAKE_CURRENT_SOURCE_DIR}/build/libcadical.a )

add_library (cadical_library cadical.h cadical.cc)
target_include_directories(cadical_library PRIVATE cadical/src)

#target_compile_options(cadical_library PRIVATE -fPIC)
target_link_libraries(cadical_library interrupt cadical_external)
