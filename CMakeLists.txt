cmake_minimum_required(VERSION 3.4)
project(stb C)

option(STB_EXPORT "Export stb source tree as package" true)
option(STB_INSTALL "Generate stb installation target" true)

set(STB_INSTALL_INCLUDE_DIR "include")

set(STB_FILES "stb_image.h;stb_image_write.h;stb_image_resize2.h;stb_perlin.h")
set(STB_DEFINES "STB_IMAGE_IMPLEMENTATION;STB_IMAGE_WRITE_IMPLEMENTATION;STB_IMAGE_RESIZE_IMPLEMENTATION;STB_PERLIN_IMPLEMENTATION")
set(STB_NAMES "image;image-write;image-resize;perlin")
list(LENGTH STB_FILES STB_LIBRARY_COUNT)
math(EXPR STB_LIBRARY_COUNT "${STB_LIBRARY_COUNT} - 1")


foreach(NTH_STB_LIBRARY RANGE 0 ${STB_LIBRARY_COUNT})
	list(GET STB_FILES ${NTH_STB_LIBRARY} STB_FILE)
	list(GET STB_NAMES ${NTH_STB_LIBRARY} STB_NAME)
	list(GET STB_DEFINES ${NTH_STB_LIBRARY} STB_DEFINE)
	
	configure_file(stb/${STB_FILE} stb/${STB_NAME}.c COPYONLY)
	add_library(${STB_NAME} ${CMAKE_CURRENT_BINARY_DIR}/stb/${STB_NAME}.c)
	add_library(stb::${STB_NAME} ALIAS ${STB_NAME})
	set_target_properties(${STB_NAME} PROPERTIES OUTPUT_NAME stb-${STB_NAME})
	target_compile_definitions(${STB_NAME} PRIVATE ${STB_DEFINE})
	install(TARGETS ${STB_NAME} EXPORT stbTargets ARCHIVE DESTINATION lib)
	target_include_directories(${STB_NAME} INTERFACE
		$<INSTALL_INTERFACE:${STB_INSTALL_INCLUDE_DIR}>
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
	)
endforeach()

# Vorbis is special
file(READ stb/stb_vorbis.c STB_VORBIS_CONTENT)
configure_file(cmake/stb_vorbis.h.in include/stb/stb_vorbis.h)
add_library(vorbis stb/stb_vorbis.c)
add_library(stb::vorbis ALIAS vorbis)
set_target_properties(vorbis PROPERTIES OUTPUT_NAME stb-vorbis)
install(TARGETS vorbis EXPORT stbTargets ARCHIVE DESTINATION lib)
target_include_directories(vorbis INTERFACE
	$<INSTALL_INTERFACE:${STB_INSTALL_INCLUDE_DIR}>
	$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include>
)

if (STB_INSTALL)
	set(STB_CONFIG_PATH "lib/cmake/stb")
	
	include(CMakePackageConfigHelpers)
	
	# build tree package config
	configure_file(
		cmake/stbConfig.cmake.in
		stbConfig.cmake
		@ONLY
	)
	
	# install tree package config
	configure_package_config_file(
		cmake/stbConfig.cmake.in
		${STB_CONFIG_PATH}/stbConfig.cmake
		INSTALL_DESTINATION ${STB_CONFIG_PATH}
		NO_CHECK_REQUIRED_COMPONENTS_MACRO
	)
	
	install(DIRECTORY stb DESTINATION ${STB_INSTALL_INCLUDE_DIR} FILES_MATCHING PATTERN "*.h")
	install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/stb DESTINATION ${STB_INSTALL_INCLUDE_DIR} FILES_MATCHING PATTERN "*.h")
	
	install(
		FILES 
			"${CMAKE_CURRENT_BINARY_DIR}/${STB_CONFIG_PATH}/stbConfig.cmake"
		DESTINATION ${STB_CONFIG_PATH}
	)
	
	install(
		EXPORT stbTargets FILE stbTargets.cmake
		NAMESPACE stb::
		DESTINATION ${STB_CONFIG_PATH}
	)
	
	export(
		EXPORT stbTargets
		NAMESPACE stb::
		FILE "${CMAKE_CURRENT_BINARY_DIR}/stbTargets.cmake"
	)
endif()

if (STB_EXPORT)
	export(PACKAGE stb)
endif()
