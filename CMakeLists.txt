cmake_minimum_required(VERSION 2.8.12)

project(rtmp-nicolive)

# Find includes in corresponding build directories
set(CMAKE_INCLUDE_CURRENT_DIR ON)
# Instruct CMake to run moc automatically when needed.
set(CMAKE_AUTOMOC ON)

# check build in OBS
if(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
	set(BUILD_IN_OBS OFF)
	MESSAGE( STATUS "RTMP NicoLive build only it")
else(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
	set(BUILD_IN_OBS ON)
	MESSAGE( STATUS "RTMP NicoLive build in OBS")
	# compiling with obs-studio
	# Please take all files on "${OBS_SRC}/plugins/rtmp-nicolive"
	# and add the following line to "${OBS_SRC}/plugins/CMakeLists.txt"
	#   add_subdirectory(rtmp-nicolive)
endif(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)

# find OBS
if(NOT BUILD_IN_OBS)
	find_package(LibObs QUIET)
	if(NOT LibObs_FOUND)
		MESSAGE( STATUS "Not Found LibObs cmake module")
		set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH}
			"${CMAKE_CURRENT_SOURCE_DIR}/tools/cmake")
		find_package(OBS REQUIRED)
		set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${OBS_SRC}/cmake/Modules/")
		find_package(CXX11 REQUIRED)
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${CXX11_FLAGS}")
		if(WIN32 AND NOT MSVC)
			set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -static-libgcc -static-libstdc++")
		endif(WIN32 AND NOT MSVC)
	endif(NOT LibObs_FOUND)
endif(NOT BUILD_IN_OBS)

# find libcurl
find_package(Libcurl REQUIRED)
include_directories(${LIBCURL_INCLUDE_DIRS})
add_definitions(${LIBCURL_DEFINITIONS})
if(NOT BUILD_IN_OBS AND WIN32)
	set(LIBCURL_LIBRARIES "${OBS_APP_BIN}/libcurl.dll")
endif(NOT BUILD_IN_OBS AND WIN32)

# header and library
if(NOT BUILD_IN_OBS AND NOT LibObs_FOUND)
	# include
	include_directories(${OBS_INCLUDES_LIBOBS})
	# libobs
	add_library(libobs SHARED IMPORTED)
	set_property(TARGET libobs
		PROPERTY IMPORTED_LOCATION ${OBS_LIB_LIBOBS})
	if(WIN32)
		set_property(TARGET libobs
			PROPERTY IMPORTED_IMPLIB ${OBS_LIB_LIBOBS})
	endif(WIN32)
endif(NOT BUILD_IN_OBS AND NOT LibObs_FOUND)

set(rtmp-nicolive_SOURCES
	nico-live-api.cpp
	nico-live-timer.cpp
	nico-live-watcher.cpp
	nico-live.cpp
	nicolive-errno.c
	nicolive-operation.c
	nicolive-api.cpp
	nicolive.cpp
	rtmp-nicolive.c)

add_library(rtmp-nicolive MODULE
	pugixml/pugixml.cpp
	# sqlite/sqlite3.c
	nicookie.cpp
	${rtmp-nicolive_SOURCES})

target_link_libraries(rtmp-nicolive
	${LIBCURL_LIBRARIES}
	libobs)

if(BUILD_IN_OBS)
	install_obs_plugin_with_data(rtmp-nicolive data)
else(BUILD_IN_OBS)
	set_property(TARGET rtmp-nicolive
		PROPERTY PREFIX "")
	install(TARGETS rtmp-nicolive
		DESTINATION "${LIBOBS_PLUGIN_DESTINATION}")
	install(DIRECTORY data/
		DESTINATION "${LIBOBS_PLUGIN_DATA_DESTINATION}/rtmp-nicolive")
endif(BUILD_IN_OBS)
