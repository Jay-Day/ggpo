include(CMakeSources.cmake)

add_library(GGPO
	${GGPO_LIB_SRC}
)

add_common_flags(GGPO)

target_include_directories(GGPO PUBLIC
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/lib/ggpo>
)

if(WIN32)
	# Define _WINDOWS for all Windows builds to correctly include platform headers
	add_definitions(-D_WINDOWS)
	
	if(MSVC)
		target_compile_options(GGPO PRIVATE "/W4" "/WX")
	else()
		# Use GCC-style flags for MinGW
		target_compile_options(GGPO PRIVATE "-W" "-Wall")
	endif()
	if(BUILD_SHARED_LIBS)
		# Link to Multimedia API and Winsocks during a shared build.
		if(MSVC)
			target_link_libraries(GGPO PUBLIC winmm.lib ws2_32.lib)
		else()
			# For MinGW, use the correct library format
			target_link_libraries(GGPO PUBLIC winmm ws2_32)
		endif()
		add_definitions(-DGGPO_SHARED_LIB)
		add_definitions(-DGGPO_SDK_EXPORT)
	else()
		# Even for static builds, ensure we link against required Windows libraries
		if(NOT MSVC)
			target_link_libraries(GGPO PUBLIC winmm ws2_32)
		endif()
	endif()
endif()

set_target_properties(GGPO PROPERTIES VERSION ${PROJECT_VERSION})

# Install
install(
	DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include
	DESTINATION ./
	FILES_MATCHING
	PATTERN "*.h"
	PATTERN "*.hpp"
)

install(
	TARGETS GGPO
	EXPORT ggpo
	RUNTIME DESTINATION bin
	LIBRARY DESTINATION lib
	ARCHIVE DESTINATION lib
)

if(MSVC)
	if(BUILD_SHARED_LIBS)
		install(
			FILES $<TARGET_PDB_FILE:GGPO>
			DESTINATION lib
			OPTIONAL
		)
	else()
		## NOTE: This is rather hacky, since CMake doesn't allow to install the PDB files for
		##	     static libraries, so we get the debug PDB file and copy it there.
		install(
			FILES "${CMAKE_ARCHIVE_OUTPUT_DIRECTORY_DEBUG}/GGPO.pdb"
			DESTINATION lib
			OPTIONAL
		)
	endif()
else()
## TODO place the symbols install when GGPO became cross-platform
endif()