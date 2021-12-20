# Adaptation of https://github.com/capnproto/capnproto-java/blob/2f7ce185bb72f98f804bf1e75f6d9f72655da4ed/cmake/CMakeLists.txt

project(capnproto-java)

cmake_minimum_required(VERSION 3.9)

#REMOVED: Don't care about PkgConfig! We supply PKG_CAPNP_INCLUDE_DIRS and PKG_CAPNP_LDFLAGS
#find_package(PkgConfig REQUIRED)

set(CMAKE_CXX_STANDARD 14)
set(CAPNPC_JAVA_SOURCES ../compiler/src/main/cpp/capnpc-java.c++)

#REMOVED: Don't care about PkgConfig!
#if(CAPNP_PKG_PATH)
#  set(ENV{PKG_CONFIG_PATH} "$ENV{PKG_CONFIG_PATH}:${CAPNP_PKG_PATH}")
#else()
#  find_path(CAPNP_PKG_PATH_VAR "capnp/lib/pkgconfig" HINTS ${CAPNP_PKG_PATH})
#  set(ENV{PKG_CONFIG_PATH} "$ENV{PKG_CONFIG_PATH}:${CAPNP_PKG_PATH_VAR}")
#endif()
#
#pkg_search_module(PKG_CAPNP REQUIRED capnp)

if(PKG_CAPNP_FOUND)
  message(STATUS "Capnp found!")
  add_executable(capnpc-java
    ${CAPNPC_JAVA_SOURCES}
  )

  target_include_directories(capnpc-java
    PRIVATE
      ${PKG_CAPNP_INCLUDE_DIRS}
    # ADDED: Export <.../main/schema>/capnp/java.capnp
    INTERFACE
      $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../compiler/src/main/schema>
  )

  target_link_libraries(capnpc-java
    ${PKG_CAPNP_LDFLAGS}
  )

  # ADDED: Export <.../main/schema>/capnp/java.capnp
  add_library(capnpc-java-schema INTERFACE
      ../compiler/src/main/schema/capnp/java.capnp)
  target_include_directories(capnpc-java-schema
      INTERFACE
      $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../compiler/src/main/schema>
      $<INSTALL_INTERFACE:include>)

  # ADDED: We need to actually install the executable and INTERFACE/PUBLIC files
  add_dependencies(capnpc-java capnpc-java-schema)
  install(TARGETS capnpc-java capnpc-java-schema ${INSTALL_TARGETS_DEFAULT_ARGS})
  install(FILES ../compiler/src/main/schema/capnp/java.capnp DESTINATION include/capnp)
else()
  message(FATAL_ERROR "Capnp not found!")
endif()
