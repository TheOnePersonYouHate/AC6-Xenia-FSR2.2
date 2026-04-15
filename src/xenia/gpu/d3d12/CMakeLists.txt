add_library(xenia-gpu-d3d12 STATIC)
xe_platform_sources(xenia-gpu-d3d12 ${CMAKE_CURRENT_SOURCE_DIR})
xe_shader_rules_dxbc(xenia-gpu-d3d12 ${CMAKE_CURRENT_SOURCE_DIR}/../shaders)
target_link_libraries(xenia-gpu-d3d12 PUBLIC
  fmt xxhash xenia-base xenia-gpu xenia-ui xenia-ui-d3d12
)
xe_target_defaults(xenia-gpu-d3d12)

if(XENIA_BUILD_MISC)
  # D3D12 trace viewer
  add_executable(xenia-gpu-d3d12-trace-viewer WIN32
    ${CMAKE_CURRENT_SOURCE_DIR}/d3d12_trace_viewer_main.cc
    ${PROJECT_SOURCE_DIR}/src/xenia/ui/windowed_app_main_win.cc
  )
  target_link_libraries(xenia-gpu-d3d12-trace-viewer PRIVATE
    xenia-apu xenia-apu-nop xenia-base xenia-core xenia-cpu
    xenia-gpu xenia-gpu-d3d12 xenia-hid xenia-hid-nop
    xenia-kernel xenia-patcher xenia-ui xenia-ui-d3d12 xenia-vfs
    aes_128 capstone dxbc fmt imgui libavcodec libavutil mspack snappy xxhash
  )
  if(XE_TARGET_X86_64)
    target_link_libraries(xenia-gpu-d3d12-trace-viewer PRIVATE xenia-cpu-backend-x64)
  endif()
  xe_target_defaults(xenia-gpu-d3d12-trace-viewer)

  # D3D12 trace dump
  add_executable(xenia-gpu-d3d12-trace-dump
    ${CMAKE_CURRENT_SOURCE_DIR}/d3d12_trace_dump_main.cc
    ${PROJECT_SOURCE_DIR}/src/xenia/base/console_app_main_win.cc
  )
  target_link_libraries(xenia-gpu-d3d12-trace-dump PRIVATE
    xenia-apu xenia-apu-nop xenia-base xenia-core xenia-cpu
    xenia-gpu xenia-gpu-d3d12 xenia-hid xenia-hid-nop
    xenia-kernel xenia-patcher xenia-ui xenia-ui-d3d12 xenia-vfs
    aes_128 capstone dxbc fmt imgui libavcodec libavutil mspack snappy xxhash
  )
  if(XE_TARGET_X86_64)
    target_link_libraries(xenia-gpu-d3d12-trace-dump PRIVATE xenia-cpu-backend-x64)
  endif()
  xe_target_defaults(xenia-gpu-d3d12-trace-dump)
endif()

# === FSR2 2.2.1 INTEGRATION ===
set(FFX_FSR2_ROOT "${PROJECT_SOURCE_DIR}/third_party/FidelityFX-FSR2" CACHE PATH "Path to FSR2 v2.2.1")

if(EXISTS "${FFX_FSR2_ROOT}/src/ffx-fsr2-api")
  target_include_directories(xenia-gpu-d3d12 PUBLIC
    "${FFX_FSR2_ROOT}/src/ffx-fsr2-api"
    "${FFX_FSR2_ROOT}/src/ffx-fsr2-api/dx12"
  )

  if(EXISTS "${FFX_FSR2_ROOT}/bin/ffx_fsr2_api/ffx_fsr2_api_dx12_x64.lib")
    target_link_libraries(xenia-gpu-d3d12 PUBLIC
      "${FFX_FSR2_ROOT}/bin/ffx_fsr2_api/ffx_fsr2_api_dx12_x64.lib"
    )
    message(STATUS "FSR2 2.2.1: Linked prebuilt DX12 backend successfully")
  else()
    message(FATAL_ERROR 
      "FSR2 ERROR: Prebuilt library not found!\n"
      "Expected file: ${FFX_FSR2_ROOT}/bin/ffx_fsr2_api/ffx_fsr2_api_dx12_x64.lib\n\n"
      "The file should be there from the build you just did.")
  endif()
else()
  message(FATAL_ERROR 
    "FSR2 ERROR: SDK folder not found!\n"
    "Expected: ${FFX_FSR2_ROOT}")
endif()
# === END FSR2 INTEGRATION ===