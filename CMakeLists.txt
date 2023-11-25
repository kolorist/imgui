cmake_policy (VERSION 3.20min)

include ("project_configs.cmake")

project (${PROJECT_NAME})

include_directories ("${PROJECT_SOURCE_DIR}/src")

if (${MSVC_PROJECT})
	file (GLOB_RECURSE file_list
		LIST_DIRECTORIES false
		"${PROJECT_SOURCE_DIR}/include/*.h"
		"${PROJECT_SOURCE_DIR}/include/*.inl"
		"${PROJECT_SOURCE_DIR}/src/*.cpp")
else ()
	set (file_list
		"${PROJECT_SOURCE_DIR}/src/imgui.cpp"
		"${PROJECT_SOURCE_DIR}/src/imgui_demo.cpp"
		"${PROJECT_SOURCE_DIR}/src/imgui_widgets.cpp"
		"${PROJECT_SOURCE_DIR}/src/imgui_draw.cpp")
endif (${MSVC_PROJECT})

add_definitions(
	-D_CRT_SECURE_NO_WARNINGS)

if (${ANDROID_BUILD})
	message(STATUS ${PROJECT_NAME} " will be built using Android configs")
	add_definitions (
		-DPLATFORM_POSIX)

	# platform abi
	if (${ANDROID_ABI} STREQUAL "arm64-v8a")
		message(STATUS ${PROJECT_NAME} " Android ABI: arm64")
		add_definitions (
			-DPOSIX64)
	else ()
		message(STATUS ${PROJECT_NAME} " Android ABI: arm")
		add_definitions (
			-DPOSIX32)
	endif (${ANDROID_ABI} STREQUAL "arm64-v8a")
else ()
	message(STATUS ${PROJECT_NAME} " will be built using Windows configs")
	add_definitions (
		-DPLATFORM_WINDOWS)
endif (${ANDROID_BUILD})

add_library (${PROJECT_NAME} ${file_list})

target_include_directories (${PROJECT_NAME} PUBLIC
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>)

if (${MSVC_PROJECT})
	# organize filters
	foreach(_source IN ITEMS ${file_list})
		get_filename_component(_source_path "${_source}" PATH)
		file(RELATIVE_PATH _source_path_rel "${PROJECT_SOURCE_DIR}" "${_source_path}")
		string(REPLACE "/" "\\" _group_path "${_source_path_rel}")
		source_group("${_group_path}" FILES "${_source}")
	endforeach()
endif (${MSVC_PROJECT})
