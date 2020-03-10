CMAKE_MINIMUM_REQUIRED(VERSION 2.8.9)

IF(POLICY CMP0003)
	# Allow linking with full library path instead of -L/path -llibrary
	CMAKE_POLICY(SET CMP0003 NEW)
ENDIF()

SET(
        CMAKE_USER_MAKE_RULES_OVERRIDE
        "${CMAKE_SOURCE_DIR}/cmake/default_cflags.cmake"
)
SET(
        CMAKE_USER_MAKE_RULES_OVERRIDE_CXX
        "${CMAKE_SOURCE_DIR}/cmake/default_cxxflags.cmake"
)


PROJECT("FreeImage version of c44 encoder")

# For config.h
INCLUDE_DIRECTORIES("${CMAKE_CURRENT_BINARY_DIR}")

INCLUDE(CheckIncludeFile)
INCLUDE(cmake/FindPthreads.cmake)
INCLUDE(cmake/SetDefaultBuildType.cmake)
INCLUDE(cmake/SetDefaultGccFlags.cmake)

SET_DEFAULT_BUILD_TYPE(Release)
IF(CMAKE_COMPILER_IS_GNUCC)
        SET_DEFAULT_GCC_FLAGS()
ENDIF(CMAKE_COMPILER_IS_GNUCC)

IF(UNIX)
	FindPthreads()
	IF(PTHREADS_FOUND)
		ADD_DEFINITIONS(${PTHREADS_CFLAGS})
		LINK_LIBRARIES(${PTHREADS_LIBS})
		add_compile_definitions(HAVE_PTHREAD)
	ELSE(PTHREADS_FOUND)
		MESSAGE(
			FATAL_ERROR
			"Could not detect threading flags.\n"
			"Try specifying them manually in PTHREADS_CFLAGS and PTHREADS_LIBS."
		)
	ENDIF(PTHREADS_FOUND)
ELSEIF(WIN32)
        ADD_DEFINITIONS(-DNOMINMAX)
ENDIF(UNIX)

CHECK_INCLUDE_FILE(stdint.h HAVE_STDINT_H)
IF(NOT HAVE_STDINT_H)
        CONFIGURE_FILE(compat/pstdint.h "${CMAKE_CURRENT_BINARY_DIR}/stdint.h" COPYONLY)
        INCLUDE_DIRECTORIES("${CMAKE_CURRENT_BINARY_DIR}")
ELSE(NOT HAVE_STDINT_H)
        FILE(REMOVE "${CMAKE_CURRENT_BINARY_DIR}/stdint.h")
        add_compile_definitions(HAVE_STDINT_H)
ENDIF(NOT HAVE_STDINT_H)

FIND_PATH(
	FI_INCLUDE_DIR FreeImage.h
	PATHS /usr/local/include /usr/include
	DOC "Path to libfreeimage headers."
)
IF(NOT FI_INCLUDE_DIR)
        MESSAGE(
                FATAL_ERROR
                "Could not find libfreeimage headers.\n"
                "You may need to install a package named libfreeimage-dev or similarly."
        )
ENDIF()

INCLUDE_DIRECTORIES("${FI_INCLUDE_DIR}" 
                    "../../djvulibre-3.5.27/libdjvu/"  # should contain GString.h etc.
)

FIND_LIBRARY(
	FI_LIBRARY libfreeimage.so
	PATHS /usr/local/lib /usr/lib
	DOC "Path to libfreeimage library."
)
IF(NOT FI_LIBRARY)
        MESSAGE(
                FATAL_ERROR
                "Could not find libfreeimage library.\n"
                "You may need to install a package named libfreeimage-dev or similarly."
        )
ENDIF()

SET(
        sources
        src/fi_c44.cpp
)

SOURCE_GROUP("Sources" FILES ${sources})

IF (WIN32)
        ADD_EXECUTABLE( fi_c44 WIN32 ${sources} )
ELSEIF(APPLE)
        ADD_EXECUTABLE( fi_c44 MACOSX_BUNDLE ${sources} )
ELSE()
        ADD_EXECUTABLE( fi_c44 ${sources} )
ENDIF()

TARGET_LINK_LIBRARIES(
        fi_c44 ${FI_LIBRARY}
)

IF(APPLE)
        INSTALL(TARGETS fi_c44 BUNDLE DESTINATION . RUNTIME DESTINATION bin)
ELSE(APPLE)
        INSTALL(TARGETS fi_c44 RUNTIME DESTINATION bin)
ENDIF(APPLE)
