############################################################################
# CMakeLists.txt
# Copyright (C) 2014-2023  Belledonne Communications, Grenoble France
#
############################################################################
#
# This program is free software; you can redistribute it and/or
# modify it under the terms of the GNU General Public License
# as published by the Free Software Foundation; either version 2
# of the License, or (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA  02110-1301, USA.
#
############################################################################

cmake_minimum_required(VERSION 3.22)

project(MSWebRTC VERSION 1.1.1 LANGUAGES C CXX)

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib/mediastreamer2/plugins")

set(PACKAGE "${PROJECT_NAME}")
set(PACKAGE_NAME "${PROJECT_NAME}")
set(PACKAGE_VERSION "${PROJECT_VERSION}")
set(PACKAGE_STRING "${PACKAGE_NAME} ${PACKAGE_VERSION}")
set(PACKAGE_BUGREPORT "support@belledonne-communications.com")
set(PACKAGE_TARNAME "mswebrtc")
set(PACKAGE_URL "")
set(VERSION "${PACKAGE_VERSION}")


option(ENABLE_AEC "Enable the WebRTC echo canceller support." YES)
option(ENABLE_AECM "Enable the WebRTC echo canceller for mobile support." YES)
option(ENABLE_ISAC "Enable the ISAC audio codec support." YES)
option(ENABLE_ILBC "Build iLBC codec filter" YES)
option(ENABLE_VAD "Build the WebRTC VAD support" YES)
set(ISAC_FLAVOUR "fix" CACHE STRING "ISAC audio codec flavour.")
set_property(CACHE ISAC_FLAVOUR PROPERTY STRINGS "fix" "main")


if(NOT ENABLE_AEC AND NOT ENABLE_AECM AND NOT ENABLE_ISAC AND NOT ENABLE_ILBC)
	message(FATAL_ERROR "You need to enable at least one feature of WebRTC.")
endif()


set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_EXTENSIONS NO)

include(GNUInstallDirs)

if(NOT CMAKE_INSTALL_RPATH AND CMAKE_INSTALL_PREFIX)
	set(CMAKE_INSTALL_RPATH ${CMAKE_INSTALL_FULL_LIBDIR})
	message(STATUS "Setting install rpath to ${CMAKE_INSTALL_RPATH}")
endif()

find_package(BCToolbox 5.3.0 REQUIRED)
find_package(Mediastreamer2 5.3.0 REQUIRED)

find_library(LIBM NAMES m)

if(ENABLE_AEC)
	set(BUILD_AEC 1)
endif()
if(ENABLE_AECM)
	set(BUILD_AECM 1)
endif()
if(ENABLE_ISAC)
	set(BUILD_ISAC 1)
endif()
if(ENABLE_ILBC)
	set(BUILD_ILBC 1)
endif()
if(ENABLE_VAD)
	set(BUILD_VAD 1)
endif()
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/config.h.cmake" "${CMAKE_CURRENT_BINARY_DIR}/config.h")

set(WEBRTC_ROOT_DIR "${CMAKE_CURRENT_SOURCE_DIR}/webrtc")
set(WEBRTC_SRC_DIR "${WEBRTC_ROOT_DIR}/webrtc")
include_directories(
	${CMAKE_CURRENT_BINARY_DIR}
	${WEBRTC_ROOT_DIR}
	${WEBRTC_SRC_DIR}/common_audio/signal_processing/include
)

add_definitions(-DHAVE_CONFIG_H)
if(WIN32)
	add_definitions(-DWEBRTC_WIN)
endif()
if(ANDROID)
	add_definitions("-DANDROID")
endif()

if(CMAKE_SYSTEM_NAME STREQUAL "WindowsPhone" AND CMAKE_GENERATOR_PLATFORM STREQUAL "ARM")
	add_definitions(
		-D__ARMEL__
		-DWEBRTC_ARCH_ARM
	)
elseif(CMAKE_SYSTEM_NAME STREQUAL "WindowsStore" AND CMAKE_GENERATOR MATCHES "^.* ARM$")
	add_definitions(
		-D__ARMEL__
		-DWEBRTC_ARCH_ARM
	)
endif()
if(UNIX)
	add_definitions(-DWEBRTC_POSIX)
	if(APPLE)
		add_definitions(-DWEBRTC_MAC -DWEBRTC_THREAD_RR -DWEBRTC_CLOCK_TYPE_REALTIME)
	else()
		add_definitions(-DWEBRTC_LINUX -DWEBRTC_THREAD_RR)
	endif()
endif()

string(TOLOWER "${CMAKE_SYSTEM_PROCESSOR}" SYSTEM_PROCESSOR)
string(REGEX MATCH "^(arm*|aarch64)" FIXED_POINT_PROCESSOR "${SYSTEM_PROCESSOR}")
if(FIXED_POINT_PROCESSOR)
	add_definitions(-D__ARMEL__)
endif()

set(AEC_CXX11_FLAGS "-std=c++11")

set(WEBRTC_SIGNAL_PROCESSING_DIR ${WEBRTC_SRC_DIR}/common_audio/signal_processing)
set(WEBRTC_SIGNAL_PROCESSING_SRCS
	${WEBRTC_SIGNAL_PROCESSING_DIR}/spl_sqrt.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/real_fft.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/filter_ar_fast_q12.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/complex_bit_reverse.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/spl_init.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/splitting_filter.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/spl_sqrt_floor.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/levinson_durbin.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/filter_ar.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/get_scaling_square.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/resample_fractional.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/resample.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/downsample_fast.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/filter_ma_fast_q12.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/auto_corr_to_refl_coef.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/auto_correlation.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/refl_coef_to_lpc.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/dot_product_with_scale.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/copy_set_operations.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/cross_correlation.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/sqrt_of_one_minus_x_squared.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/ilbc_specific_functions.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/resample_by_2.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/complex_fft.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/resample_48khz.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/energy.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/get_hanning_window.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/randomization_functions.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/vector_scaling_operations.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/resample_by_2_internal.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/lpc_to_refl_coef.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/min_max_operations.c
	${WEBRTC_SIGNAL_PROCESSING_DIR}/division_operations.c
)
set(WEBRTC_COMMON_DIR "${WEBRTC_SRC_DIR}/common_audio")
set(WEBRTC_COMMON_SRCS
	${WEBRTC_COMMON_DIR}/fft4g.c
	${WEBRTC_COMMON_DIR}/ring_buffer.c
)
include_directories(${WEBRTC_COMMOND_DIR}/include)
if(ENABLE_AEC)
	set(AEC_SRC_DIR "${WEBRTC_SRC_DIR}/modules/audio_processing/aec")
	set(AEC_SRCS
		aec.c
		aec_splitting_filter.cc
		${AEC_SRC_DIR}/aec_core.c
		${AEC_SRC_DIR}/aec_rdft.c
		${AEC_SRC_DIR}/aec_resampler.c
		${AEC_SRC_DIR}/echo_cancellation.c
		${WEBRTC_SRC_DIR}/base/checks.cc
		${WEBRTC_SRC_DIR}/common_audio/audio_util.cc
		${WEBRTC_SRC_DIR}/common_audio/sparse_fir_filter.cc
		${WEBRTC_SRC_DIR}/system_wrappers/source/cpu_features.cc
		${WEBRTC_SRC_DIR}/modules/audio_processing/three_band_filter_bank.cc
		${WEBRTC_SRC_DIR}/modules/audio_processing/utility/delay_estimator.c
		${WEBRTC_SRC_DIR}/modules/audio_processing/utility/delay_estimator_wrapper.c
	)
	set(AEC_CXX11_SRCS
		aec_splitting_filter.cc
		${WEBRTC_SRC_DIR}/common_audio/audio_util.cc
		${WEBRTC_SRC_DIR}/common_audio/sparse_fir_filter.cc
		${WEBRTC_SRC_DIR}/modules/audio_processing/three_band_filter_bank.cc
	)
	bc_apply_compile_flags(AEC_CXX11_SRCS AEC_CXX11_FLAGS)
	if(SYSTEM_PROCESSOR MATCHES "(^aarch64.*|arm64)")
		add_definitions(
			-DWEBRTC_ARCH_ARM64=1
		)
	endif()

	if(SYSTEM_PROCESSOR MATCHES "^arm.*" OR SYSTEM_PROCESSOR MATCHES "^aarch64.*")
		list(APPEND AEC_SRCS
			${AEC_SRC_DIR}/aec_core_neon.c
			${AEC_SRC_DIR}/aec_rdft_neon.c
		)
	elseif(SYSTEM_PROCESSOR MATCHES "^i.?86.*" OR SYSTEM_PROCESSOR MATCHES "^x86.*" OR SYSTEM_PROCESSOR MATCHES "^x86_64.*" OR SYSTEM_PROCESSOR MATCHES "^amd64.*" OR SYSTEM_PROCESSOR MATCHES "^x64.*")
		list(APPEND AEC_SRCS
			${AEC_SRC_DIR}/aec_core_sse2.c
			${AEC_SRC_DIR}/aec_rdft_sse2.c
		)
	endif()
	include_directories(
		${AEC_SRC_DIR}/include
		${WEBRTC_SRC_DIR}/common_audio/include
		${WEBRTC_SRC_DIR}/modules/audio_processing
	)
endif()
if(ENABLE_AECM)
	set(AECM_SRC_DIR "${WEBRTC_SRC_DIR}/modules/audio_processing/aecm")
	set(AECM_SRCS
		aec.c
		${AECM_SRC_DIR}/echo_control_mobile.c
		${AECM_SRC_DIR}/aecm_core.c
		${AECM_SRC_DIR}/aecm_core_c.c
		${AECM_SRC_DIR}/../utility/
		${AECM_SRC_DIR}/../utility/delay_estimator.c
		${AECM_SRC_DIR}/../utility/delay_estimator_wrapper.c
	)
	include_directories(
		${AECM_SRC_DIR}/include
		${WEBRTC_SRC_DIR}/modules/audio_processing/utility
		${WEBRTC_SRC_DIR}/system_wrappers/interface
	)
endif()
if(ENABLE_ISAC)
	set(ISAC_SRC_DIR "${WEBRTC_SRC_DIR}/modules/audio_coding/codecs/isac/${ISAC_FLAVOUR}")
	set(ISAC_SRCS
		isac_dec.c
		isac_enc.c
		${ISAC_SRC_DIR}/source/fft.c
		${ISAC_SRC_DIR}/source/arith_routines.c
		${ISAC_SRC_DIR}/source/arith_routines_hist.c
		${ISAC_SRC_DIR}/source/arith_routines_logist.c
		${ISAC_SRC_DIR}/source/bandwidth_estimator.c
		${ISAC_SRC_DIR}/source/decode.c
		${ISAC_SRC_DIR}/source/decode_bwe.c
		${ISAC_SRC_DIR}/source/encode.c
		${ISAC_SRC_DIR}/source/entropy_coding.c
		${ISAC_SRC_DIR}/source/filterbank_tables.c
		${ISAC_SRC_DIR}/source/filterbanks.c
		${ISAC_SRC_DIR}/source/lattice.c
		${ISAC_SRC_DIR}/source/lpc_tables.c
		${ISAC_SRC_DIR}/source/pitch_estimator.c
		${ISAC_SRC_DIR}/source/pitch_estimator_c.c
		${ISAC_SRC_DIR}/source/pitch_filter.c
		${ISAC_SRC_DIR}/source/pitch_gain_tables.c
		${ISAC_SRC_DIR}/source/pitch_lag_tables.c
		${ISAC_SRC_DIR}/source/spectrum_ar_model_tables.c
		${ISAC_SRC_DIR}/source/transform.c
		${ISAC_SRC_DIR}/source/transform_tables.c
	)
	if("${ISAC_FLAVOUR}" STREQUAL "fix")
		list(APPEND ISAC_SRCS
			${ISAC_SRC_DIR}/source/decode_plc.c
			${ISAC_SRC_DIR}/source/filters.c
			${ISAC_SRC_DIR}/source/initialize.c
			${ISAC_SRC_DIR}/source/isacfix.c
			${ISAC_SRC_DIR}/source/lattice_c.c
			${ISAC_SRC_DIR}/source/pitch_filter_c.c
			${ISAC_SRC_DIR}/source/lpc_masking_model.c
		)
	else()
		list(APPEND ISAC_SRCS
			${ISAC_SRC_DIR}/source/initialize.c
			${ISAC_SRC_DIR}/source/crc.c
			${ISAC_SRC_DIR}/source/encode_lpc_swb.c
			${ISAC_SRC_DIR}/source/filter_functions.c
			${ISAC_SRC_DIR}/source/isac.c
			${ISAC_SRC_DIR}/source/lpc_analysis.c
			${ISAC_SRC_DIR}/source/lpc_gain_swb_tables.c
			${ISAC_SRC_DIR}/source/lpc_shape_swb12_tables.c
			${ISAC_SRC_DIR}/source/lpc_shape_swb16_tables.c
			${ISAC_SRC_DIR}/util/utility.c
		)
	endif()
	include_directories(
		${ISAC_SRC_DIR}/source
		${ISAC_SRC_DIR}/util
		${ISAC_SRC_DIR}/include
	)
endif()
if(ENABLE_ILBC)
	set(ILBC_SRC_DIR ${WEBRTC_SRC_DIR}/modules/audio_coding/codecs/ilbc)
	set(ILBC_SRCS
		ilbc.c
		${ILBC_SRC_DIR}/abs_quant.c
		${ILBC_SRC_DIR}/abs_quant_loop.c
		${ILBC_SRC_DIR}/augmented_cb_corr.c
		${ILBC_SRC_DIR}/bw_expand.c
		${ILBC_SRC_DIR}/cb_construct.c
		${ILBC_SRC_DIR}/cb_mem_energy_augmentation.c
		${ILBC_SRC_DIR}/cb_mem_energy.c
		${ILBC_SRC_DIR}/cb_mem_energy_calc.c
		${ILBC_SRC_DIR}/cb_search.c
		${ILBC_SRC_DIR}/cb_search_core.c
		${ILBC_SRC_DIR}/cb_update_best_index.c
		${ILBC_SRC_DIR}/chebyshev.c
		${ILBC_SRC_DIR}/comp_corr.c
		${ILBC_SRC_DIR}/constants.c
		${ILBC_SRC_DIR}/create_augmented_vec.c
		${ILBC_SRC_DIR}/decode.c
		${ILBC_SRC_DIR}/decode_residual.c
		${ILBC_SRC_DIR}/decoder_interpolate_lsf.c
		${ILBC_SRC_DIR}/do_plc.c
		${ILBC_SRC_DIR}/encode.c
		${ILBC_SRC_DIR}/energy_inverse.c
		${ILBC_SRC_DIR}/enhancer.c
		${ILBC_SRC_DIR}/enhancer_interface.c
		${ILBC_SRC_DIR}/enh_upsample.c
		${ILBC_SRC_DIR}/filtered_cb_vecs.c
		${ILBC_SRC_DIR}/frame_classify.c
		${ILBC_SRC_DIR}/gain_dequant.c
		${ILBC_SRC_DIR}/gain_quant.c
		${ILBC_SRC_DIR}/get_cd_vec.c
		${ILBC_SRC_DIR}/get_lsp_poly.c
		${ILBC_SRC_DIR}/get_sync_seq.c
		${ILBC_SRC_DIR}/hp_input.c
		${ILBC_SRC_DIR}/hp_output.c
		${ILBC_SRC_DIR}/ilbc.c
		${ILBC_SRC_DIR}/index_conv_dec.c
		${ILBC_SRC_DIR}/index_conv_enc.c
		${ILBC_SRC_DIR}/init_decode.c
		${ILBC_SRC_DIR}/init_encode.c
		${ILBC_SRC_DIR}/interpolate.c
		${ILBC_SRC_DIR}/interpolate_samples.c
		${ILBC_SRC_DIR}/lpc_encode.c
		${ILBC_SRC_DIR}/lsf_check.c
		${ILBC_SRC_DIR}/lsf_interpolate_to_poly_dec.c
		${ILBC_SRC_DIR}/lsf_interpolate_to_poly_enc.c
		${ILBC_SRC_DIR}/lsf_to_lsp.c
		${ILBC_SRC_DIR}/lsf_to_poly.c
		${ILBC_SRC_DIR}/lsp_to_lsf.c
		${ILBC_SRC_DIR}/my_corr.c
		${ILBC_SRC_DIR}/nearest_neighbor.c
		${ILBC_SRC_DIR}/pack_bits.c
		${ILBC_SRC_DIR}/poly_to_lsf.c
		${ILBC_SRC_DIR}/poly_to_lsp.c
		${ILBC_SRC_DIR}/refiner.c
		${ILBC_SRC_DIR}/simple_interpolate_lsf.c
		${ILBC_SRC_DIR}/simple_lpc_analysis.c
		${ILBC_SRC_DIR}/simple_lsf_dequant.c
		${ILBC_SRC_DIR}/simple_lsf_quant.c
		${ILBC_SRC_DIR}/smooth.c
		${ILBC_SRC_DIR}/smooth_out_data.c
		${ILBC_SRC_DIR}/sort_sq.c
		${ILBC_SRC_DIR}/split_vq.c
		${ILBC_SRC_DIR}/state_construct.c
		${ILBC_SRC_DIR}/state_search.c
		${ILBC_SRC_DIR}/swap_bytes.c
		${ILBC_SRC_DIR}/unpack_bits.c
		${ILBC_SRC_DIR}/vq3.c
		${ILBC_SRC_DIR}/vq4.c
		${ILBC_SRC_DIR}/window32_w32.c
		${ILBC_SRC_DIR}/xcorr_coef.c
	)
	include_directories(${ILBC_SRC_DIR}/include)
endif()

if(ENABLE_VAD)
	set(VAD_SRC_DIR ${WEBRTC_SRC_DIR}/common_audio/vad)
	set(VAD_SRCS
		vad.cc
		${VAD_SRC_DIR}/vad.cc
		${VAD_SRC_DIR}/vad_core.c
		${VAD_SRC_DIR}/vad_filterbank.c
		${VAD_SRC_DIR}/vad_gmm.c
		${VAD_SRC_DIR}/vad_sp.c
		${VAD_SRC_DIR}/webrtc_vad.c
	)
	include_directories(${VAD_SRC_DIR}/include)
endif()

set(SOURCE_FILES
	mswebrtc.c
	${WEBRTC_COMMON_SRCS}
	${WEBRTC_SIGNAL_PROCESSING_SRCS}
)
if(ENABLE_AEC)
	list(APPEND SOURCE_FILES ${AEC_SRCS})
endif()
if(ENABLE_AECM)
	list(APPEND SOURCE_FILES ${AECM_SRCS})
endif()
if(ENABLE_ISAC)
	list(APPEND SOURCE_FILES ${ISAC_SRCS})
endif()
if(ENABLE_ILBC)
	list(APPEND SOURCE_FILES ${ILBC_SRCS})
endif()
if(ENABLE_VAD)
	list(APPEND SOURCE_FILES ${VAD_SRCS})
endif()

set(LIBS )

if(LIBM)
	list(APPEND LIBS ${LIBM})
endif()

if(BUILD_SHARED_LIBS)
	if(IOS)
		add_library(mswebrtc SHARED ${SOURCE_FILES})
	else()
		add_library(mswebrtc MODULE ${SOURCE_FILES})
	endif()
else()
	add_library(mswebrtc STATIC ${SOURCE_FILES})
endif()
target_link_libraries(mswebrtc PRIVATE ${Mediastreamer2_TARGET} ${LIBS})
set_target_properties(mswebrtc PROPERTIES LINKER_LANGUAGE CXX)
if(BUILD_SHARED_LIBS)
	if(APPLE)
		if(IOS)
			set_target_properties(mswebrtc 	PROPERTIES
							FRAMEWORK TRUE
							MACOSX_FRAMEWORK_IDENTIFIER org.linphone.mswebrtc
							MACOSX_FRAMEWORK_INFO_PLIST "${PROJECT_SOURCE_DIR}/build/osx/Info.plist.in"
			)
		endif()
	endif()
	if(MSVC)
		install(FILES $<TARGET_PDB_FILE:mswebrtc>
			DESTINATION ${CMAKE_INSTALL_BINDIR}
			PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
			CONFIGURATIONS Debug RelWithDebInfo
		)
		if(NOT IOS)
			set_target_properties(mswebrtc PROPERTIES PREFIX "lib")
		endif()
	endif()
endif()

install(TARGETS mswebrtc
		RUNTIME DESTINATION ${Mediastreamer2_PLUGINS_DIR}
		LIBRARY DESTINATION ${Mediastreamer2_PLUGINS_DIR}
		ARCHIVE DESTINATION ${Mediastreamer2_PLUGINS_DIR}
		FRAMEWORK DESTINATION Frameworks
		PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
)


add_subdirectory(build)

