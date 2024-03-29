############################################################################
# CMakeLists.txt
# Copyright (C) 2016-2018  Belledonne Communications, Grenoble France
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

if(NOT CPACK_PACKAGE_NAME)
	string(TOLOWER "${PROJECT_NAME}" LOWERCASE_PROJECT_NAME)
	set(CPACK_PACKAGE_NAME "${LOWERCASE_PROJECT_NAME}")
endif()

set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/../COPYING")

set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})

set(CPACK_PACKAGE_FILE_NAME ${CPACK_PACKAGE_NAME}-${PROJECT_VERSION})

set(CPACK_SOURCE_GENERATOR "TGZ")
set(CPACK_SOURCE_IGNORE_FILES
	"^${CMAKE_BINARY_DIR}"
	"^${PROJECT_SOURCE_DIR}/.git*"
	"^${PROJECT_SOURCE_DIR}/webrtc/build"
	"^${PROJECT_SOURCE_DIR}/webrtc/build_overrides"
	"^${PROJECT_SOURCE_DIR}/webrtc/chromium"
	"^${PROJECT_SOURCE_DIR}/webrtc/data"
	"^${PROJECT_SOURCE_DIR}/webrtc/infra"
	"^${PROJECT_SOURCE_DIR}/webrtc/resources"
	"^${PROJECT_SOURCE_DIR}/webrtc/talk"
	"^${PROJECT_SOURCE_DIR}/webrtc/third_party"
	"^${PROJECT_SOURCE_DIR}/webrtc/tools"
	"^${PROJECT_SOURCE_DIR}/webrtc/webrtc/audio"
	"^${PROJECT_SOURCE_DIR}/webrtc/webrtc/build"
	"^${PROJECT_SOURCE_DIR}/webrtc/webrtc/call"
	"^${PROJECT_SOURCE_DIR}/webrtc/webrtc/common_video"
	"^${PROJECT_SOURCE_DIR}/webrtc/webrtc/examples"
	"^${PROJECT_SOURCE_DIR}/webrtc/webrtc/libjingle"
	"^${PROJECT_SOURCE_DIR}/webrtc/webrtc/modules/audio_conference_mixer"
	"^${PROJECT_SOURCE_DIR}/webrtc/webrtc/modules/audio_device"
	"^${PROJECT_SOURCE_DIR}/webrtc/webrtc/modules/bitrate_controller"
	"^${PROJECT_SOURCE_DIR}/webrtc/webrtc/modules/desktop_capture"
	"^${PROJECT_SOURCE_DIR}/webrtc/webrtc/modules/media_file"
	"^${PROJECT_SOURCE_DIR}/webrtc/webrtc/modules/pacing"
	"^${PROJECT_SOURCE_DIR}/webrtc/webrtc/modules/remote_bitrate_estimator"
	"^${PROJECT_SOURCE_DIR}/webrtc/webrtc/modules/rtp_rtcp"
	"^${PROJECT_SOURCE_DIR}/webrtc/webrtc/modules/utility"
	"^${PROJECT_SOURCE_DIR}/webrtc/webrtc/modules/video*"
	"^${PROJECT_SOURCE_DIR}/webrtc/webrtc/p2p"
	"^${PROJECT_SOURCE_DIR}/webrtc/webrtc/sound"
	"^${PROJECT_SOURCE_DIR}/webrtc/webrtc/test"
	"^${PROJECT_SOURCE_DIR}/webrtc/webrtc/tools"
	"^${PROJECT_SOURCE_DIR}/webrtc/webrtc/video*"
	"^${PROJECT_SOURCE_DIR}/webrtc/webrtc/voice_engine"
)

bc_project_build_version(${PROJECT_VERSION} PROJECT_VERSION_BUILD)
if(PROJECT_VERSION_BUILD)
	set(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_FILE_NAME}-${PROJECT_VERSION_BUILD}")
endif()

message("-- Package file name is ${CPACK_PACKAGE_FILE_NAME}" )

set(CPACK_SOURCE_PACKAGE_FILE_NAME ${CPACK_PACKAGE_FILE_NAME})

bc_generate_rpm_specfile("rpm/mswebrtc.spec.cmake" "${PROJECT_SOURCE_DIR}/mswebrtc.spec")

include(CPack)

