# Copyright 2020 The Shaderc Authors. All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

project(libshaderc)

# Even though shaderc.hpp is a headers-only library, adding
# a dependency here will force clients of the library to rebuild
# when it changes.
set(SHADERC_SOURCES
  include/shaderc/shaderc.h
  include/shaderc/shaderc.hpp
  src/shaderc.cc
  src/shaderc_private.h
)

add_library(shaderc STATIC ${SHADERC_SOURCES})
shaderc_default_compile_options(shaderc)
target_include_directories(shaderc
    PUBLIC include
    PRIVATE ${glslang_SOURCE_DIR}
            ${SPIRV-Headers_SOURCE_DIR}/include)

add_library(shaderc_shared SHARED ${SHADERC_SOURCES})
shaderc_default_compile_options(shaderc_shared)
target_include_directories(shaderc_shared
    PUBLIC include
    PRIVATE ${glslang_SOURCE_DIR}
            ${SPIRV-Headers_SOURCE_DIR}/include)
target_compile_definitions(shaderc_shared
    PRIVATE SHADERC_IMPLEMENTATION
    PUBLIC SHADERC_SHAREDLIB
)
set_target_properties(shaderc_shared PROPERTIES SOVERSION 1)

if(SHADERC_ENABLE_INSTALL)
  install(
    FILES
      include/shaderc/env.h
      include/shaderc/status.h
      include/shaderc/visibility.h
      include/shaderc/shaderc.h
      include/shaderc/shaderc.hpp
    DESTINATION
      ${CMAKE_INSTALL_INCLUDEDIR}/shaderc)

  install(TARGETS shaderc shaderc_shared
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    BUNDLE DESTINATION ${CMAKE_INSTALL_BINDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})
endif(SHADERC_ENABLE_INSTALL)

find_package(Threads)
set(SHADERC_LIBS
  glslang ${CMAKE_THREAD_LIBS_INIT}
  shaderc_util
  SPIRV # from glslang
  SPIRV-Tools
)

target_link_libraries(shaderc PRIVATE ${SHADERC_LIBS})
target_link_libraries(shaderc_shared PRIVATE ${SHADERC_LIBS})

shaderc_combine_static_lib(shaderc_combined shaderc)

if(SHADERC_ENABLE_INSTALL)
  install(TARGETS shaderc_combined DESTINATION ${CMAKE_INSTALL_LIBDIR})
endif(SHADERC_ENABLE_INSTALL)
