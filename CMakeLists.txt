cmake_minimum_required(VERSION 3.16)

project(proxysocket VERSION 0.1.12 LANGUAGES C)

set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

option(BUILD_SHARED_LIBS "Build shared libraries" OFF)
option(PROXYSOCKET_BUILD_TOOLS "Build command-line tools" ON)
set(PROXYSOCKET_DEBUG_POSTFIX "d" CACHE STRING "Suffix appended to target names in Debug builds")
set(PROXYSOCKET_RELWITHDEBINFO_POSTFIX "rd" CACHE STRING "Suffix appended to target names in RelWithDebInfo builds")
set(PROXYSOCKET_INSTALL_CMAKEDIR "${CMAKE_INSTALL_LIBDIR}/cmake/proxysocket" CACHE STRING "Install path for proxysocket CMake package files")

set(PROXYSOCKET_SOURCES
  src/proxysocket.c
)

set(PROXYSOCKET_HEADERS
  src/proxysocket.h
)

set(PROXYSOCKET_BUILD_INCLUDE_DIRS
  ${CMAKE_CURRENT_SOURCE_DIR}/src
)

if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/include")
  list(APPEND PROXYSOCKET_BUILD_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/include)
endif()

set(PROXYSOCKET_PUBLIC_INCLUDE_DIRS
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/include")
  list(APPEND PROXYSOCKET_PUBLIC_INCLUDE_DIRS
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  )
endif()

set(PROXYSOCKET_WINDOWS_DEFINITIONS)
if(WIN32)
  include(CheckCSourceCompiles)

  set(CMAKE_REQUIRED_DEFINITIONS -D_GNU_SOURCE)
  check_c_source_compiles(
    "#include <stdio.h>\nint main(void) { char* p = 0; return asprintf(&p, \".\"); }"
    PROXYSOCKET_HAVE_ASPRINTF
  )
  unset(CMAKE_REQUIRED_DEFINITIONS)

  list(APPEND PROXYSOCKET_WINDOWS_DEFINITIONS __WIN32__)
  if(PROXYSOCKET_HAVE_ASPRINTF)
    list(APPEND PROXYSOCKET_WINDOWS_DEFINITIONS HAVE_VASPRINTF HAVE_ASPRINTF)
  endif()
endif()

function(proxysocket_apply_compile_options target_name)
  if(MSVC)
    target_compile_options(${target_name} PRIVATE /O1)
  elseif(CMAKE_C_COMPILER_ID MATCHES "GNU|Clang|AppleClang")
    target_compile_options(${target_name} PRIVATE -Os)
  endif()
endfunction()

add_library(proxysocket ${PROXYSOCKET_SOURCES} ${PROXYSOCKET_HEADERS})
set_target_properties(proxysocket PROPERTIES
  OUTPUT_NAME proxysocket
  DEBUG_POSTFIX ${PROXYSOCKET_DEBUG_POSTFIX}
  RELWITHDEBINFO_POSTFIX ${PROXYSOCKET_RELWITHDEBINFO_POSTFIX}
  VERSION ${PROJECT_VERSION}
  SOVERSION ${PROJECT_VERSION_MAJOR}
)
target_compile_definitions(proxysocket PRIVATE
  $<$<BOOL:${BUILD_SHARED_LIBS}>:BUILD_PROXYSOCKET_DLL>
  $<$<NOT:$<BOOL:${BUILD_SHARED_LIBS}>>:BUILD_PROXYSOCKET_STATIC>
)
target_include_directories(proxysocket
  PUBLIC
    ${PROXYSOCKET_PUBLIC_INCLUDE_DIRS}
)
proxysocket_apply_compile_options(proxysocket)

if(WIN32)
  target_compile_definitions(proxysocket PRIVATE ${PROXYSOCKET_WINDOWS_DEFINITIONS})
  target_link_libraries(proxysocket PUBLIC ws2_32)
endif()

if(PROXYSOCKET_BUILD_TOOLS)
  add_executable(ipify examples/ipify.c)
  set_target_properties(ipify PROPERTIES
    DEBUG_POSTFIX ${PROXYSOCKET_DEBUG_POSTFIX}
    RELWITHDEBINFO_POSTFIX ${PROXYSOCKET_RELWITHDEBINFO_POSTFIX}
  )
  target_include_directories(ipify PRIVATE ${PROXYSOCKET_BUILD_INCLUDE_DIRS})
  target_link_libraries(ipify PRIVATE proxysocket)
  proxysocket_apply_compile_options(ipify)

  if(WIN32)
    target_compile_definitions(ipify PRIVATE ${PROXYSOCKET_WINDOWS_DEFINITIONS})
  endif()

  add_custom_target(tools DEPENDS ipify)
endif()

find_package(Doxygen QUIET)
if(DOXYGEN_FOUND)
  add_custom_target(doc
    COMMAND ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/doc/Doxyfile
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    COMMENT "Generating API documentation with Doxygen"
    VERBATIM
  )
endif()

install(FILES ${PROXYSOCKET_HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

install(TARGETS proxysocket
  EXPORT proxysocketTargets
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

if(TARGET ipify)
  install(TARGETS ipify
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  )
endif()

configure_package_config_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/cmake/proxysocketConfig.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/proxysocketConfig.cmake
  INSTALL_DESTINATION ${PROXYSOCKET_INSTALL_CMAKEDIR}
)

write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/proxysocketConfigVersion.cmake
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY SameMajorVersion
)

install(EXPORT proxysocketTargets
  FILE proxysocketTargets.cmake
  NAMESPACE proxysocket::
  DESTINATION ${PROXYSOCKET_INSTALL_CMAKEDIR}
)

install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/proxysocketConfig.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/proxysocketConfigVersion.cmake
  DESTINATION ${PROXYSOCKET_INSTALL_CMAKEDIR}
)
