# Let's bring the grpc module.
add_subdirectory(${ANDROID_QEMU2_TOP_DIR}/../grpc/emulator grpc)

set(grpc_proto_src automation.proto)
protobuf_generate_grpc_cpp(SOURCE_DIR ${CMAKE_CURRENT_LIST_DIR}
                           SOURCES android/emulation/control/emulator_controller.proto
                           OUTPUT_DIR ${CMAKE_CURRENT_BINARY_DIR}
                           GENERATED EMULATOR_GRPC_SRC)

set(android-grpc_src ${EMULATOR_GRPC_SRC} android/emulation/control/EmulatorService.cpp
    android/emulation/control/EmulatorServiceStub.cpp
    android/emulation/control/keyboard/EmulatorKeyEventSender.cpp
    android/emulation/control/GrpcServices.cpp
    )

# Let's add in the library
android_add_library(android-grpc)
target_link_libraries(android-grpc PRIVATE grpc++ android-emu-base png PUBLIC libprotobuf)
target_include_directories(android-grpc
    PRIVATE
    # For Renderer.h
    ${ANDROID_QEMU2_TOP_DIR}/android/android-emugl/host/include
    PUBLIC
    ${CMAKE_CURRENT_BINARY_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR})
target_compile_definitions(android-grpc PUBLIC -DANDROID_GRPC)

# Lets distribute the proto file
install(FILES android/emulation/control/emulator_controller.proto DESTINATION lib)


# Add the benchmark
set(keytranslate_benchmark_src android/emulation/control/keyboard/KeytranslatePerf.cpp)
android_add_executable(keytranslate_benchmark)
target_link_libraries(keytranslate_benchmark PRIVATE android-grpc android-emu-base emulator-gbench)
