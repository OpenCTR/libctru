#####################################################################
#  This software is provided 'as-is', without any express or implied
#  warranty.  In no event will the authors be held liable for any
#  damages arising from the use of this software.
#
#  Permission is granted to anyone to use this software for any
#  purpose, including commercial applications, and to alter it and
#  redistribute it freely, subject to the following restrictions:
#
#  1. The origin of this software must not be misrepresented; you
#     must not claim that you wrote the original software. If you use
#     this software in a product, an acknowledgment in the product
#     documentation would be appreciated but is not required.
#  2. Altered source versions must be plainly marked as such, and
#     must not be misrepresented as being the original software.
#  3. This notice may not be removed or altered from any source
#     distribution.
#####################################################################

cmake_minimum_required(VERSION 3.0.0)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "Debug" CACHE INTERNAL "Values: Debug, Release, RelWithDebInfo, MinSizeRel")
endif()

if(NOT DEFINED CTR_ROOT)
  message(SEND_ERROR "CTR_ROOT is not set.")
endif()

# CMake Cross-Compiler Toolchain
set(CMAKE_TOOLCHAIN_FILE "${CMAKE_CURRENT_SOURCE_DIR}/cmake/Modules/OpenCTR-Toolchain.cmake")

# Install everything to <OPENCTR_INSTALL_DIR>/libctru
set(CMAKE_INSTALL_PREFIX "${CTR_ROOT}/libctru" CACHE INTERNAL "CMAKE_INSTALL_ROOT")

include(CMakePackageConfigHelpers)
include(CMakeProjectSettings)

project(libctr)

# Require artool for ctr_add_resources()
find_package(artool REQUIRED)

set(LIBCTRU_VERSION_MAJOR 0)
set(LIBCTRU_VERSION_MINOR 5)
set(LIBCTRU_VERSION_PATCH 0)
set(LIBCTRU_VERSION "${LIBCTRU_VERSION_MAJOR}.${LIBCTRU_VERSION_MINOR}.${LIBCTRU_VERSION_PATCH}")

set(CMAKE_INCLUDE_CURRENT_DIR TRUE)

include_directories(
	${CMAKE_CURRENT_SOURCE_DIR}/include
	${CMAKE_CURRENT_SOURCE_DIR}/source
	${CMAKE_CURRENT_SOURCE_DIR}/source/allocator
	${CMAKE_CURRENT_SOURCE_DIR}/source/gpu
	${CMAKE_CURRENT_SOURCE_DIR}/source/services
	${CMAKE_CURRENT_SOURCE_DIR}/source/services/soc
	${CMAKE_CURRENT_SOURCE_DIR}/source/util/rbtree
	${CMAKE_CURRENT_SOURCE_DIR}/source/util/utf
	${CMAKE_CURRENT_SOURCE_DIR}/source/system
)

ctr_add_resources(DEFAULT_FONT HEADER default_font_bin.h data/default_font.bin)

add_library(ctru
	source/allocator/linear.cpp
	source/allocator/mem_pool.cpp
	source/allocator/vram.cpp
	source/console.c
	source/gfx.c
	source/gpu/gpu.c
	source/gpu/gpuResetSequence.c
	source/gpu/shaderProgram.c
	source/gpu/shbin.c
	source/os.c
	source/sdmc_dev.c
	source/services/ac.c
	source/services/am.c
	source/services/apt.c
	source/services/cfgnor.c
	source/services/cfgu.c
	source/services/csnd.c
	source/services/fs.c
	source/services/gsp.c
	source/services/gx.c
	source/services/hb.c
	source/services/hid.c
	source/services/httpc.c
	source/services/ir.c
	source/services/irrst.c
	source/services/mic.c
	source/services/mvd.c
	source/services/ns.c
	source/services/pm.c
	source/services/ps.c
	source/services/ptm.c
	source/services/qtm.c
	source/services/soc/soc_accept.c
	source/services/soc/soc_bind.c
	source/services/soc/soc_closesocket.c
	source/services/soc/soc_common.c
	source/services/soc/soc_connect.c
	source/services/soc/soc_fcntl.c
	source/services/soc/soc_gethostbyname.c
	source/services/soc/soc_gethostid.c
	source/services/soc/soc_getpeername.c
	source/services/soc/soc_getsockname.c
	source/services/soc/soc_getsockopt.c
	source/services/soc/soc_herror.c
	source/services/soc/soc_hstrerror.c
	source/services/soc/soc_inet_addr.c
	source/services/soc/soc_inet_aton.c
	source/services/soc/soc_inet_ntoa.c
	source/services/soc/soc_init.c
	source/services/soc/soc_ioctl.c
	source/services/soc/soc_listen.c
	source/services/soc/soc_poll.c
	source/services/soc/soc_recv.c
	source/services/soc/soc_recvfrom.c
	source/services/soc/soc_select.c
	source/services/soc/soc_send.c
	source/services/soc/soc_sendto.c
	source/services/soc/soc_setsockopt.c
	source/services/soc/soc_shutdown.c
	source/services/soc/soc_sockatmark.c
	source/services/soc/soc_socket.c
	source/srv.c
	source/svc.s
	source/system/allocateHeaps.c
	source/system/appExit.c
	source/system/appInit.c
	source/system/ctru_exit.c
	source/system/initArgv.c
	source/system/initSystem.c
	source/system/stack_adjust.s
	source/util/rbtree/rbtree_clear.c
	source/util/rbtree/rbtree_empty.c
	source/util/rbtree/rbtree_find.c
	source/util/rbtree/rbtree_init.c
	source/util/rbtree/rbtree_insert.c
	source/util/rbtree/rbtree_iterator.c
	source/util/rbtree/rbtree_minmax.c
	source/util/rbtree/rbtree_remove.c
	source/util/rbtree/rbtree_rotate.c
	source/util/rbtree/rbtree_size.c
	source/util/utf/decode_utf16.c
	source/util/utf/decode_utf8.c
	source/util/utf/encode_utf16.c
	source/util/utf/encode_utf8.c
	source/util/utf/utf16_to_utf32.c
	source/util/utf/utf16_to_utf8.c
	source/util/utf/utf32_to_utf16.c
	source/util/utf/utf32_to_utf8.c
	source/util/utf/utf8_to_utf16.c
	source/util/utf/utf8_to_utf32.c
	${DEFAULT_FONT}
)

# Export CTR::libctru instead of CTR::ctru
set_property(TARGET ctru PROPERTY EXPORT_NAME "libctru")

if(WIN32)
	# Win32 CMake find_package() searches <prefix>/CMake/
	set(CONFIG_PREFIX "CMake")
else()
	# UNIX CMake find_package() searches <prefix>/share/cmake
	set(CONFIG_PREFIX "share/cmake")
endif()

# Install default_icon.png
install(FILES default_icon.png
	DESTINATION .
)

# Configure libctru-config.cmake
configure_package_config_file(
	${CMAKE_CURRENT_SOURCE_DIR}/libctru-config.cmake.in
	${CMAKE_CURRENT_BINARY_DIR}/libctru-config.cmake
	INSTALL_DESTINATION ${CONFIG_PREFIX}
	PATH_VARS CONFIG_PREFIX
)

# Install libctru headers
install(DIRECTORY include/
	DESTINATION include
	USE_SOURCE_PERMISSIONS
	PATTERN .* EXCLUDE
)

# Install CMake modules (imported from libctr)
install(DIRECTORY cmake/Modules/
	DESTINATION ${CONFIG_PREFIX}
	PATTERN .* EXCLUDE
	PATTERN CMake*.cmake EXCLUDE
)

# Install libctru.a
install(TARGETS ctru 
	EXPORT libctru
	ARCHIVE DESTINATION lib
	INCLUDES DESTINATION include
)

# Install libctru-targets.cmake
install(EXPORT libctru
	DESTINATION ${CONFIG_PREFIX}
	FILE libctru-targets.cmake
	NAMESPACE CTR::
)

# Install libctru-config.cmake
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/libctru-config.cmake 
	DESTINATION ${CONFIG_PREFIX}
)