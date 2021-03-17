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

# should point to DjVuLibre sources
SET(djvu_libre_sources_path "./djvulibre/")
SET(libdjvu "${djvu_libre_sources_path}libdjvu/" )

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

#some required params which aren't checked in hope your system have them
add_compile_definitions(HAVE_STDINCLUDES HAS_WCHAR HAVE_MBSTATE_T HAVE_NAMESPACES)

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
                    "${djvu_libre_sources_path}libdjvu/" # should contain GString.h etc.
		    "${djvu_libre_sources_path}tools/"   # should contain common.h
)

FIND_LIBRARY(
        FI_LIBRARY freeimage
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
        djvu_libre_sources
	${libdjvu}Arrays.cpp ${libdjvu}ByteStream.cpp
	${libdjvu}DjVuFile.cpp ${libdjvu}DjVuInfo.cpp
	${libdjvu}DjVuMessage.cpp ${libdjvu}DjVuMessageLite.cpp
	${libdjvu}GBitmap.cpp ${libdjvu}GContainer.cpp ${libdjvu}GException.cpp
	${libdjvu}GOS.cpp ${libdjvu}GPixmap.cpp ${libdjvu}GRect.cpp ${libdjvu}GSmartPointer.cpp
	${libdjvu}GString.cpp ${libdjvu}GThreads.cpp ${libdjvu}GURL.cpp ${libdjvu}GUnicode.cpp
	${libdjvu}IFFByteStream.cpp ${libdjvu}IW44EncodeCodec.cpp ${libdjvu}IW44Image.cpp
	${libdjvu}JB2EncodeCodec.cpp
	${libdjvu}MMX.cpp ${libdjvu}UnicodeByteStream.cpp ${libdjvu}XMLParser.cpp ${libdjvu}XMLTags.cpp
	${libdjvu}ZPCodec.cpp
)

SET(
        sources
        src/c44-fi.cpp
)


IF (WIN32)
        ADD_EXECUTABLE( c44-fi WIN32 ${sources} ${djvu_libre_sources})
	add_compile_definitions(_WIN32)
ELSEIF(APPLE)
        ADD_EXECUTABLE( c44-fi MACOSX_BUNDLE ${sources} ${djvu_libre_sources})
	add_compile_definitions(macintosh)
ELSE()
        ADD_EXECUTABLE( c44-fi ${sources} ${djvu_libre_sources})
	add_compile_definitions(UNIX)
ENDIF()

TARGET_LINK_LIBRARIES(
        c44-fi ${FI_LIBRARY} ${LIBRE_LIBRARY}
)

IF(APPLE)
        INSTALL(TARGETS c44-fi BUNDLE DESTINATION . RUNTIME DESTINATION bin)
ELSE(APPLE)
        INSTALL(TARGETS c44-fi RUNTIME DESTINATION bin)
ENDIF(APPLE)
