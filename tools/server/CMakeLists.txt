set(TARGET llama-server)
set(TARGET_DIFFUSION llama-server-diffusion)

include_directories(${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_BINARY_DIR})

if (MINGW)
    # fix: https://github.com/ggml-org/llama.cpp/actions/runs/9651004652/job/26617901362?pr=8006
    add_compile_definitions(_WIN32_WINNT=${GGML_WIN_VER})
endif()

set(TARGET_SRCS
    server.cpp
    utils.hpp
)

set(TARGET_DIFFUSION_SRCS
    server-diffusion.cpp
    utils.hpp
)

set(PUBLIC_ASSETS
    index.html.gz
    loading.html
)

foreach(asset ${PUBLIC_ASSETS})
    set(input "${CMAKE_CURRENT_SOURCE_DIR}/public/${asset}")
    set(output "${CMAKE_CURRENT_BINARY_DIR}/${asset}.hpp")
    list(APPEND TARGET_SRCS ${output})
    list(APPEND TARGET_DIFFUSION_SRCS ${output})
    add_custom_command(
        DEPENDS "${input}"
        OUTPUT "${output}"
        COMMAND "${CMAKE_COMMAND}" "-DINPUT=${input}" "-DOUTPUT=${output}" -P "${PROJECT_SOURCE_DIR}/scripts/xxd.cmake"
    )
    set_source_files_properties(${output} PROPERTIES GENERATED TRUE)
endforeach()

# Original server target
add_executable(${TARGET} ${TARGET_SRCS})
install(TARGETS ${TARGET} RUNTIME)

target_include_directories(${TARGET} PRIVATE ../mtmd)
target_include_directories(${TARGET} PRIVATE ${CMAKE_SOURCE_DIR})
target_link_libraries(${TARGET} PRIVATE common mtmd ${CMAKE_THREAD_LIBS_INIT})

# New diffusion server target
add_executable(${TARGET_DIFFUSION} ${TARGET_DIFFUSION_SRCS})
install(TARGETS ${TARGET_DIFFUSION} RUNTIME)

target_include_directories(${TARGET_DIFFUSION} PRIVATE ../mtmd)
target_include_directories(${TARGET_DIFFUSION} PRIVATE ${CMAKE_SOURCE_DIR})
target_link_libraries(${TARGET_DIFFUSION} PRIVATE common mtmd ${CMAKE_THREAD_LIBS_INIT})

if (WIN32)
    TARGET_LINK_LIBRARIES(${TARGET} PRIVATE ws2_32)
    TARGET_LINK_LIBRARIES(${TARGET_DIFFUSION} PRIVATE ws2_32)
endif()

target_compile_features(${TARGET} PRIVATE cxx_std_17)
target_compile_features(${TARGET_DIFFUSION} PRIVATE cxx_std_17)