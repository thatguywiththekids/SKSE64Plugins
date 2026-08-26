cmake_minimum_required(VERSION 3.18)

project(SKSE64Plugins LANGUAGES CXX RC)

if(PROJECT_SOURCE_DIR STREQUAL PROJECT_BINARY_DIR)
    message(FATAL_ERROR "In-source builds not allowed.")
endif()

if(POLICY CMP0169)
    cmake_policy(SET CMP0169 OLD)
endif()

set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>" CACHE STRING "")

# Option for store version (0 = Steam, 1 = GOG)
set(STORE_VERSION 0 CACHE STRING "Store version: 0 for Steam, 1 for GOG")

include(FetchContent)

set(BUILD_TOOLS OFF CACHE BOOL "" FORCE)
set(BUILD_SAMPLE OFF CACHE BOOL "" FORCE)
set(BUILD_DX11 ON CACHE BOOL "" FORCE)
set(BUILD_DX12 OFF CACHE BOOL "" FORCE)
set(CMAKE_DISABLE_PRECOMPILE_HEADERS ON CACHE BOOL "" FORCE)
set(USE_PREBUILT_SHADERS ON CACHE BOOL "" FORCE)

set(DIRECTXTEX_COMPILED_SHADERS_DIR "${CMAKE_CURRENT_BINARY_DIR}/directxtex_shaders")
file(MAKE_DIRECTORY "${DIRECTXTEX_COMPILED_SHADERS_DIR}")
set(COMPILED_SHADERS "${DIRECTXTEX_COMPILED_SHADERS_DIR}" CACHE PATH "" FORCE)

FetchContent_Declare(
    directxtex
    GIT_REPOSITORY https://github.com/microsoft/DirectXTex.git
    GIT_TAG sept2023
    GIT_SHALLOW TRUE
)

FetchContent_GetProperties(directxtex)
if(NOT directxtex_POPULATED)
    FetchContent_Populate(directxtex)
endif()

set(FXC_EXECUTABLE "${CMAKE_CURRENT_SOURCE_DIR}/skee64/cmake/fxc.sh")

set(DT_SHADER_OUTPUTS "")

function(add_dt_shader SHADER_BASE SHADER_ENTRY)
    set(SHADER_FILE "${directxtex_SOURCE_DIR}/DirectXTex/Shaders/${SHADER_BASE}.hlsl")
    set(OUT_50 "${DIRECTXTEX_COMPILED_SHADERS_DIR}/${SHADER_BASE}_${SHADER_ENTRY}.inc")
    set(OUT_40 "${DIRECTXTEX_COMPILED_SHADERS_DIR}/${SHADER_BASE}_${SHADER_ENTRY}_cs40.inc")

    add_custom_command(
        OUTPUT "${OUT_50}"
        COMMAND "${FXC_EXECUTABLE}" "${SHADER_FILE}" /nologo /WX /Ges /Zi /Zpc /Qstrip_reflect /Qstrip_debug /Tcs_5_0 "/E${SHADER_ENTRY}" "/Fh${OUT_50}" "/Vn${SHADER_BASE}_${SHADER_ENTRY}"
        COMMAND sed -i -e "s/#endif/\\n#endif/" "${OUT_50}"
        DEPENDS "${SHADER_FILE}" "${FXC_EXECUTABLE}"
        COMMENT "Compiling DirectXTex shader ${SHADER_BASE} ${SHADER_ENTRY} cs_5_0"
        VERBATIM
    )

    add_custom_command(
        OUTPUT "${OUT_40}"
        COMMAND "${FXC_EXECUTABLE}" "${SHADER_FILE}" /nologo /WX /Ges /Zi /Zpc /Qstrip_reflect /Qstrip_debug /Tcs_4_0 /DEMULATE_F16C "/E${SHADER_ENTRY}" "/Fh${OUT_40}" "/Vn${SHADER_BASE}_${SHADER_ENTRY}"
        COMMAND sed -i -e "s/#endif/\\n#endif/" "${OUT_40}"
        DEPENDS "${SHADER_FILE}" "${FXC_EXECUTABLE}"
        COMMENT "Compiling DirectXTex shader ${SHADER_BASE} ${SHADER_ENTRY} cs_4_0"
        VERBATIM
    )

    set(DT_SHADER_OUTPUTS ${DT_SHADER_OUTPUTS} "${OUT_50}" "${OUT_40}" PARENT_SCOPE)
endfunction()

add_dt_shader(BC7Encode TryMode456CS)
add_dt_shader(BC7Encode TryMode137CS)
add_dt_shader(BC7Encode TryMode02CS)
add_dt_shader(BC7Encode EncodeBlockCS)
add_dt_shader(BC6HEncode TryModeG10CS)
add_dt_shader(BC6HEncode TryModeLE10CS)
add_dt_shader(BC6HEncode EncodeBlockCS)

add_custom_target(directxtex_shaders ALL DEPENDS ${DT_SHADER_OUTPUTS})

add_subdirectory(${directxtex_SOURCE_DIR} ${directxtex_BINARY_DIR})
add_dependencies(DirectXTex directxtex_shaders)

add_subdirectory(jsoncpp)
add_subdirectory(skee64)
