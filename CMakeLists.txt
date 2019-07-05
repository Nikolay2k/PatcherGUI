cmake_minimum_required(VERSION 3.13)

set(MODULE_NAME PatcherGUI)
project(${MODULE_NAME})

set_property(GLOBAL PROPERTY USE_FOLDERS ON)
set(CMAKE_POSITION_INDEPENDENT_CODE True)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

set(CMAKE_CXX_STANDARD 17) 
set(CMAKE_CXX_STANDARD_REQUIRED ON)

include(build_options)

if (CMAKE_RUNTIME_OUTPUT_DIRECTORY)
else()
  set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${dbpg_BINARY_DIR}/bin)
endif()

message("CMAKE_RUNTIME_OUTPUT_DIRECTORY = ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}")

#------------------------------------------------------------------------------
#  Setup
#------------------------------------------------------------------------------

if (WIN32)
  set(PostgreSQL_INCLUDE_DIR      "$ENV{PostgreSQL_ROOT}/include")
  set(PostgreSQL_TYPE_INCLUDE_DIR "$ENV{PostgreSQL_ROOT}/include")
  set(PostgreSQL_LIBRARY_DIR      "$ENV{PostgreSQL_ROOT}/lib")
endif()
find_package(PostgreSQL QUIET)

if (PostgreSQL_FOUND AND PostgreSQL_LIBRARY)
  set(PostgreSQL_FOUND TRUE PARENT_SCOPE)
else()
  set(PostgreSQL_FOUND FALSE PARENT_SCOPE)
endif()

if (PostgreSQL_FOUND)
  set(SKIP_BUILD_TEST  ON CACHE BOOL "" FORCE)
  set(SKIP_PQXX_SHARED ON CACHE BOOL "" FORCE)
  add_subdirectory(dependencies/pqxx)
  set(pqxx_libs pqxx_static ${PostgreSQL_LIBRARY})
else()
  set(pqxx_libs "")
endif()
#----------------------------------------------

message("pqxx_libs = ${pqxx_libs}")

#---------------------------------
set(PatcherGUI_INCS
  include/PatcherGUI.h
  include/Resource.h
  include/stdafx.h
  include/targetver.h
  )

set(PatcherGUI_SRC
  src/PatcherGUI.cpp
  src/stdafx.cpp
  )

set(PatcherGUI_res
  res/PatcherGUI.ico
  res/PatcherGUI.rc
  res/small.ico
)
#---------------------------------
target_include_directories(${MODULE_NAME}
  PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
  )
add_executable(${MODULE_NAME} src/PatcherGUI.cpp)
target_link_libraries(${MODULE_NAME}_test PRIVATE ${pqxx_libs} ${FILESYSTEM_LIBRARY})

set_target_properties(${MODULE_NAME} PROPERTIES
  MACOSX_BUNDLE TRUE
  MACOSX_FRAMEWORK_IDENTIFIER org.cmake.${MODULE_NAME}
  RESOURCE "${PatcherGUI_res}"
)
#---------------------------------
if (PostgreSQL_FOUND)
  if (WIN32)
    find_path(pqdll_path NAMES libpq.dll PATHS ${PostgreSQL_LIBRARY_DIR} ${PostgreSQL_ROOT}/bin)
    add_custom_command(TARGET ${MODULE_NAME} POST_BUILD
      COMMAND "${CMAKE_COMMAND}" -E copy_if_different "${pqdll_path}/libpq.dll"      "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/$<CONFIG>/libpq.dll"
      COMMAND "${CMAKE_COMMAND}" -E copy_if_different "${pqdll_path}/libeay32.dll"   "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/$<CONFIG>/libeay32.dll"
      COMMAND "${CMAKE_COMMAND}" -E copy_if_different "${pqdll_path}/libiconv-2.dll" "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/$<CONFIG>/libiconv-2.dll"
      COMMAND "${CMAKE_COMMAND}" -E copy_if_different "${pqdll_path}/libintl-8.dll"  "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/$<CONFIG>/libintl-8.dll"
      COMMAND "${CMAKE_COMMAND}" -E copy_if_different "${pqdll_path}/ssleay32.dll"   "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/$<CONFIG>/ssleay32.dll"
      COMMAND "${CMAKE_COMMAND}" -E copy_if_different "${pqdll_path}/zlib1.dll"      "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/$<CONFIG>/zlib1.dll"
      )
    unset(pqdll_path)
  endif(WIN32)
endif(PostgreSQL_FOUND)