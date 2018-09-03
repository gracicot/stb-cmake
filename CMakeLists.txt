cmake_minimum_required(VERSION 3.4)
project(stb C)

option(STB_EXPORT "Export stb source tree as package" true)
option(STB_INSTALL "Generate stb installation target" true)

set(STB_INSTALL_INCLUDE_DIR "include")

configure_file(stb/stb_image.h stb_image.c COPYONLY)
add_library(image ${CMAKE_CURRENT_BINARY_DIR}/stb_image.c)
add_library(stb::image ALIAS image)
set_target_properties(image PROPERTIES OUTPUT_NAME stb_image)
target_compile_definitions(image PRIVATE STB_IMAGE_IMPLEMENTATION)
target_include_directories(image INTERFACE
	$<INSTALL_INTERFACE:${STB_INSTALL_INCLUDE_DIR}>
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
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
	install(TARGETS image EXPORT stbTargets ARCHIVE DESTINATION lib)
	
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
endif()

if (STB_EXPORT)
	export(
		EXPORT stbTargets
		NAMESPACE stb::
		FILE "${CMAKE_CURRENT_BINARY_DIR}/stbTargets.cmake"
	)
	
	export(PACKAGE stb)
endif()
