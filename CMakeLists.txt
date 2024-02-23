cmake_minimum_required(VERSION 3.11)

if(POLICY CMP0077)
    cmake_policy(SET CMP0077 NEW)
endif()

project("sodium" LANGUAGES C)

option(SODIUM_DISABLE_TESTS "Disable tests" OFF)
option(SODIUM_MINIMAL "Only compile the minimum set of functions required for the high-level API" OFF)
option(SODIUM_ENABLE_BLOCKING_RANDOM "Enable this switch only if /dev/urandom is totally broken on the target platform" OFF)

file(GLOB_RECURSE LIB_HEADERS_H ${CMAKE_CURRENT_SOURCE_DIR}/*.h)
file(GLOB_RECURSE LIB_SOUCES_C ${CMAKE_CURRENT_SOURCE_DIR}/*.c)

add_library(${PROJECT_NAME}
        ${LIB_HEADERS_H}
        ${LIB_SOUCES_C}
)

if(NOT SODIUM_MINIMAL)
    target_sources(${PROJECT_NAME}
        PRIVATE
            libsodium/src/libsodium/crypto_box/curve25519xchacha20poly1305/box_curve25519xchacha20poly1305.c
            libsodium/src/libsodium/crypto_box/curve25519xchacha20poly1305/box_seal_curve25519xchacha20poly1305.c
            libsodium/src/libsodium/crypto_core/ed25519/core_ed25519.c
            libsodium/src/libsodium/crypto_core/ed25519/core_ristretto255.c
            libsodium/src/libsodium/crypto_pwhash/scryptsalsa208sha256/crypto_scrypt-common.c
            libsodium/src/libsodium/crypto_pwhash/scryptsalsa208sha256/crypto_scrypt.h
            libsodium/src/libsodium/crypto_pwhash/scryptsalsa208sha256/nosse/pwhash_scryptsalsa208sha256_nosse.c
            libsodium/src/libsodium/crypto_pwhash/scryptsalsa208sha256/pbkdf2-sha256.c
            libsodium/src/libsodium/crypto_pwhash/scryptsalsa208sha256/pbkdf2-sha256.h
            libsodium/src/libsodium/crypto_pwhash/scryptsalsa208sha256/pwhash_scryptsalsa208sha256.c
            libsodium/src/libsodium/crypto_pwhash/scryptsalsa208sha256/scrypt_platform.c
            libsodium/src/libsodium/crypto_pwhash/scryptsalsa208sha256/sse/pwhash_scryptsalsa208sha256_sse.c
            libsodium/src/libsodium/crypto_scalarmult/ed25519/ref10/scalarmult_ed25519_ref10.c
            libsodium/src/libsodium/crypto_scalarmult/ristretto255/ref10/scalarmult_ristretto255_ref10.c
            libsodium/src/libsodium/crypto_secretbox/xchacha20poly1305/secretbox_xchacha20poly1305.c
            libsodium/src/libsodium/crypto_shorthash/siphash24/ref/shorthash_siphashx24_ref.c
            libsodium/src/libsodium/crypto_shorthash/siphash24/shorthash_siphashx24.c
            libsodium/src/libsodium/crypto_sign/ed25519/ref10/obsolete.c
            libsodium/src/libsodium/crypto_stream/salsa2012/ref/stream_salsa2012_ref.c
            libsodium/src/libsodium/crypto_stream/salsa2012/stream_salsa2012.c
            libsodium/src/libsodium/crypto_stream/salsa208/ref/stream_salsa208_ref.c
            libsodium/src/libsodium/crypto_stream/salsa208/stream_salsa208.c
            libsodium/src/libsodium/crypto_stream/xchacha20/stream_xchacha20.c
    )
endif()

set_target_properties(${PROJECT_NAME}
    PROPERTIES
        C_STANDARD 99
)

target_include_directories(${PROJECT_NAME}
    PUBLIC
        libsodium/src/libsodium/include
    PRIVATE
        libsodium/src/libsodium/include/sodium
)

target_compile_definitions(${PROJECT_NAME}
    PUBLIC
        $<$<NOT:$<BOOL:${BUILD_SHARED_LIBS}>>:SODIUM_STATIC>
        $<$<BOOL:${SODIUM_MINIMAL}>:SODIUM_LIBRARY_MINIMAL>
    PRIVATE
        CONFIGURED
        $<$<BOOL:${BUILD_SHARED_LIBS}>:SODIUM_DLL_EXPORT>
        $<$<BOOL:${SODIUM_ENABLE_BLOCKING_RANDOM}>:USE_BLOCKING_RANDOM>
        $<$<BOOL:${SODIUM_MINIMAL}>:MINIMAL>
        $<$<C_COMPILER_ID:MSVC>:_CRT_SECURE_NO_WARNINGS>
)

# Variables that need to be exported to version.h.in
set(VERSION 1.0.19)
set(SODIUM_LIBRARY_VERSION_MAJOR 26)
set(SODIUM_LIBRARY_VERSION_MINOR 1)
if(SODIUM_MINIMAL)
    set(SODIUM_LIBRARY_MINIMAL_DEF "#define SODIUM_LIBRARY_MINIMAL 1")
endif()

configure_file(
    libsodium/src/libsodium/include/sodium/version.h.in
    ${CMAKE_CURRENT_SOURCE_DIR}/libsodium/src/libsodium/include/sodium/version.h
)

if(NOT SODIUM_DISABLE_TESTS)
    enable_testing()
    add_subdirectory(test)
endif()
