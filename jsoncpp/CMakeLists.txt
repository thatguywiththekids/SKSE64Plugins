cmake_minimum_required(VERSION 3.18)

project(jsoncpp LANGUAGES CXX)

set(JSONCPP_SOURCES
    json_reader.cpp
    json_value.cpp
    json_writer.cpp
)

set(JSONCPP_HEADERS
    json/assertions.h
    json/autolink.h
    json/config.h
    json/features.h
    json/forwards.h
    json/json.h
    json/reader.h
    json/value.h
    json/writer.h
    json_batchallocator.h
    json_tool.h
    version.h
)

add_library(jsoncpp STATIC
    ${JSONCPP_SOURCES}
    ${JSONCPP_HEADERS}
)

target_include_directories(jsoncpp PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/..>
    $<INSTALL_INTERFACE:include>
)

target_compile_features(jsoncpp PUBLIC cxx_std_20)

set_target_properties(jsoncpp PROPERTIES
    MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>"
)
