cmake_minimum_required (VERSION 3.5)

add_subdirectory(micro-ecc)
set_property(TARGET microecc PROPERTY FOLDER "3rdparty")

add_subdirectory(fonts)

SET(UTF8PROC_INSTALL OFF CACHE BOOL "Enable installation of utf8proc" FORCE)
add_subdirectory(utf8proc)
set_property(TARGET utf8proc PROPERTY FOLDER "3rdparty")

add_subdirectory(launchinfo)
set_property(TARGET launchinfo PROPERTY FOLDER "3rdparty")

SET(RE2_BUILD_TESTING OFF CACHE BOOL "enable testing for RE2" FORCE)
add_subdirectory(re2 EXCLUDE_FROM_ALL)
target_include_directories(re2 PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/re2>
)
set_property(TARGET re2 PROPERTY FOLDER "3rdparty")
set_property(TARGET re2 PROPERTY XCODE_ATTRIBUTE_GCC_WARN_ABOUT_MISSING_FIELD_INITIALIZERS NO)	# -Wmissing-field-initializers
set_property(TARGET re2 PROPERTY XCODE_ATTRIBUTE_WARNING_CFLAGS "-Wno-missing-field-initializers")

add_subdirectory(EmbeddedJSONSignature)
set_property(TARGET EmbeddedJSONSignature PROPERTY FOLDER "3rdparty")

if(ENABLE_DISCORD)
	add_subdirectory(discord-rpc EXCLUDE_FROM_ALL)
	set_property(TARGET discord-rpc PROPERTY FOLDER "3rdparty")
endif()

# glad library

set(GLAD_HEADERS
	glad/include/glad/glad.h
	glad/include/KHR/khrplatform.h
)

set(GLAD_SOURCES
	glad/src/glad.c
)

add_library(glad STATIC ${GLAD_HEADERS} ${GLAD_SOURCES})
set_property(TARGET glad PROPERTY FOLDER "3rdparty")
target_link_libraries(glad PRIVATE ${CMAKE_DL_LIBS})
target_include_directories(glad PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/glad/include")

# quickjs library

set(QUICKJS_HEADERS
	quickjs/cutils.h
#	quickjs/libbf.h
	quickjs/libregexp.h
	quickjs/libunicode.h
	quickjs/list.h
	quickjs/quickjs.h
	# extensions
	quickjs/quickjs-debugger.h
	quickjs/quickjs-limitedcontext.h
)

set(QUICKJS_SOURCES
	quickjs/cutils.c
#	quickjs/libbf.c
	quickjs/libregexp.c
	quickjs/libunicode.c
	quickjs/quickjs.c
	# extensions
	# INCLUDED DIRECTLY IN QUICKJS.C: quickjs/quickjs-debugger.c
	# INCLUDED DIRECTLY IN QUICKJS.C: quickjs/quickjs-limitedcontext.c
)

file(STRINGS "quickjs/VERSION" QUICKJS_VERSION_STR)

INCLUDE(CheckIncludeFiles)
CHECK_INCLUDE_FILES("sys/time.h" HAVE_SYS_TIME_H LANGUAGE C)
add_library(quickjs STATIC ${QUICKJS_HEADERS} ${QUICKJS_SOURCES})
set_property(TARGET quickjs PROPERTY FOLDER "3rdparty")
target_include_directories(quickjs PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/quickjs")
target_compile_definitions(quickjs PRIVATE
	CONFIG_VERSION="${QUICKJS_VERSION_STR}"
	_GNU_SOURCE
	QUICKJS_DISABLE_ATOMICS
)
if(HAVE_SYS_TIME_H)
	target_compile_definitions(quickjs PRIVATE QUICKJS_HAVE_SYS_TIME_H)
endif()
set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
set(THREADS_PREFER_PTHREAD_FLAG TRUE)
find_package(Threads REQUIRED)
target_link_libraries(quickjs PRIVATE Threads::Threads)
if(MSVC)
	# Disable some warnings for QuickJS
	set(QUICKJS_COMPILE_OPTIONS "")

	# warning C4018: '<': signed/unsigned mismatch
	set(QUICKJS_COMPILE_OPTIONS ${QUICKJS_COMPILE_OPTIONS} /wd4018)

	# warning C4191: 'type cast': unsafe conversion from 'JSCFunctionMagic (__cdecl *)' to 'JSCFunction (__cdecl *)'
	set(QUICKJS_COMPILE_OPTIONS ${QUICKJS_COMPILE_OPTIONS} /wd4191)

	target_compile_options(quickjs PRIVATE ${QUICKJS_COMPILE_OPTIONS})
	target_compile_definitions(quickjs PRIVATE "_CRT_SECURE_NO_WARNINGS")
endif()
