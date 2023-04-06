cmake_minimum_required(VERSION 3.12)
project(definabilitychecker LANGUAGES CXX)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

option(STATIC_BUILD "Build a static binary." OFF)
if(STATIC_BUILD)
  set(CMAKE_FIND_LIBRARY_SUFFIXES ".a")
  set(BUILD_SHARED_LIBS OFF)
  set(CMAKE_EXE_LINKER_FLAGS "-static")
  set(Boost_USE_STATIC_LIBS        ON)
endif()

set(CMAKE_CXX_FLAGS_RELEASE "-O3 -DNDEBUG")
add_compile_options (-fpic)

include_directories(include)
include_directories(interrupt)

add_subdirectory(interrupt)
add_subdirectory(cadical_ipasir)
add_subdirectory(abc-ufo EXCLUDE_FROM_ALL)
add_subdirectory(minisat EXCLUDE_FROM_ALL)
add_subdirectory(avy)
add_subdirectory(interpolatingsolver)

find_package(Abc REQUIRED)
if (ABC_FOUND)
  include_directories(${ABC_INCLUDE_DIR})
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${ABC_CXXFLAGS}")
  add_definitions (-DABC_NAMESPACE=${ABC_NAMESPACE})
else()
  set(ABC_LIBRARY "")
endif ()

find_package(Minisat REQUIRED)
if (MINISAT_FOUND)
  include_directories(${MINISAT_INCLUDE_DIR})
else()
  set (MINISAT_LIBRARY "")
endif()

find_package(Avy REQUIRED)
if (AVY_FOUND)
  include_directories(${AVY_SOURCE_DIR} ${AVY_INCLUDE_DIR})
endif ()

find_package(InterpolatingSolver REQUIRED)
if (InterpolatingSolver_FOUND)
  include_directories(${INTERPOLATING_SOLVER_INCLUDE_DIR})
endif()

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

include_directories(cadical_ipasir)

add_library(ITPsolver src/ITPsolver.h src/ITPsolver.cc)
target_link_libraries(ITPsolver PRIVATE ${INTERPOLATING_SOLVER_LIBRARY})

add_library(definabilitychecker include/definabilitychecker.h src/definabilitychecker.cc)
target_link_libraries(definabilitychecker PRIVATE ITPsolver cadical_library)

install(TARGETS definabilitychecker DESTINATION lib)