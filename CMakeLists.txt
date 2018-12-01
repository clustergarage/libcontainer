cmake_minimum_required(VERSION 3.10)
project(libcontainer VERSION 0.1.0 LANGUAGES CXX)

include(CMakePackageConfigHelpers)

set(LIBCONTAINER_INSTALL ON)
set(LIBCONTAINER_HEADERS ${PROJECT_SOURCE_DIR}/include/container_util.h)
set(LIBCONTAINER_SOURCES ${PROJECT_SOURCE_DIR}/src/container_util.cc)

add_library(container ${LIBCONTAINER_SOURCES}
  ${LIBCONTAINER_HEADERS})

target_include_directories(container PUBLIC
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>)

set_target_properties(container PROPERTIES
  VERSION ${PROJECT_VERSION}
  SOVERSION 1)

# Install targets.
if(LIBCONTAINER_INSTALL)
  include(GNUInstallDirs)
  include(CMakePackageConfigHelpers)

  set(INSTALL_TARGETS container)
  set(LIBCONTAINER_CMAKE_DIR "${CMAKE_INSTALL_LIBDIR}/cmake/libcontainer" CACHE STRING
    "Installation directory for cmake files, relative to ${CMAKE_INSTALL_PREFIX}.")
  set(_LIBCONTAINER_VERSION_CONFIG "${PROJECT_BINARY_DIR}/libcontainer-config-version.cmake")
  set(_LIBCONTAINER_PROJECT_CONFIG "${PROJECT_BINARY_DIR}/libcontainer-config.cmake")
  set(_LIBCONTAINER_PKGCONFIG "${PROJECT_BINARY_DIR}/libcontainer.pc")
  set(_LIBCONTAINER_TARGETS_EXPORT_NAME libcontainer-targets)
  set(LIBCONTAINER_LIB_DIR ${CMAKE_INSTALL_LIBDIR} CACHE STRING
    "Installation directory for libraries, relative to ${CMAKE_INSTALL_PREFIX}.")
  set(LIBCONTAINER_INCLUDE_DIR "${CMAKE_INSTALL_INCLUDEDIR}/libcontainer" CACHE STRING
    "Installation directory for include files, relative to ${CMAKE_INSTALL_PREFIX}.")
  set(LIBCONTAINER_PKGCONFIG_DIR "${CMAKE_INSTALL_PREFIX}/share/pkgconfig" CACHE PATH
    "Installation directory for pkgconfig (.pc) files, relative to ${CMAKE_INSTALL_PREFIX}.")

  # Generate the version, config and target files into the build directory.
  write_basic_package_version_file(${_LIBCONTAINER_VERSION_CONFIG}
    VERSION ${LIBCONTAINER_VERSION}
    COMPATIBILITY AnyNewerVersion)
  configure_file("${PROJECT_SOURCE_DIR}/support/cmake/libcontainer.pc.in"
    "${_LIBCONTAINER_PKGCONFIG}" @ONLY)
  configure_package_config_file("${PROJECT_SOURCE_DIR}/support/cmake/libcontainer-config.cmake.in"
    ${_LIBCONTAINER_PROJECT_CONFIG}
    INSTALL_DESTINATION ${LIBCONTAINER_CMAKE_DIR})
  # Use a namespace because CMake provides better diagnostics for namespaced
  # imported targets.
  export(TARGETS ${INSTALL_TARGETS}
    NAMESPACE clustergarage::container::
    FILE "${PROJECT_BINARY_DIR}/${_LIBCONTAINER_TARGETS_EXPORT_NAME}.cmake")

  # Install version, config and target files.
  install(FILES ${_LIBCONTAINER_PROJECT_CONFIG} ${_LIBCONTAINER_VERSION_CONFIG}
    DESTINATION ${LIBCONTAINER_CMAKE_DIR})
  install(EXPORT ${_LIBCONTAINER_TARGETS_EXPORT_NAME}
    DESTINATION ${LIBCONTAINER_CMAKE_DIR}
    NAMESPACE clustergarage::container::)

  # Install the library and headers.
  install(TARGETS ${INSTALL_TARGETS}
    EXPORT ${_LIBCONTAINER_TARGETS_EXPORT_NAME}
    DESTINATION ${LIBCONTAINER_LIB_DIR})
  install(FILES ${LIBCONTAINER_HEADERS}
    DESTINATION ${LIBCONTAINER_INCLUDE_DIR})
  install(FILES ${_LIBCONTAINER_PKGCONFIG}
    DESTINATION ${LIBCONTAINER_PKGCONFIG_DIR})
endif()
