cmake_minimum_required(VERSION 3.18)

project(skee64 LANGUAGES CXX RC)

if(PROJECT_SOURCE_DIR STREQUAL PROJECT_BINARY_DIR)
    message(FATAL_ERROR "In-source builds not allowed.")
endif()

# Option for store version (0 = Steam, 1 = GOG)
set(STORE_VERSION 0 CACHE STRING "Store version: 0 for Steam, 1 for GOG")

# ---- Shader compilation ----
set(FXC_EXECUTABLE "${CMAKE_CURRENT_SOURCE_DIR}/cmake/fxc.sh")

set(COMPILED_SHADERS "")

function(add_hlsl_shader SHADER_SRC SHADER_TARGET SHADER_ENTRY SHADER_OUT)
    set(FULL_SRC "${CMAKE_CURRENT_SOURCE_DIR}/${SHADER_SRC}")
    set(FULL_OUT "${CMAKE_CURRENT_BINARY_DIR}/${SHADER_OUT}")
    get_filename_component(OUT_DIR "${FULL_OUT}" DIRECTORY)

    add_custom_command(
        OUTPUT "${FULL_OUT}"
        COMMAND ${CMAKE_COMMAND} -E make_directory "${OUT_DIR}"
        COMMAND "${FXC_EXECUTABLE}" /nologo /T "${SHADER_TARGET}" /E "${SHADER_ENTRY}" /Fo "${FULL_OUT}" "${FULL_SRC}"
        DEPENDS "${FULL_SRC}" "${FXC_EXECUTABLE}"
        COMMENT "Compiling shader ${SHADER_SRC} -> ${SHADER_OUT}"
        VERBATIM
    )
    set(COMPILED_SHADERS ${COMPILED_SHADERS} "${FULL_OUT}" PARENT_SCOPE)
endfunction()

# CharGen Shaders
add_hlsl_shader("Shaders/CharGen/brush_ps.hlsl" ps_5_0 BrushPShader "Shaders/CharGen/Shaders/Compiled/brush_ps.cso")
add_hlsl_shader("Shaders/CharGen/brush_vs.hlsl" vs_5_0 BrushVShader "Shaders/CharGen/Shaders/Compiled/brush_vs.cso")
add_hlsl_shader("Shaders/CharGen/shader_ps.hlsl" ps_5_0 LightPixelShader "Shaders/CharGen/Shaders/Compiled/shader_ps.cso")
add_hlsl_shader("Shaders/CharGen/shader_vs.hlsl" vs_5_0 LightVertexShader "Shaders/CharGen/Shaders/Compiled/shader_vs.cso")
add_hlsl_shader("Shaders/CharGen/wireframe_gs.hlsl" gs_5_0 WireframeGeometryShader "Shaders/CharGen/Shaders/Compiled/wireframe_gs.cso")
add_hlsl_shader("Shaders/CharGen/wireframe_ps.hlsl" ps_5_0 WireframePixelShader "Shaders/CharGen/Shaders/Compiled/wireframe_ps.cso")
add_hlsl_shader("Shaders/CharGen/wireframe_vs.hlsl" vs_5_0 WireframeVertexShader "Shaders/CharGen/Shaders/Compiled/wireframe_vs.cso")

# NiOverride Shaders
add_hlsl_shader("Shaders/NiOverride/texture.fx" vs_5_0 TextureVertex "Shaders/NiOverride/Shaders/Compiled/texture.cso")
add_hlsl_shader("Shaders/NiOverride/Effects/Color.fx" ps_5_0 color "Shaders/NiOverride/Shaders/Compiled/Effects/Color.cso")
add_hlsl_shader("Shaders/NiOverride/Effects/ColorBurn.fx" ps_5_0 colorburn "Shaders/NiOverride/Shaders/Compiled/Effects/ColorBurn.cso")
add_hlsl_shader("Shaders/NiOverride/Effects/ColorDodge.fx" ps_5_0 colordodge "Shaders/NiOverride/Shaders/Compiled/Effects/ColorDodge.cso")
add_hlsl_shader("Shaders/NiOverride/Effects/Darken.fx" ps_5_0 darken "Shaders/NiOverride/Shaders/Compiled/Effects/Darken.cso")
add_hlsl_shader("Shaders/NiOverride/Effects/Grayscale.fx" ps_5_0 grayscale "Shaders/NiOverride/Shaders/Compiled/Effects/Grayscale.cso")
add_hlsl_shader("Shaders/NiOverride/Effects/Lighten.fx" ps_5_0 lighten "Shaders/NiOverride/Shaders/Compiled/Effects/Lighten.cso")
add_hlsl_shader("Shaders/NiOverride/Effects/LinearBurn.fx" ps_5_0 linearburn "Shaders/NiOverride/Shaders/Compiled/Effects/LinearBurn.cso")
add_hlsl_shader("Shaders/NiOverride/Effects/LinearDodge.fx" ps_5_0 lineardodge "Shaders/NiOverride/Shaders/Compiled/Effects/LinearDodge.cso")
add_hlsl_shader("Shaders/NiOverride/Effects/LinearLight.fx" ps_5_0 linearlight "Shaders/NiOverride/Shaders/Compiled/Effects/LinearLight.cso")
add_hlsl_shader("Shaders/NiOverride/Effects/Multiply.fx" ps_5_0 multiply "Shaders/NiOverride/Shaders/Compiled/Effects/Multiply.cso")
add_hlsl_shader("Shaders/NiOverride/Effects/Normal.fx" ps_5_0 normal "Shaders/NiOverride/Shaders/Compiled/Effects/Normal.cso")
add_hlsl_shader("Shaders/NiOverride/Effects/Overlay.fx" ps_5_0 overlay "Shaders/NiOverride/Shaders/Compiled/Effects/Overlay.cso")
add_hlsl_shader("Shaders/NiOverride/Effects/RNM.fx" ps_5_0 rnm "Shaders/NiOverride/Shaders/Compiled/Effects/RNM.cso")
add_hlsl_shader("Shaders/NiOverride/Effects/SoftLight.fx" ps_5_0 softlight "Shaders/NiOverride/Shaders/Compiled/Effects/SoftLight.cso")
add_hlsl_shader("Shaders/NiOverride/Effects/Tint.fx" ps_5_0 tint "Shaders/NiOverride/Shaders/Compiled/Effects/Tint.cso")

add_custom_target(skee64_shaders ALL DEPENDS ${COMPILED_SHADERS})

# ---- SKSE64 API Static Library ----
include("${CMAKE_CURRENT_SOURCE_DIR}/../../skse64/cmake/versioning.cmake")

set(SKSE64_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../../skse64")

set(SKSE64_SDK_SOURCES
    "${SKSE64_DIR}/skse64/BSModelDB.cpp"
    "${SKSE64_DIR}/skse64/Colors.cpp"
    "${SKSE64_DIR}/skse64/CustomMenu.cpp"
    "${SKSE64_DIR}/skse64/GameAPI.cpp"
    "${SKSE64_DIR}/skse64/GameBSExtraData.cpp"
    "${SKSE64_DIR}/skse64/GameCamera.cpp"
    "${SKSE64_DIR}/skse64/GameData.cpp"
    "${SKSE64_DIR}/skse64/GameEvents.cpp"
    "${SKSE64_DIR}/skse64/GameExtraData.cpp"
    "${SKSE64_DIR}/skse64/GameFormComponents.cpp"
    "${SKSE64_DIR}/skse64/GameForms.cpp"
    "${SKSE64_DIR}/skse64/GameHandlers.cpp"
    "${SKSE64_DIR}/skse64/GameInput.cpp"
    "${SKSE64_DIR}/skse64/GameMenus.cpp"
    "${SKSE64_DIR}/skse64/GameObjects.cpp"
    "${SKSE64_DIR}/skse64/GamePathing.cpp"
    "${SKSE64_DIR}/skse64/GameRTTI.cpp"
    "${SKSE64_DIR}/skse64/GameReferences.cpp"
    "${SKSE64_DIR}/skse64/GameResources.cpp"
    "${SKSE64_DIR}/skse64/GameSettings.cpp"
    "${SKSE64_DIR}/skse64/GameStreams.cpp"
    "${SKSE64_DIR}/skse64/GameThreads.cpp"
    "${SKSE64_DIR}/skse64/GameTypes.cpp"
    "${SKSE64_DIR}/skse64/GameUtilities.cpp"
    "${SKSE64_DIR}/skse64/GlobalLocks.cpp"
    "${SKSE64_DIR}/skse64/HashUtil.cpp"
    "${SKSE64_DIR}/skse64/InputMap.cpp"
    "${SKSE64_DIR}/skse64/InternalTasks.cpp"
    "${SKSE64_DIR}/skse64/NiAdditionalGeometryData.cpp"
    "${SKSE64_DIR}/skse64/NiControllers.cpp"
    "${SKSE64_DIR}/skse64/NiExtraData.cpp"
    "${SKSE64_DIR}/skse64/NiGeometry.cpp"
    "${SKSE64_DIR}/skse64/NiInterpolators.cpp"
    "${SKSE64_DIR}/skse64/NiLight.cpp"
    "${SKSE64_DIR}/skse64/NiMaterial.cpp"
    "${SKSE64_DIR}/skse64/NiNodes.cpp"
    "${SKSE64_DIR}/skse64/NiObjects.cpp"
    "${SKSE64_DIR}/skse64/NiProperties.cpp"
    "${SKSE64_DIR}/skse64/NiRTTI.cpp"
    "${SKSE64_DIR}/skse64/NiRenderer.cpp"
    "${SKSE64_DIR}/skse64/NiSerialization.cpp"
    "${SKSE64_DIR}/skse64/NiTextures.cpp"
    "${SKSE64_DIR}/skse64/NiTypes.cpp"
    "${SKSE64_DIR}/skse64/ObScript.cpp"
    "${SKSE64_DIR}/skse64/PapyrusArgs.cpp"
    "${SKSE64_DIR}/skse64/PapyrusDelayFunctors.cpp"
    "${SKSE64_DIR}/skse64/PapyrusInterfaces.cpp"
    "${SKSE64_DIR}/skse64/PapyrusNativeFunctions.cpp"
    "${SKSE64_DIR}/skse64/PapyrusObjects.cpp"
    "${SKSE64_DIR}/skse64/PapyrusVM.cpp"
    "${SKSE64_DIR}/skse64/PapyrusValue.cpp"
    "${SKSE64_DIR}/skse64/ScaleformAPI.cpp"
    "${SKSE64_DIR}/skse64/ScaleformCallbacks.cpp"
    "${SKSE64_DIR}/skse64/ScaleformExtendedData.cpp"
    "${SKSE64_DIR}/skse64/ScaleformLoader.cpp"
    "${SKSE64_DIR}/skse64/ScaleformMovie.cpp"
    "${SKSE64_DIR}/skse64/ScaleformState.cpp"
    "${SKSE64_DIR}/skse64/ScaleformTypes.cpp"
    "${SKSE64_DIR}/skse64/ScaleformVM.cpp"
    "${SKSE64_DIR}/skse64/ScaleformValue.cpp"
    "${SKSE64_DIR}/skse64/Serialization.cpp"
    "${SKSE64_DIR}/skse64/Translation.cpp"
)

# Stub for ResolveModIndex needed by Serialization.cpp when built as static library
file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/skse64_stubs.cpp" [=[
#include "common/ITypes.h"
UInt32 ResolveModIndex(UInt32 modIndex) { return 0xFF; }
]=])

add_library(skse64_api STATIC ${SKSE64_SDK_SOURCES} "${CMAKE_CURRENT_BINARY_DIR}/skse64_stubs.cpp")
target_include_directories(skse64_api PUBLIC
    "${SKSE64_DIR}"
    "${SKSE64_DIR}/skse64"
    "${SKSE64_DIR}/skse64_common"
    "${SKSE64_DIR}/../common"
    "${SKSE64_DIR}/../extern/include"
)
target_compile_features(skse64_api PUBLIC cxx_std_11)
target_compile_definitions(skse64_api PRIVATE
    WIN32
    _WINDOWS
    RUNTIME
    RUNTIME_VERSION=${RUNTIME_VERSION_PACKED}
)
target_compile_options(skse64_api PRIVATE /FIcommon/IPrefix.h)
set_target_properties(skse64_api PROPERTIES
    MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>"
)

# ---- skee64 Source files ----
set(SKEE64_SOURCES
    ActorArmorTangentUpdater.cpp
    ActorUpdateManager.cpp
    AttachmentInterface.cpp
    BodyMorphInterface.cpp
    CDXBrush.cpp
    CDXBrushMesh.cpp
    CDXBSShaderResource.cpp
    CDXCamera.cpp
    CDXEditableMesh.cpp
    CDXEditableScene.cpp
    CDXMaterial.cpp
    CDXMesh.cpp
    CDXNifBrush.cpp
    CDXNifCommands.cpp
    CDXNifMaterial.cpp
    CDXNifMesh.cpp
    CDXNifPixelShaderCache.cpp
    CDXNifScene.cpp
    CDXNifTextureRenderer.cpp
    CDXPicker.cpp
    CDXPixelShaderCache.cpp
    CDXRenderState.cpp
    CDXResetMask.cpp
    CDXScene.cpp
    CDXShader.cpp
    CDXShaderCompile.cpp
    CDXShaderFactory.cpp
    CDXStroke.cpp
    CDXTextureRenderer.cpp
    CDXUndo.cpp
    CommandInterface.cpp
    FaceMorphInterface.cpp
    FileUtils.cpp
    FormTagInterface.cpp
    ILogger.cpp
    ItemDataInterface.cpp
    main.cpp
    Morpher.cpp
    NiTransformInterface.cpp
    NifUtils.cpp
    OverlayInterface.cpp
    OverrideInterface.cpp
    OverrideVariant.cpp
    PapyrusCharGen.cpp
    PapyrusNiOverride.cpp
    PapyrusUtilities.cpp
    PartHandler.cpp
    PluginInterface.cpp
    PresetInterface.cpp
    SKEEHooks.cpp
    ScaleformCharGenFunctions.cpp
    ScaleformFunctions.cpp
    ScaleformUtils.cpp
    ShaderUtilities.cpp
    SkeletonExtender.cpp
    SkinLayerInterface.cpp
    StringTable.cpp
    TintMaskInterface.cpp
    tinyxml2.cpp
    Utilities.cpp
)

set(SKEE64_HEADERS
    ActorArmorTangentUpdater.h
    ActorUpdateManager.h
    AttachmentInterface.h
    BodyMorphInterface.h
    CDXBrush.h
    CDXBrushMesh.h
    CDXBSShaderResource.h
    CDXCamera.h
    CDXD3DDevice.h
    CDXEditableMesh.h
    CDXEditableScene.h
    CDXMaterial.h
    CDXMesh.h
    CDXNifBrush.h
    CDXNifCommands.h
    CDXNifMaterial.h
    CDXNifMesh.h
    CDXNifPixelShaderCache.h
    CDXNifScene.h
    CDXNifTextureRenderer.h
    CDXPicker.h
    CDXPixelShaderCache.h
    CDXRenderState.h
    CDXResetMask.h
    CDXScene.h
    CDXShader.h
    CDXShaderCompile.h
    CDXShaderFactory.h
    CDXStroke.h
    CDXTextureRenderer.h
    CDXTypes.h
    CDXUndo.h
    CommandInterface.h
    FaceMorphInterface.h
    FileUtils.h
    FormTagInterface.h
    Hooks_chargen.h
    IHashType.h
    ILogger.h
    IPluginInterface.h
    ITypes.h
    ItemDataInterface.h
    Morpher.h
    NiTransformInterface.h
    NifUtils.h
    OverlayInterface.h
    OverrideInterface.h
    OverrideVariant.h
    PapyrusCharGen.h
    PapyrusNiOverride.h
    PapyrusUtilities.h
    PartHandler.h
    PluginInterface.h
    PresetInterface.h
    SKEEHooks.h
    ScaleformCharGenFunctions.h
    ScaleformFunctions.h
    ScaleformUtils.h
    ShaderUtilities.h
    SkeletonExtender.h
    SkinLayerInterface.h
    StringTable.h
    TintMaskInterface.h
    Utilities.h
    half.hpp
    kd_matcher.hpp
    resource.h
    shape.hpp
    tinyxml2.h
)

# ---- Target ----
add_library(${PROJECT_NAME} SHARED
    ${SKEE64_SOURCES}
    ${SKEE64_HEADERS}
    Resource.rc
    exports.def
)

add_dependencies(${PROJECT_NAME} skee64_shaders)

target_compile_features(${PROJECT_NAME} PUBLIC cxx_std_20)

target_compile_definitions(${PROJECT_NAME} PRIVATE
    WIN32
    _WINDOWS
    _USRDLL
    HALF_ROUND_STYLE=1
    HALF_ENABLE_CPP11_CMATH=0
    XBYAK_NO_OP_NAMES
    STORE_VERSION=${STORE_VERSION}
)

target_compile_options(${PROJECT_NAME} PRIVATE
    /FIITypes.h
)

target_include_directories(${PROJECT_NAME} PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/..
    ${CMAKE_CURRENT_SOURCE_DIR}/../../skse64
    ${CMAKE_CURRENT_SOURCE_DIR}/../../skse64/skse64
    ${CMAKE_CURRENT_SOURCE_DIR}/../../skse64/skse64_common
    ${CMAKE_CURRENT_SOURCE_DIR}/../../extern/include
    ${CMAKE_CURRENT_SOURCE_DIR}/../../common
)

# Check if skse64_common is available as target
if(TARGET skse64_common)
    target_link_libraries(${PROJECT_NAME} PRIVATE skse64_common)
else()
    target_link_libraries(${PROJECT_NAME} PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/../../skse64/build/skse64/skse64_common/skse64_common.lib")
endif()

# Check if common is available as target
if(TARGET common)
    target_link_libraries(${PROJECT_NAME} PRIVATE common)
elseif(TARGET xse::common)
    target_link_libraries(${PROJECT_NAME} PRIVATE xse::common)
else()
    target_link_libraries(${PROJECT_NAME} PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/../../extern/lib/common.lib")
endif()

target_link_libraries(${PROJECT_NAME} PRIVATE
    skse64_api
    jsoncpp
    DirectXTex
    shlwapi.lib
    d3d11.lib
    dxgi.lib
    d3dcompiler.lib
)

set_target_properties(${PROJECT_NAME} PROPERTIES
    MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>"
    OUTPUT_NAME "skee64"
)


