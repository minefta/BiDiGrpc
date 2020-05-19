cmake_minimum_required(VERSION 3.16)
project(protofiles)

include(${CMAKE_SOURCE_DIR}/common.cmake)

find_program(GRPC_CPP_PLUGIN NAMES grpc_cpp_plugin)
find_library(GRPC_LIBRARY NAMES grpc)
find_library(GRPCPP_LIBRARY NAMES grpc++)
find_library(GPR_LIBRARY NAMES gpr)

#set(OPENSSL_USE_STATIC_LIBS TRUE)
#find_package(OpenSSL 1.0.0 REQUIRED)
#include_directories(${OPENSSL_INCLUDE_DIR})


set(GRPC_LIBRARIES ${GRPCPP_LIBRARY} ${GRPC_LIBRARY} ${GPR_LIBRARY})

if(GRPC_LIBRARIES)
    message(STATUS "Found GRPC: ${GRPC_LIBRARIES}; plugin - ${GRPC_CPP_PLUGIN}")
endif()

find_package(Protobuf REQUIRED)


set(GENERATED_PROTOBUF_PATH "${CMAKE_BINARY_DIR}/generated_protos")

set(PROTO_FILES
        subscribe.proto
        timestamp.proto
        )

set(GENERATED_PROTOBUF_FILES
        ${GENERATED_PROTOBUF_PATH}/subscribe.pb.cc
        ${GENERATED_PROTOBUF_PATH}/subscribe.pb.h
        ${GENERATED_PROTOBUF_PATH}/subscribe.grpc.pb.cc
        ${GENERATED_PROTOBUF_PATH}/subscribe.grpc.pb.h

        ${GENERATED_PROTOBUF_PATH}/timestamp.pb.cc
        ${GENERATED_PROTOBUF_PATH}/timestamp.pb.h
        ${GENERATED_PROTOBUF_PATH}/timestamp.grpc.pb.cc
        ${GENERATED_PROTOBUF_PATH}/timestamp.grpc.pb.h

        )

add_custom_command(
        DEPENDS ${PROTO_FILES}
        OUTPUT ${GENERATED_PROTOBUF_FILES}

        COMMAND ${PROTOBUF_PROTOC_EXECUTABLE}
        ARGS "--proto_path=${CMAKE_CURRENT_SOURCE_DIR}"
        "--cpp_out=${GENERATED_PROTOBUF_PATH}"
        ${PROTO_FILES}

        COMMAND ${PROTOBUF_PROTOC_EXECUTABLE}
        ARGS "--proto_path=${CMAKE_CURRENT_SOURCE_DIR}"
        "--grpc_out=${GENERATED_PROTOBUF_PATH}"
        "--plugin=protoc-gen-grpc=${GRPC_CPP_PLUGIN}"
        ${PROTO_FILES}

        COMMENT "Running gRPC C++ protocol buffer compiler on ${PROTO_FILES}"
        VERBATIM
)

include_directories(${GENERATED_PROTOBUF_PATH})
include_directories(${Protobuf_INCLUDE_DIRS})

add_library(${PROJECT_NAME} STATIC ${GENERATED_PROTOBUF_FILES} ${PROTO_FILES})

target_link_libraries(${PROJECT_NAME}
        ${GRPC_LIBRARIES}
        ${PROTOBUF_LIBRARY}
#        ${OPENSSL_LIBRARIES}
        z
        pthread
        )
