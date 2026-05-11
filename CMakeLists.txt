cmake_minimum_required(VERSION 3.16)
project(tlozoos C CXX)

include(CheckIPOSupported)
include(FetchContent)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE MinSizeRel)
endif()
option(GBRECOMP_SHRINK "Aggressive size shrink (dead-strip + symbol strip)" ON)
if(CMAKE_C_COMPILER_ID MATCHES "GNU|Clang")
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        add_compile_options(-O0 -g)
    elseif(CMAKE_BUILD_TYPE STREQUAL "Release")
        add_compile_options(-O3)
    endif()
    if(GBRECOMP_SHRINK)
        add_compile_options(-ffunction-sections -fdata-sections)
        add_link_options(-Wl,--gc-sections)
        if(NOT CMAKE_BUILD_TYPE STREQUAL "Debug")
            add_link_options(-s)
        endif()
    endif()
endif()

# ---------------------------------------------------------------------------
# gb-recompiled runtime
# ---------------------------------------------------------------------------
if(NOT TARGET gbrt)
    set(GBRT_REF "main" CACHE STRING "git ref of GB-Recomp/gb-recompiled to use")
    FetchContent_Declare(gb_recompiled
        GIT_REPOSITORY https://github.com/GB-Recomp/gb-recompiled.git
        GIT_TAG ${GBRT_REF}
        GIT_SHALLOW TRUE
    )
    FetchContent_GetProperties(gb_recompiled)
    if(NOT gb_recompiled_POPULATED)
        FetchContent_Populate(gb_recompiled)
    endif()
    add_subdirectory(${gb_recompiled_SOURCE_DIR}/runtime ${CMAKE_BINARY_DIR}/_gbrt_build)
endif()

# ---------------------------------------------------------------------------
# Cart sources
# ---------------------------------------------------------------------------
file(GLOB TLOZOOS_SOURCES CONFIGURE_DEPENDS
    "${CMAKE_CURRENT_SOURCE_DIR}/tlozoos.c"
    "${CMAKE_CURRENT_SOURCE_DIR}/tlozoos_main.c"
    "${CMAKE_CURRENT_SOURCE_DIR}/tlozoos_rom.c"
    "${CMAKE_CURRENT_SOURCE_DIR}/tlozoos_dispatch_chunk_*.c"
    "${CMAKE_CURRENT_SOURCE_DIR}/tlozoos_funcs_*.c"
)

set(GBRECOMP_GENERATED_OPT_LEVEL "1" CACHE STRING "Optimization level for generated ROM source files")
set_source_files_properties(${TLOZOOS_SOURCES} PROPERTIES COMPILE_OPTIONS "-O${GBRECOMP_GENERATED_OPT_LEVEL}")

# ---------------------------------------------------------------------------
# tlozoos_cart: static library so umbrella compilations can link us in.
# ---------------------------------------------------------------------------
add_library(tlozoos_cart STATIC ${TLOZOOS_SOURCES})
target_include_directories(tlozoos_cart PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
target_link_libraries(tlozoos_cart PUBLIC gbrt)

# ---------------------------------------------------------------------------
# Standalone launcher executable.
# ---------------------------------------------------------------------------
if(CMAKE_PROJECT_NAME STREQUAL "tlozoos")
    add_executable(tlozoos_launcher launcher_main.cpp)
    target_link_libraries(tlozoos_launcher PRIVATE tlozoos_cart gbrt)
    set_target_properties(tlozoos_launcher PROPERTIES OUTPUT_NAME tlozoos)
endif()
