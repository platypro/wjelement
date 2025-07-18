project(WJElement C)
cmake_minimum_required(VERSION 3.0)
include(CheckIncludeFiles)
include(CheckSymbolExists)

option(WJE_BUILD_CLI "Build the command line interface" ON)

option(WJE_STATIC_LIB "Compile as static libraries" OFF)
if(WJE_STATIC_LIB)
  set(BUILD_SHARED_LIBS OFF)
  add_definitions(-DCOMPILE_AS_STATIC)
else(WJE_STATIC_LIB)
  set(BUILD_SHARED_LIBS ON)
endif(WJE_STATIC_LIB)

option(WJE_DISTINGUISH_INTEGERS "Distinguish between integer and non-integer numbers" OFF)
if(WJE_DISTINGUISH_INTEGERS)
  add_definitions(-DWJE_DISTINGUISH_INTEGER_TYPE)
endif()


if("${CMAKE_SYSTEM}" MATCHES "Linux")
  add_definitions(-D_GNU_SOURCE)
endif()

if(NOT CMAKE_INSTALL_PREFIX)
  set(CMAKE_INSTALL_PREFIX /usr/local)
endif(NOT CMAKE_INSTALL_PREFIX)
set(INC_DEST_DIR ${CMAKE_INSTALL_PREFIX}/include)
if(NOT LIB_DEST_DIR)
  set(LIB_DEST_DIR ${CMAKE_INSTALL_PREFIX}/lib)
endif(NOT LIB_DEST_DIR)
set(BIN_DEST_DIR ${CMAKE_INSTALL_PREFIX}/bin)

if(CMAKE_BUILD_TYPE MATCHES Debug)
  add_definitions(-DDEBUG)
  option(WJE_SHOWNOTES "Show preprocessor notes" OFF)

  if(CMAKE_COMPILER_IS_GNUCC)
	# GCC specific debug options
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O0 -g3 -ggdb3 -gdwarf-2")
	set(AVOID_VERSION -avoid-version)
  endif(CMAKE_COMPILER_IS_GNUCC)
endif(CMAKE_BUILD_TYPE MATCHES Debug)

add_definitions(-DHAVE_CONFIG_H)

check_include_files(string.h HAVE_STRING_H)
check_symbol_exists(strcasecmp "strings.h" HAVE_STRCASECMP)

check_include_files(regex.h HAVE_REGEX_H)
if(HAVE_REGEX_H)
	add_definitions(-DHAVE_REGEX_H)
else(HAVE_REGEX_H)
	message("***  WARNING: GNU C regex library not found.")
	message("     WJESchemaValidate() will not support:")
	message("         pattern")
	message("         patternProperties")
	message("         format")
endif(HAVE_REGEX_H)

# use, i.e. don't skip the full RPATH for the build tree
SET(CMAKE_SKIP_BUILD_RPATH  FALSE)
# when building, don't use the install RPATH already
# (but later on when installing)
SET(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)
# the RPATH to be used when installing
SET(CMAKE_INSTALL_RPATH ${LIB_DEST_DIR})
# add the automatically determined parts of the RPATH
# which point to directories outside the build tree to the install RPATH
SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

ENABLE_TESTING(1)

INSTALL(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/documentation/
	DESTINATION doc/wjelement/
)

include_directories(include)
add_subdirectory(src)
add_subdirectory(include)

# pkg-config .pc
SET(PKG_CONFIG_REQUIRES glib-2.0)
SET(PKG_CONFIG_LIBDIR
    ${LIB_DEST_DIR}
)
SET(PKG_CONFIG_INCLUDEDIR
	${INC_DEST_DIR}
)
SET(PKG_CONFIG_LIBS
    "-L\${libdir} -lwjelement -lwjreader -lwjwriter"
)
SET(PKG_CONFIG_CFLAGS
    "-I\${includedir}"
)
CONFIGURE_FILE(
  "${CMAKE_CURRENT_SOURCE_DIR}/pkg-config.pc.cmake"
  "${CMAKE_CURRENT_BINARY_DIR}/wjelement.pc"
)
INSTALL(FILES "${CMAKE_CURRENT_BINARY_DIR}/wjelement.pc"
        DESTINATION ${LIB_DEST_DIR}/pkgconfig)

# uninstall target
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
    IMMEDIATE @ONLY)

add_custom_target(uninstall
    COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)

SET(MAJOR_VERSION 1)
SET(MINOR_VERSION 0)
SET(PATCH_VERSION 1)
IF(EXISTS "${CMAKE_ROOT}/Modules/CPack.cmake")

INCLUDE (InstallRequiredSystemLibraries)

SET (CPACK_SET_DESTDIR "on")
SET (CPACK_PACKAGING_INSTALL_PREFIX "/tmp")
SET (CPACK_GENERATOR "DEB")

SET (CPACK_DEBIAN_PACKAGE_PRIORITY "extra")
SET (CPACK_DEBIAN_PACKAGE_SECTION "libs")
SET (CPACK_DEBIAN_ARCHITECTURE ${CMAKE_SYSTEM_PROCESSOR})
# autogenerate dependency information
set (CPACK_DEBIAN_PACKAGE_SHLIBDEPS ON)

SET (CPACK_PACKAGE_DESCRIPTION "Short description")
SET (CPACK_PACKAGE_DESCRIPTION_SUMMARY "Long description")
SET (CPACK_PACKAGE_VENDOR "Titan Lien (digbil)")
SET (CPACK_PACKAGE_CONTACT "titan@digbil.com")
SET (CPACK_PACKAGE_VERSION_MAJOR "${MAJOR_VERSION}")
SET (CPACK_PACKAGE_VERSION_MINOR "${MINOR_VERSION}")
SET (CPACK_PACKAGE_VERSION_PATCH "${PATCH_VERSION}")

SET (CPACK_PACKAGE_FILE_NAME "${CMAKE_PROJECT_NAME}_${MAJOR_VERSION}.${MINOR_VERSION}.${CPACK_PACKAGE_VERSION_PATCH}")
SET (CPACK_SOURCE_PACKAGE_FILE_NAME "${CMAKE_PROJECT_NAME}_${MAJOR_VERSION}.${MINOR_VERSION}.${CPACK_PACKAGE_VERSION_PATCH}")

SET (CPACK_COMPONENTS_ALL Libraries ApplicationData)
INCLUDE (CPack)
ENDIF(EXISTS "${CMAKE_ROOT}/Modules/CPack.cmake")

