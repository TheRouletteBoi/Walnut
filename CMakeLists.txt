# Walnut/CMakeLists.txt
# Handles: ImGui, GLFW, yaml-cpp, Walnut core, Walnut-Networking

# ==============================================================================
# Vendor: yaml-cpp (needed for both GUI and Headless)
# ==============================================================================
set(YAML_CPP_DIR ${CMAKE_CURRENT_SOURCE_DIR}/vendor/yaml-cpp)

file(GLOB_RECURSE YAML_CPP_SOURCES
    ${YAML_CPP_DIR}/src/*.h
    ${YAML_CPP_DIR}/src/*.cpp
    ${YAML_CPP_DIR}/include/*.h
)

add_library(yaml-cpp STATIC ${YAML_CPP_SOURCES})
target_include_directories(yaml-cpp PUBLIC ${YAML_CPP_DIR}/include)
target_compile_definitions(yaml-cpp PUBLIC YAML_CPP_STATIC_DEFINE)
set_target_properties(yaml-cpp PROPERTIES
    CXX_STANDARD 17
    POSITION_INDEPENDENT_CODE ON
)

# ==============================================================================
# GUI-only dependencies (ImGui, GLFW)
# ==============================================================================
if(NOT WL_HEADLESS)

# ------------------------------------------------------------------------------
# Vendor: ImGui
# ------------------------------------------------------------------------------
set(IMGUI_DIR ${CMAKE_CURRENT_SOURCE_DIR}/vendor/imgui)

add_library(ImGui STATIC
    ${IMGUI_DIR}/imconfig.h
    ${IMGUI_DIR}/imgui.h
    ${IMGUI_DIR}/imgui.cpp
    ${IMGUI_DIR}/imgui_draw.cpp
    ${IMGUI_DIR}/imgui_internal.h
    ${IMGUI_DIR}/imgui_tables.cpp
    ${IMGUI_DIR}/imgui_widgets.cpp
    ${IMGUI_DIR}/imstb_rectpack.h
    ${IMGUI_DIR}/imstb_textedit.h
    ${IMGUI_DIR}/imstb_truetype.h
    ${IMGUI_DIR}/imgui_demo.cpp
)

target_include_directories(ImGui PUBLIC ${IMGUI_DIR})
set_target_properties(ImGui PROPERTIES
    CXX_STANDARD 17
    POSITION_INDEPENDENT_CODE ON
)

# ------------------------------------------------------------------------------
# Vendor: GLFW
# ------------------------------------------------------------------------------
set(GLFW_DIR ${CMAKE_CURRENT_SOURCE_DIR}/vendor/glfw)

set(GLFW_COMMON_SOURCES
    ${GLFW_DIR}/src/internal.h
    ${GLFW_DIR}/src/platform.h
    ${GLFW_DIR}/src/mappings.h
    ${GLFW_DIR}/src/context.c
    ${GLFW_DIR}/src/init.c
    ${GLFW_DIR}/src/input.c
    ${GLFW_DIR}/src/monitor.c
    ${GLFW_DIR}/src/platform.c
    ${GLFW_DIR}/src/vulkan.c
    ${GLFW_DIR}/src/window.c
    ${GLFW_DIR}/src/egl_context.c
    ${GLFW_DIR}/src/osmesa_context.c
    ${GLFW_DIR}/src/null_init.c
    ${GLFW_DIR}/src/null_joystick.c
    ${GLFW_DIR}/src/null_joystick.h
    ${GLFW_DIR}/src/null_monitor.c
    ${GLFW_DIR}/src/null_platform.h
    ${GLFW_DIR}/src/null_window.c
)

if(APPLE)
    set(GLFW_PLATFORM_SOURCES
        ${GLFW_DIR}/src/cocoa_platform.h
        ${GLFW_DIR}/src/cocoa_joystick.h
        ${GLFW_DIR}/src/cocoa_init.m
        ${GLFW_DIR}/src/cocoa_joystick.m
        ${GLFW_DIR}/src/cocoa_monitor.m
        ${GLFW_DIR}/src/cocoa_window.m
        ${GLFW_DIR}/src/cocoa_time.c
        ${GLFW_DIR}/src/cocoa_time.h
        ${GLFW_DIR}/src/nsgl_context.m
        ${GLFW_DIR}/src/posix_thread.c
        ${GLFW_DIR}/src/posix_thread.h
        ${GLFW_DIR}/src/posix_module.c
    )
    set(GLFW_PLATFORM_DEFINES _GLFW_COCOA)
elseif(WIN32)
    set(GLFW_PLATFORM_SOURCES
        ${GLFW_DIR}/src/win32_platform.h
        ${GLFW_DIR}/src/win32_joystick.h
        ${GLFW_DIR}/src/win32_init.c
        ${GLFW_DIR}/src/win32_joystick.c
        ${GLFW_DIR}/src/win32_module.c
        ${GLFW_DIR}/src/win32_monitor.c
        ${GLFW_DIR}/src/win32_time.c
        ${GLFW_DIR}/src/win32_time.h
        ${GLFW_DIR}/src/win32_thread.c
        ${GLFW_DIR}/src/win32_thread.h
        ${GLFW_DIR}/src/win32_window.c
        ${GLFW_DIR}/src/wgl_context.c
    )
    set(GLFW_PLATFORM_DEFINES _GLFW_WIN32 _CRT_SECURE_NO_WARNINGS)
elseif(UNIX)
    set(GLFW_PLATFORM_SOURCES
        ${GLFW_DIR}/src/x11_platform.h
        ${GLFW_DIR}/src/xkb_unicode.h
        ${GLFW_DIR}/src/linux_joystick.h
        ${GLFW_DIR}/src/posix_time.h
        ${GLFW_DIR}/src/posix_thread.h
        ${GLFW_DIR}/src/x11_init.c
        ${GLFW_DIR}/src/x11_monitor.c
        ${GLFW_DIR}/src/x11_window.c
        ${GLFW_DIR}/src/xkb_unicode.c
        ${GLFW_DIR}/src/posix_module.c
        ${GLFW_DIR}/src/posix_time.c
        ${GLFW_DIR}/src/posix_thread.c
        ${GLFW_DIR}/src/glx_context.c
        ${GLFW_DIR}/src/linux_joystick.c
    )
    set(GLFW_PLATFORM_DEFINES _GLFW_X11)
endif()

add_library(GLFW STATIC ${GLFW_COMMON_SOURCES} ${GLFW_PLATFORM_SOURCES})
target_include_directories(GLFW PUBLIC ${GLFW_DIR}/include)
target_compile_definitions(GLFW PRIVATE ${GLFW_PLATFORM_DEFINES})
set_target_properties(GLFW PROPERTIES
    POSITION_INDEPENDENT_CODE ON
)

if(APPLE)
    target_link_libraries(GLFW PRIVATE
        "-framework Cocoa"
        "-framework IOKit"
        "-framework CoreFoundation"
    )
endif()

endif() # NOT WL_HEADLESS

# ==============================================================================
# Walnut Core Library
# ==============================================================================
set(WALNUT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/Walnut)

# Common sources (used by both GUI and Headless)
file(GLOB_RECURSE WALNUT_COMMON_SOURCES
    ${WALNUT_DIR}/Source/*.h
    ${WALNUT_DIR}/Source/*.cpp
)

if(WL_HEADLESS)
    # Headless platform sources
    file(GLOB_RECURSE WALNUT_PLATFORM_SOURCES
        ${WALNUT_DIR}/Platform/Headless/*.h
        ${WALNUT_DIR}/Platform/Headless/*.cpp
    )
    
    add_library(Walnut-Headless STATIC ${WALNUT_COMMON_SOURCES} ${WALNUT_PLATFORM_SOURCES})
    
    target_include_directories(Walnut-Headless PUBLIC
        ${WALNUT_DIR}/Source
        ${WALNUT_DIR}/Platform/Headless
        ${CMAKE_CURRENT_SOURCE_DIR}/vendor/glm
        ${CMAKE_CURRENT_SOURCE_DIR}/vendor/spdlog/include
    )
    
    target_compile_definitions(Walnut-Headless PUBLIC WL_HEADLESS)
    
    # Create alias for unified usage
    add_library(Walnut::Core ALIAS Walnut-Headless)
    
else()
    # GUI platform sources
    file(GLOB_RECURSE WALNUT_PLATFORM_SOURCES
        ${WALNUT_DIR}/Platform/GUI/*.h
        ${WALNUT_DIR}/Platform/GUI/*.cpp
    )
    
    add_library(Walnut STATIC ${WALNUT_COMMON_SOURCES} ${WALNUT_PLATFORM_SOURCES})
    
    target_include_directories(Walnut PUBLIC
        ${WALNUT_DIR}/Source
        ${WALNUT_DIR}/Platform/GUI
        ${CMAKE_CURRENT_SOURCE_DIR}/vendor/imgui
        ${CMAKE_CURRENT_SOURCE_DIR}/vendor/glfw/include
        ${CMAKE_CURRENT_SOURCE_DIR}/vendor/stb_image
        ${CMAKE_CURRENT_SOURCE_DIR}/vendor/glm
        ${CMAKE_CURRENT_SOURCE_DIR}/vendor/spdlog/include
    )
    
    target_link_libraries(Walnut PUBLIC
        ImGui
        GLFW
        Vulkan::Vulkan
    )
    
    if(APPLE)
        target_link_libraries(Walnut PUBLIC
            "-framework Cocoa"
            "-framework IOKit"
            "-framework CoreFoundation"
        )
    endif()
    
    # Create alias for unified usage
    add_library(Walnut::Core ALIAS Walnut)
    
endif()

# ==============================================================================
# Walnut-Networking Module
# ==============================================================================
set(WALNUT_NET_DIR ${CMAKE_CURRENT_SOURCE_DIR}/Walnut-Modules/Walnut-Networking)

file(GLOB WALNUT_NETWORKING_SOURCES
        ${WALNUT_NET_DIR}/Source/*.h
        ${WALNUT_NET_DIR}/Source/*.cpp
        ${WALNUT_NET_DIR}/Source/Walnut/Networking/*.h
        ${WALNUT_NET_DIR}/Source/Walnut/Networking/*.cpp
)

# Platform-specific source glob only (no linking yet)
if(APPLE)
    file(GLOB_RECURSE WALNUT_NETWORKING_PLATFORM_SOURCES
            ${WALNUT_NET_DIR}/Platform/MacOS/*.h
            ${WALNUT_NET_DIR}/Platform/MacOS/*.cpp
    )
    set(GNS_LIB_DIR ${WALNUT_NET_DIR}/vendor/GameNetworkingSockets/bin/Macos)
elseif(WIN32)
    file(GLOB_RECURSE WALNUT_NETWORKING_PLATFORM_SOURCES
            ${WALNUT_NET_DIR}/Platform/Windows/*.h
            ${WALNUT_NET_DIR}/Platform/Windows/*.cpp
    )
    set(GNS_LIB_DIR ${WALNUT_NET_DIR}/vendor/GameNetworkingSockets/bin/Windows)
elseif(UNIX)
    file(GLOB_RECURSE WALNUT_NETWORKING_PLATFORM_SOURCES
            ${WALNUT_NET_DIR}/Platform/Linux/*.h
            ${WALNUT_NET_DIR}/Platform/Linux/*.cpp
    )
    set(GNS_LIB_DIR ${WALNUT_NET_DIR}/vendor/GameNetworkingSockets/bin/Linux)
endif()

# Target must be created before any target_* calls
add_library(Walnut-Networking STATIC
        ${WALNUT_NETWORKING_SOURCES}
        ${WALNUT_NETWORKING_PLATFORM_SOURCES}
)

target_include_directories(Walnut-Networking PUBLIC
        ${WALNUT_NET_DIR}/Source
        ${WALNUT_NET_DIR}/vendor/GameNetworkingSockets/include
        ${CMAKE_CURRENT_SOURCE_DIR}/Walnut/Source
        ${CMAKE_CURRENT_SOURCE_DIR}/vendor/glm
        ${CMAKE_CURRENT_SOURCE_DIR}/vendor/spdlog/include
)

if(NOT WL_HEADLESS)
    target_include_directories(Walnut-Networking PUBLIC
            ${CMAKE_CURRENT_SOURCE_DIR}/vendor/imgui
            ${CMAKE_CURRENT_SOURCE_DIR}/vendor/glfw/include
    )
endif()

# Platform-specific linking
if(APPLE)
    target_include_directories(Walnut-Networking PRIVATE
            ${WALNUT_NET_DIR}/Platform/MacOS
    )

    function(add_static_lib NAME)
        add_library(${NAME} STATIC IMPORTED)
        set_target_properties(${NAME} PROPERTIES
                IMPORTED_LOCATION ${GNS_LIB_DIR}/lib${NAME}.a
        )
    endfunction()

    add_static_lib(GameNetworkingSockets_s)
    add_static_lib(ssl)
    add_static_lib(crypto)
    add_static_lib(protobuf)
    add_static_lib(protobuf-lite)
    add_static_lib(upb)
    add_static_lib(utf8_range)
    add_static_lib(utf8_validity)

    foreach(ABSL_LIB
            absl_base absl_borrowed_fixup_buffer absl_city absl_civil_time
            absl_cord_internal absl_cord absl_cordz_functions absl_cordz_handle
            absl_cordz_info absl_cordz_sample_token absl_crc_cord_state
            absl_crc_cpu_detect absl_crc_internal absl_crc32c
            absl_debugging_internal absl_decode_rust_punycode absl_demangle_internal
            absl_demangle_rust absl_die_if_null absl_examine_stack
            absl_exponential_biased absl_failure_signal_handler
            absl_flags_commandlineflag_internal absl_flags_commandlineflag
            absl_flags_config absl_flags_internal absl_flags_marshalling
            absl_flags_parse absl_flags_private_handle_accessor
            absl_flags_program_name absl_flags_reflection absl_flags_usage_internal
            absl_flags_usage absl_generic_printer_internal absl_graphcycles_internal
            absl_hash absl_hashtable_profiler absl_hashtablez_sampler absl_int128
            absl_kernel_timeout_internal absl_leak_check absl_log_entry
            absl_log_flags absl_log_globals absl_log_initialize
            absl_log_internal_check_op absl_log_internal_conditions
            absl_log_internal_fnmatch absl_log_internal_format
            absl_log_internal_globals absl_log_internal_log_sink_set
            absl_log_internal_message absl_log_internal_nullguard
            absl_log_internal_proto absl_log_internal_structured_proto
            absl_log_severity absl_log_sink absl_malloc_internal
            absl_periodic_sampler absl_poison absl_profile_builder
            absl_random_distributions absl_random_internal_distribution_test_util
            absl_random_internal_entropy_pool absl_random_internal_platform
            absl_random_internal_randen_hwaes_impl absl_random_internal_randen_hwaes
            absl_random_internal_randen_slow absl_random_internal_randen
            absl_random_internal_seed_material absl_random_seed_gen_exception
            absl_random_seed_sequences absl_raw_hash_set absl_raw_logging_internal
            absl_scoped_set_env absl_spinlock_wait absl_stacktrace absl_status
            absl_statusor absl_str_format_internal absl_strerror
            absl_strings_internal absl_strings absl_symbolize absl_synchronization
            absl_throw_delegate absl_time_zone absl_time absl_tracing_internal
            absl_utf8_for_code_point absl_vlog_config_internal
    )
        add_static_lib(${ABSL_LIB})
    endforeach()

    target_link_libraries(Walnut-Networking PUBLIC
            GameNetworkingSockets_s
            ssl crypto
            protobuf protobuf-lite upb utf8_range utf8_validity
            absl_base absl_strings absl_strings_internal absl_str_format_internal
            absl_synchronization absl_time absl_time_zone absl_civil_time
            absl_hash absl_city absl_int128 absl_raw_hash_set
            absl_hashtablez_sampler absl_hashtable_profiler
            absl_status absl_statusor absl_log_entry absl_log_globals
            absl_log_initialize absl_log_internal_message absl_log_internal_format
            absl_log_internal_globals absl_log_internal_log_sink_set
            absl_log_internal_check_op absl_log_internal_conditions
            absl_log_internal_nullguard absl_log_internal_proto
            absl_log_internal_structured_proto absl_log_severity absl_log_sink
            absl_log_flags absl_flags_parse absl_flags_config absl_flags_internal
            absl_flags_commandlineflag absl_flags_commandlineflag_internal
            absl_flags_marshalling absl_flags_reflection absl_flags_program_name
            absl_flags_usage absl_flags_usage_internal
            absl_flags_private_handle_accessor absl_raw_logging_internal
            absl_spinlock_wait absl_malloc_internal absl_throw_delegate
            absl_exponential_biased absl_graphcycles_internal
            absl_kernel_timeout_internal absl_cord absl_cord_internal
            absl_cordz_functions absl_cordz_handle absl_cordz_info
            absl_cordz_sample_token absl_crc32c absl_crc_internal
            absl_crc_cpu_detect absl_crc_cord_state absl_random_distributions
            absl_random_internal_randen absl_random_internal_randen_hwaes
            absl_random_internal_randen_hwaes_impl absl_random_internal_randen_slow
            absl_random_internal_seed_material absl_random_internal_platform
            absl_random_internal_entropy_pool absl_random_seed_sequences
            absl_random_seed_gen_exception absl_random_internal_distribution_test_util
            absl_stacktrace absl_symbolize absl_debugging_internal
            absl_demangle_internal absl_demangle_rust absl_decode_rust_punycode
            absl_utf8_for_code_point absl_examine_stack absl_failure_signal_handler
            absl_leak_check absl_die_if_null absl_strerror absl_periodic_sampler
            absl_scoped_set_env absl_tracing_internal absl_profile_builder
            absl_generic_printer_internal absl_poison absl_borrowed_fixup_buffer
            absl_vlog_config_internal
            "-framework Security"
            "-framework CoreFoundation"
    )

elseif(WIN32)
    target_include_directories(Walnut-Networking PRIVATE
            ${WALNUT_NET_DIR}/Platform/Windows
    )
    target_link_directories(Walnut-Networking PUBLIC ${GNS_LIB_DIR})
    target_link_libraries(Walnut-Networking PUBLIC
            Ws2_32
            GameNetworkingSockets
    )

elseif(UNIX)
    target_include_directories(Walnut-Networking PRIVATE
            ${WALNUT_NET_DIR}/Platform/Linux
    )
    target_link_directories(Walnut-Networking PUBLIC ${GNS_LIB_DIR})
    target_link_libraries(Walnut-Networking PUBLIC GameNetworkingSockets)
endif()