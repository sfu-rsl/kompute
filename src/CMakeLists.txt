# SPDX-License-Identifier: Apache-2.0

cmake_minimum_required(VERSION 3.20)

if(KOMPUTE_OPT_ANDROID_BUILD)
    find_library(android android)
endif()

cmake_minimum_required(VERSION 3.20)

add_library(kompute Algorithm.cpp
    Manager.cpp
    OpAlgoDispatch.cpp
    OpMemoryBarrier.cpp
    OpTensorCopy.cpp
    OpTensorSyncDevice.cpp
    OpTensorSyncLocal.cpp
    Sequence.cpp
    Tensor.cpp
    Core.cpp)

add_library(kompute::kompute ALIAS kompute)

# Set version for shared libraries.
set_target_properties(kompute
    PROPERTIES
    VERSION ${${PROJECT_NAME}_VERSION}
    SOVERSION ${${PROJECT_NAME}_VERSION_MAJOR})

# Import GNU common install directory variables
include(GNUInstallDirs)

install(TARGETS kompute
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})

# Include CMake helpers for package config files
# Follow this installation guideline: https://cmake.org/cmake/help/latest/manual/cmake-packages.7.html
include(CMakePackageConfigHelpers)

configure_package_config_file(${PROJECT_SOURCE_DIR}/cmake/komputeConfig.cmake.in
    "${PROJECT_BINARY_DIR}/kompute/komputeConfig.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/kompute)

install(FILES ${PROJECT_BINARY_DIR}/kompute/komputeConfig.cmake
    ${PROJECT_BINARY_DIR}/kompute/komputeConfigVersion.cmake DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/kompute)

# VMA
# add_subdirectory(${PROJECT_SOURCE_DIR}/external/VulkanMemoryAllocator {CMAKE_CURRENT_BINARY_DIR}/kompute_vma)
# target_link_libraries(kompute VulkanMemoryAllocator)
target_include_directories(kompute PUBLIC ${PROJECT_SOURCE_DIR}/external/VulkanMemoryAllocator/include)


# ####################################################
# Linking
# ####################################################
if(KOMPUTE_OPT_ANDROID_BUILD)
    target_link_libraries(kompute PUBLIC vulkanAndroid
        android
        kp_logger
        kp_shader
        fmt::fmt)
else()
    target_link_libraries(kompute PUBLIC Vulkan::Vulkan
        kp_logger
        kp_shader
        fmt::fmt)
endif()

if(KOMPUTE_OPT_BUILD_PYTHON)
    include_directories(${PYTHON_INCLUDE_DIRS})

    target_link_libraries(kompute PRIVATE pybind11::headers ${PYTHON_LIBRARIES})
endif()

if(KOMPUTE_OPT_USE_BUILT_IN_VULKAN_HEADER)
    target_link_libraries(kompute PUBLIC Vulkan-Headers)
endif()

# ####################################################
# Misc
# ####################################################
add_subdirectory(logger)
add_subdirectory(shaders)
add_subdirectory(include)
