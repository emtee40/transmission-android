cmake_minimum_required(VERSION 3.4.1)
project(transmissionbtc)

if (NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Build type: [Debug|Release]" FORCE)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release")
endif ()

set(EXT_C_FLAGS "-DANDROID -fno-unwind-tables -no-canonical-prefixes -D_FORTIFY_SOURCE=1 \
                 -D_FILE_OFFSET_BITS=64 -D_LARGE_FILES=1")
set(EXT_CXX_FLAGS "${EXT_C_FLAGS} -fno-exceptions -frtti -std=gnu++17")
set(EXT_EXE_LINKER_FLAGS "-Wl,--exclude-libs,libgcc.a -Wl,--exclude-libs,libgcc_real.a \
                          -Wl,--exclude-libs,libatomic.a -Wl,--gc-sections -static-libstdc++")

if (CMAKE_BUILD_TYPE STREQUAL "Debug")
    set(EXT_C_FLAGS "-g -O0 ${EXT_C_FLAGS}")
    set(EXT_EXE_LINKER_FLAGS "-O0 ${EXT_EXE_LINKER_FLAGS}")
    set(CMAKE_C_FLAGS_DEBUG ${EXT_C_FLAGS})
    set(CMAKE_EXE_LINKER_FLAGS_DEBUG ${EXT_EXE_LINKER_FLAGS})
else ()
    set(EXT_C_FLAGS "-O3 -DNDEBUG -flto -fvisibility=hidden -fdata-sections -ffunction-sections ${EXT_C_FLAGS}")
    set(EXT_CXX_FLAGS "-O3 -DNDEBUG -flto -fvisibility=hidden -fdata-sections -ffunction-sections ${EXT_CXX_FLAGS}")
    set(EXT_EXE_LINKER_FLAGS "-fuse-ld=gold -flto -O3 -Wl,--strip-all ${EXT_EXE_LINKER_FLAGS}")
    set(CMAKE_C_FLAGS_RELEASE ${EXT_C_FLAGS})
    set(CMAKE_CXX_FLAGS_RELEASE ${EXT_CXX_FLAGS})
    set(CMAKE_EXE_LINKER_FLAGS_RELEASE ${EXT_EXE_LINKER_FLAGS})
endif ()

set(EXT_CMAKE_ARGS -DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE}
        -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
        -DANDROID_ABI=${ANDROID_ABI}
        -DANDROID_PLATFORM=${ANDROID_NATIVE_API_LEVEL}
        -DCMAKE_FIND_ROOT_PATH=${CMAKE_CURRENT_BINARY_DIR}
        -DCMAKE_FIND_ROOT_PATH_MODE_PROGRAM=true
        -DCMAKE_FIND_ROOT_PATH_MODE_LIBRARY=true
        -DCMAKE_FIND_ROOT_PATH_MODE_INCLUDE=true
        -DCMAKE_FIND_ROOT_PATH_MODE_PACKAGE=true
        -DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}
        -DCMAKE_DEBUG_POSTFIX=''
        -DCMAKE_C_FLAGS=${EXT_C_FLAGS}
        -DCMAKE_CXX_FLAGS=${EXT_CXX_FLAGS}
        -DCMAKE_EXE_LINKER_FLAGS=${EXT_EXE_LINKER_FLAGS}
        )

add_library(${PROJECT_NAME} SHARED
        "${CMAKE_SOURCE_DIR}/src/main/cpp/native_to_java.cc"
        "${CMAKE_SOURCE_DIR}/src/main/cpp/env.cc"
        "${CMAKE_SOURCE_DIR}/src/main/cpp/sem.cc"
        "${CMAKE_SOURCE_DIR}/src/main/cpp/curl.cc"
        "${CMAKE_SOURCE_DIR}/src/main/cpp/hash.cc"
        "${CMAKE_SOURCE_DIR}/src/main/cpp/commons.cc"
        "${CMAKE_SOURCE_DIR}/src/main/cpp/torrent.cc"
        "${CMAKE_SOURCE_DIR}/src/main/cpp/transmission.cc"
        "${CMAKE_SOURCE_DIR}/src/main/cpp/stdredirect.cc"
        )

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
include(OpenSSL)
include(cURL)
include(Event)
include(Transmission)

include_directories(SYSTEM
        ${OPENSSL_INCLUDE_DIR}
        ${CURL_INCLUDE_DIR}
        ${EVENT_INCLUDE_DIR}
        ${TR_INCLUDE_DIR})

target_link_libraries(${PROJECT_NAME}
        ${TR_LIBRARIES}
        ${CURL_LIBRARIES}
        ${EVENT_LIBRARIES}
        ${OPENSSL_LIBRARIES}
        log
        z)
