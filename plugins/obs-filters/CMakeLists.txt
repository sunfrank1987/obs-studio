project(obs-filters)

option(DISABLE_SPEEXDSP "Disable building of the SpeexDSP-based Noise Suppression filter" OFF)
option(DISABLE_RNNOISE "Disable building of the RNNoise-based Noise Suppression filter" OFF)

if(DISABLE_SPEEXDSP AND DISABLE_RNNOISE)
	message(STATUS "SpeexDSP and RNNoise support disabled")
	set(NOISEREDUCTION_ENABLED FALSE)
else()
	if(DISABLE_SPEEXDSP)
		message(STATUS "SpeexDSP support disabled")
		set(LIBSPEEXDSP_FOUND FALSE)
	else()
		find_package(Libspeexdsp QUIET)
		if(NOT LIBSPEEXDSP_FOUND)
			message(STATUS "SpeexDSP support not found")
			set(LIBSPEEXDSP_FOUND FALSE)
		else()
			message(STATUS "SpeexDSP supported")
			add_definitions(-DLIBSPEEXDSP_ENABLED)
		endif()
	endif()

	if(DISABLE_RNNOISE)
		message(STATUS "RNNoise support disabled")
		set(LIBRNNOISE_FOUND FALSE)
	else()
		find_package(Librnnoise QUIET)
		if(NOT LIBRNNOISE_FOUND)
			message(STATUS "RNNoise support not found")
			set(LIBRNNOISE_FOUND FALSE)
		else()
			message(STATUS "RNNoise supported")
			add_definitions(-DLIBRNNOISE_ENABLED)
		endif()

	endif()

	if(LIBSPEEXDSP_FOUND OR LIBRNNOISE_FOUND)
		message(STATUS "Noise Reduction effect enabled")
		set(obs-filters_NOISEREDUCTION_SOURCES
			noise-suppress-filter.c)
		set(obs-filters_NOISEREDUCTION_LIBRARIES
			${LIBSPEEXDSP_LIBRARIES} ${LIBRNNOISE_LIBRARIES})
		set(NOISEREDUCTION_ENABLED TRUE)
	else()
		set(NOISEREDUCTION_ENABLED FALSE)
	endif()
endif()

configure_file("${CMAKE_CURRENT_SOURCE_DIR}/obs-filters-config.h.in"
	"${CMAKE_BINARY_DIR}/plugins/obs-filters/config/obs-filters-config.h")

set(obs-filters_config_HEADERS
	"${CMAKE_BINARY_DIR}/plugins/obs-filters/config/obs-filters-config.h")
include_directories(${LIBSPEEXDSP_INCLUDE_DIRS} ${LIBRNNOISE_INCLUDE_DIRS}
	"${CMAKE_BINARY_DIR}/plugins/obs-filters/config")

if(MSVC)
	set(obs-filters_PLATFORM_DEPS
		w32-pthreads)
endif()

set(obs-filters_SOURCES
	obs-filters.c
	color-correction-filter.c
	async-delay-filter.c
	gpu-delay.c
	crop-filter.c
	scale-filter.c
	scroll-filter.c
	chroma-key-filter.c
	color-key-filter.c
	color-grade-filter.c
	sharpness-filter.c
	gain-filter.c
	noise-gate-filter.c
	mask-filter.c
	invert-audio-polarity.c
	compressor-filter.c
	limiter-filter.c
	expander-filter.c
	luma-key-filter.c)

if(WIN32)
	set(MODULE_DESCRIPTION "OBS A/V Filters")
	configure_file(${CMAKE_SOURCE_DIR}/cmake/winrc/obs-module.rc.in obs-filters.rc)
	list(APPEND obs-filters_SOURCES
		obs-filters.rc)
endif()

add_library(obs-filters MODULE
	${obs-filters_SOURCES}
	${obs-filters_config_HEADERS}
	${obs-filters_NOISEREDUCTION_SOURCES})
target_link_libraries(obs-filters
	libobs
	${obs-filters_PLATFORM_DEPS}
	${obs-filters_NOISEREDUCTION_LIBRARIES})
set_target_properties(obs-filters PROPERTIES FOLDER "plugins")

install_obs_plugin_with_data(obs-filters data)
