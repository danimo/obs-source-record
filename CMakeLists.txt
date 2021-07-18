cmake_minimum_required(VERSION 3.18)
project(source-record VERSION 0.2.2)
set(PROJECT_FULL_NAME "Source Record")


set(CMAKE_INCLUDE_CURRENT_DIR ON)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/version.h.in ${CMAKE_CURRENT_BINARY_DIR}/version.h)

find_package (LibObs REQUIRED)

if(MSVC)
	set(source-record_PLATFORM_DEPS
		w32-pthreads)
endif()

set(source-record_HEADERS
	source-record.h)
set(source-record_SOURCES
	source-record.c)

if(WIN32)
	get_filename_component(ISS_FILES_DIR "${CMAKE_BINARY_DIR}\\..\\package" ABSOLUTE)
	file(TO_NATIVE_PATH "${ISS_FILES_DIR}" ISS_FILES_DIR)
	get_filename_component(ISS_PACKAGE_DIR "${CMAKE_PACKAGE_PREFIX}\\.." ABSOLUTE)
	file(TO_NATIVE_PATH "${ISS_PACKAGE_DIR}" ISS_PACKAGE_DIR)
	get_filename_component(ISS_SOURCE_DIR "${PROJECT_SOURCE_DIR}" ABSOLUTE)
	file(TO_NATIVE_PATH "${ISS_SOURCE_DIR}" ISS_SOURCE_DIR)
	configure_file("installer.iss.in"
			"${PROJECT_BINARY_DIR}/installer.iss"
		)
	configure_file(resource.rc.in source-record.rc)
	list(APPEND source-record_SOURCES
		source-record.rc)
endif()

#include_directories("${LIBOBS_INCLUDE_DIR}/obs-frontend-api")

add_library(source-record MODULE
	${source-record_HEADERS}
	${source-record_SOURCES})
target_include_directories(source-record PUBLIC ${source-record_INCLUDES})
target_link_libraries(source-record
	obs-frontend-api
	libobs
	${source-record_PLATFORM_DEPS})

if(UNIX AND NOT APPLE)
	include(GNUInstallDirs)

	file(GLOB locale_files data/locale/*.ini)

        set(CMAKE_INSTALL_DEFAULT_DIRECTORY_PERMISSIONS
		OWNER_READ OWNER_WRITE OWNER_EXECUTE
		GROUP_READ GROUP_WRITE GROUP_EXECUTE
		WORLD_READ WORLD_EXECUTE
	)

	install(TARGETS source-record LIBRARY
		DESTINATION "${LIBOBS_PLUGIN_DESTINATION}"
		PERMISSIONS OWNER_READ OWNER_WRITE GROUP_READ WORLD_READ)

	install(FILES ${locale_files}
		DESTINATION "${LIBOBS_PLUGIN_DATA_DESTINATION}/obs-source-record/locale")
endif()

#set_target_properties(source-record PROPERTIES FOLDER "plugins/exeldro")
#install_obs_plugin_with_data(source-record data)
