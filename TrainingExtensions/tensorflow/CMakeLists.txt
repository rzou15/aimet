#==============================================================================
#  @@-COPYRIGHT-START-@@
#
#  Copyright (c) 2018, Qualcomm Innovation Center, Inc. All rights reserved.
#
#  Redistribution and use in source and binary forms, with or without
#  modification, are permitted provided that the following conditions are met:
#
#  1. Redistributions of source code must retain the above copyright notice,
#     this list of conditions and the following disclaimer.
#
#  2. Redistributions in binary form must reproduce the above copyright notice,
#     this list of conditions and the following disclaimer in the documentation
#     and/or other materials provided with the distribution.
#
#  3. Neither the name of the copyright holder nor the names of its contributors
#     may be used to endorse or promote products derived from this software
#     without specific prior written permission.
#
#  THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
#  AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
#  IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
#  ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
#  LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
#  CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
#  SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
#  INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
#  CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
#  ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
#  POSSIBILITY OF SUCH DAMAGE.
#
#  SPDX-License-Identifier: BSD-3-Clause
#
#  @@-COPYRIGHT-END-@@
#==============================================================================
cmake_minimum_required(VERSION 3.5)

add_library(TrainingExtensionsTf SHARED
        src/qc_quantize_op_deprecated.cpp
        src/qc_quantize_op_deprecated.hpp
        src/AimetOpUtils.h
        src/AimetOpUtils.cpp
        src/qc_quantize_op.hpp
        src/qc_quantize_op.cpp
        )

target_compile_options(TrainingExtensionsTf
      PRIVATE
         -DGOOGLE_CUDA=1
      )

target_include_directories(TrainingExtensionsTf PRIVATE
      /usr/local/lib/python3.6/dist-packages/tensorflow_core/include
      /usr/local/lib/python3.6/dist-packages/tensorflow_core/include/external/nsync/public
      ${CMAKE_CURRENT_SOURCE_DIR}/../common/include
      )

target_link_libraries(TrainingExtensionsTf PUBLIC
      MoDlCompression
      MoDlCompressionCuda
      MoDlQuantization
      MoDlQuantizationCuda
      TensorFlow
      TrainingExtensionsTfCuda
      )

set_target_properties(TrainingExtensionsTf PROPERTIES
      OUTPUT_NAME "aimet_tf_ops"
      LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/artifacts"
      )

set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS}
      -DGOOGLE_CUDA=1
      -I /usr/local/lib/python3.6/dist-packages/tensorflow_core/include
      --expt-relaxed-constexpr
      )

get_target_property(dirs MoDlQuantization INCLUDE_DIRECTORIES)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/../common/include /usr/local/lib/python3.6/dist-packages/tensorflow_core/include/external/nsync/public ${dirs})

add_library(TrainingExtensionsTfCuda
        src/qc_quantize_op_gpu_deprecated.cu
        src/AimetOpUtilsGpu.cu
      )

target_compile_options(TrainingExtensionsTfCuda
      PRIVATE
         -Xcompiler -fPIC -D_GLIBCXX_USE_CXX11_ABI=0 -DGOOGLE_CUDA=1 -I /usr/local/lib/python3.6/dist-packages/tensorflow_core/include --expt-relaxed-constexpr
      )

install(FILES $<TARGET_FILE:TrainingExtensionsTf>
      DESTINATION ${AIMET_INSTALL_DIR}/lib/x86_64-linux-gnu
      )

add_subdirectory(src/python)
add_subdirectory(test)


