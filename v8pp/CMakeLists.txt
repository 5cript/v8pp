# v8pp target

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)

configure_file(config.hpp.in ${CMAKE_CURRENT_BINARY_DIR}/config.hpp)

if(V8_COMPRESS_POINTERS)
	set(V8PP_DEFINES V8_COMPRESS_POINTERS V8_31BIT_SMIS_ON_64BIT_ARCH)
endif()

set(V8PP_HEADERS
	${CMAKE_CURRENT_BINARY_DIR}/config.hpp
	call_from_v8.hpp
	call_v8.hpp
	class.hpp
	context.hpp
	convert.hpp
	factory.hpp
	function.hpp
	json.hpp
	module.hpp
	object.hpp
	property.hpp
	ptr_traits.hpp
	throw_ex.hpp
	utility.hpp
	version.hpp
)

if(V8PP_HEADER_ONLY)
	list(APPEND V8PP_HEADERS
		class.ipp
		json.ipp
		throw_ex.ipp
		version.ipp
	)
	set(V8PP_SOURCES)
else()
	set(V8PP_SOURCES
		class.cpp
		context.cpp
		convert.cpp
		json.cpp
		throw_ex.cpp
		version.cpp
	)
endif()

#set(CMAKE_CXX_RTTI OFF)
if(MSVC)
	set(V8PP_COMPILE_OPTIONS -fno-rtti -fexceptions -Wall -Wextra -Wpedantic)
endif()

if(V8PP_HEADER_ONLY)
	add_library(v8pp INTERFACE)
	target_compile_definitions(v8pp INTERFACE ${V8PP_DEFINES})
	target_compile_options(v8pp INTERFACE ${V8PP_COMPILE_OPTIONS})
	target_include_directories(v8pp INTERFACE
		$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>
		$<BUILD_INTERFACE:${PROJECT_BINARY_DIR}>
		$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)
	target_link_libraries(v8pp INTERFACE v8 v8_libplatform)
else()
	add_library(v8pp ${V8PP_HEADERS} ${V8PP_SOURCES})
	target_compile_definitions(v8pp PUBLIC ${V8PP_DEFINES})
	target_compile_options(v8pp PUBLIC ${V8PP_COMPILE_OPTIONS})
	target_include_directories(v8pp PUBLIC
		$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>
		$<BUILD_INTERFACE:${PROJECT_BINARY_DIR}>
		$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)
	target_link_libraries(v8pp PUBLIC v8 v8_libplatform)
	if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
		target_link_libraries(v8pp PUBLIC v8 libc++)
	else()
		target_link_libraries(v8pp PUBLIC v8 libstdc++)
	endif()
	if(BUILD_SHARED_LIBS)
		target_link_libraries(v8pp PUBLIC ${CMAKE_DL_LIBS})
	endif()
endif()

#source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} FILES ${V8PP_HEADERS} ${V8PP_SOURCES})
