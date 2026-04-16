cmake_minimum_required(VERSION 3.15)
project(bltool VERSION 1.0.0 LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Build type" FORCE)
endif()
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

# OpenSSL find
find_package(OpenSSL REQUIRED)

# SQLite find
# find_library(SQLite3_LIBRARY sqlite3)
# if(NOT SQLite3_LIBRARY)
#     message(WARNING "SQLite3 not found, search engine will be limited")
#     set(SQLite3_LIBRARY "")
# endif()

# WinFsp
if(NOT WINFSP_ROOT_DIR)
    set(WINFSP_ROOT_DIR "C:/Program Files (x86)/WinFsp" CACHE PATH "WinFsp installation directory")
endif()
set(WINFSP_INCLUDE_DIR ${WINFSP_ROOT_DIR}/inc)
set(WINFSP_LIBRARY ${WINFSP_ROOT_DIR}/lib/winfsp-x64.lib)
if(NOT EXISTS ${WINFSP_INCLUDE_DIR})
    message(FATAL_ERROR "WinFsp include directory not found: ${WINFSP_INCLUDE_DIR}")
endif()
if(NOT EXISTS ${WINFSP_LIBRARY})
    message(FATAL_ERROR "WinFsp library not found: ${WINFSP_LIBRARY}")
endif()
message(STATUS "WinFsp include: ${WINFSP_INCLUDE_DIR}")
message(STATUS "WinFsp library: ${WINFSP_LIBRARY}")

# Include modular services package FIRST to set up service sources
add_subdirectory(services)

# Source files
set(SOURCES
    src/main.cpp
    src/disk/disk_reader.cpp
    src/bitlocker/bitlocker_metadata_compat.cpp
    src/bitlocker/recovery_key.cpp
    src/bitlocker/fvek_derivation.cpp
    src/crypto/aes_xts.cpp
    src/crypto/aes_cbc.cpp
    src/crypto/elephant.cpp
    src/crypto/crypto_utils.cpp
    src/virtual_disk/decrypted_disk.cpp
    src/ntfs/ntfs_boot.cpp
    src/ntfs/mft_parser.cpp
    src/ntfs/runlist.cpp
    src/ntfs/attribute_list.cpp
    src/ntfs/directory.cpp
    src/ntfs/file_reader.cpp
    src/ntfs/path_resolver.cpp
    src/winfsp/winfsp_filesystem.cpp
    src/gui/forensic_gui.cpp
    src/cli/command_parser.cpp
    src/cli/imaging_command.cpp
    src/cli/mount_command.cpp
    src/cli/verify_command.cpp
    src/repair/repair_volume.cpp
    src/imaging/disk_imager.cpp
    src/imaging/imaging_engine.cpp
    src/imaging/hash_verifier.cpp
    src/imaging/image_reader.cpp
    src/imaging/integrity_verifier.cpp
    src/utils/logging.cpp
    src/utils/progress_tracker.cpp
    src/utils/database.cpp
    src/recovery/file_recovery.cpp
    src/recovery/bad_sector_engine.cpp
    src/recovery/file_carver.cpp
    src/recovery/fragment_recovery.cpp
    src/cli/recovery_command.cpp
    src/hashing/hash_engine.cpp
    src/mounting/virtual_mounter.cpp
    src/reporting/forensic_report.cpp
    ${SERVICES_SRCS}  # Include service layer sources
)

add_executable(bltool ${SOURCES})

target_include_directories(bltool PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/include
    ${CMAKE_CURRENT_SOURCE_DIR}/src
    ${CMAKE_CURRENT_SOURCE_DIR}/src/imaging
    ${CMAKE_CURRENT_SOURCE_DIR}/src/utils
    ${CMAKE_CURRENT_SOURCE_DIR}/src/recovery
    ${CMAKE_CURRENT_SOURCE_DIR}/src/hashing
    ${CMAKE_CURRENT_SOURCE_DIR}/src/mounting
    ${CMAKE_CURRENT_SOURCE_DIR}/src/reporting
    ${CMAKE_CURRENT_SOURCE_DIR}/services/include
    ${OPENSSL_INCLUDE_DIR}
    ${WINFSP_INCLUDE_DIR}
)

target_link_libraries(bltool PRIVATE
    OpenSSL::Crypto
    ${WINFSP_LIBRARY}
    advapi32
    user32
    shell32
    # WMI libraries for BitLocker API
    wbemuuid
    ole32
    oleaut32
)

if(MSVC)
    target_compile_options(bltool PRIVATE /W4 /EHsc)
    set_property(TARGET bltool PROPERTY MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")
endif()
if(WIN32)
    set_target_properties(bltool PROPERTIES WIN32_EXECUTABLE FALSE)
endif()
install(TARGETS bltool RUNTIME DESTINATION bin)
message(STATUS "OpenSSL found: ${OPENSSL_VERSION}")
message(STATUS "WinFsp manually configured")

