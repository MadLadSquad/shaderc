# Suppress all warnings from third-party projects.
set_property(DIRECTORY APPEND PROPERTY COMPILE_OPTIONS -w)

set(SHADERC_THIRD_PARTY_ROOT_DIR "${CMAKE_CURRENT_SOURCE_DIR}" CACHE STRING
  "Root location of all third_party projects")
set(SHADERC_SPIRV_TOOLS_DIR "${SHADERC_THIRD_PARTY_ROOT_DIR}/spirv-tools" CACHE STRING
  "Location of spirv-tools source")
set(SHADERC_SPIRV_HEADERS_DIR "${SHADERC_THIRD_PARTY_ROOT_DIR}/spirv-headers" CACHE STRING
  "Location of spirv-headers source")
set(SHADERC_GLSLANG_DIR "${SHADERC_THIRD_PARTY_ROOT_DIR}/glslang" CACHE STRING
  "Location of glslang source")
set(SHADERC_EFFCEE_DIR "${SHADERC_THIRD_PARTY_ROOT_DIR}/effcee" CACHE STRING
  "Location of effcee source")
set(SHADERC_RE2_DIR "${SHADERC_THIRD_PARTY_ROOT_DIR}/re2" CACHE STRING
  "Location of re2 source")
set(SHADERC_TINT_DIR "${SHADERC_THIRD_PARTY_ROOT_DIR}/tint" CACHE STRING
  "Location of tint source")
set(SHADERC_ABSL_DIR "${SHADERC_THIRD_PARTY_ROOT_DIR}/abseil_cpp" CACHE STRING
  "Location of re2 source")

set( SKIP_GLSLANG_INSTALL ${SHADERC_SKIP_INSTALL} )
set( SKIP_SPIRV_TOOLS_INSTALL ${SHADERC_SKIP_INSTALL} )

set(OLD_PLATFORM_TOOLSET ${CMAKE_GENERATOR_TOOLSET})
check_cxx_compiler_flag(-fPIC COMPILER_SUPPORTS_PIC)


if (IS_DIRECTORY ${SHADERC_SPIRV_HEADERS_DIR})
  set(SPIRV_HEADERS_SKIP_EXAMPLES OFF)
  add_subdirectory(${SHADERC_SPIRV_HEADERS_DIR} spirv-headers)
endif()

if (NOT TARGET SPIRV-Tools)
  # Check SPIRV-Tools before glslang so that it is linked into glslang.
  # we control optimizations via glslang API calls directly.
  if (IS_DIRECTORY ${SHADERC_SPIRV_TOOLS_DIR})
    if ("${SHADERC_SKIP_TESTS}")
      # Also skip building tests in SPIRV-Tools.
      set(SPIRV_SKIP_TESTS ON CACHE BOOL "Skip building SPIRV-Tools tests")
    elseif(NOT "${SPIRV_SKIP_TESTS}")
    endif()
    add_subdirectory(${SHADERC_SPIRV_TOOLS_DIR} spirv-tools)
    if (NOT "${SPIRV_SKIP_TESTS}")
      if (MSVC)
        if (${MSVC_VERSION} LESS 1920)
	  # VS 2017 requires /bigobj on test_opt
	  # https://github.com/google/shaderc/issues/1345
	  # https://github.com/KhronosGroup/SPIRV-Tools/issues/5335
	  target_compile_options(test_opt PRIVATE /bigobj)
        endif()
      endif()
    endif()
  endif()
  if (NOT TARGET SPIRV-Tools)
    message(FATAL_ERROR "SPIRV-Tools was not found - required for compilation")
  endif()
endif()

if (NOT TARGET glslang)
  if (IS_DIRECTORY ${SHADERC_GLSLANG_DIR})
    if (SHADERC_ENABLE_TESTS)
      # Glslang tests are off by default. Turn them on if testing Shaderc.
      set(GLSLANG_TESTS ON)
    endif()
    set(GLSLANG_ENABLE_INSTALL $<NOT:${SKIP_GLSLANG_INSTALL}>)
    add_subdirectory(${SHADERC_GLSLANG_DIR} glslang)
  endif()
  if (NOT TARGET glslang)
    message(FATAL_ERROR "glslang was not found - required for compilation")
  endif()
  if(WIN32)
      # This is unfortunate but glslang forces our
      # platform toolset to be v110, which we may not even have
      # installed, undo anything glslang has done to it.
      set(CMAKE_GENERATOR_TOOLSET "${OLD_PLATFORM_TOOLSET}" CACHE STRING
        "Platform Toolset" FORCE)
  endif()
endif()

if (SHADERC_ENABLE_WGSL_OUTPUT)
  # Use Google Tint for WGSL output.
  if (NOT TARGET libtint)
    if (IS_DIRECTORY ${SHADERC_TINT_DIR})
      if (NOT IS_DIRECTORY ${SHADERC_TINT_DIR}/out/docs)
        # The Tint Doxygen configuration assumes it can write to the out/docs
        # source directory.
        message(STATUS "Tint source directory out/docs does not exist. Disabling doc generation")
        set(TINT_BUILD_DOCS OFF)
      endif()
      set(TINT_BUILD_SPV_READER ON CACHE BOOL "Built Tint SPIR-V reader" FORCE)
      set(TINT_BUILD_WGSL_WRITER ON CACHE BOOL "Build Tint WGSL writer" FORCE)
      add_subdirectory(${SHADERC_TINT_DIR} tint)
    endif()
  endif()
  if (NOT TARGET libtint)
    message(FATAL_ERROR "Tint was not found - required for WGSL output")
  endif()
endif (SHADERC_ENABLE_WGSL_OUTPUT)
