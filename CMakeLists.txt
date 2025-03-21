cmake_minimum_required(VERSION 3.10)
project(LoggingSystem)

# 设置测试输出目录的根目录
set(TEST_ROOT_DIR ${CMAKE_SOURCE_DIR}/test)

# 设置C++标准
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 设置编译选项
if(MSVC)
    add_compile_options(/utf-8)
    add_definitions(-D_WIN64 -D_AMD64_ -DWIN32 -D_WINDOWS)
endif()

# 主程序输出目录
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

# 查找依赖包
find_package(spdlog CONFIG REQUIRED)
find_package(nlohmann_json CONFIG REQUIRED)

# 添加库目标
add_library(logger
    src/Utils/Logger.cpp
    include/Utils/Logger.h
)

# 设置库的包含目录
target_include_directories(logger PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/include
)

# 链接依赖库
target_link_libraries(logger PUBLIC
    spdlog::spdlog
    nlohmann_json::nlohmann_json
)

# 添加示例程序（保持原输出目录）
add_executable(log_example examples/log_example.cpp)
target_link_libraries(log_example PRIVATE logger)

# 复制配置文件到输出目录
add_custom_command(TARGET log_example POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E make_directory "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/Release/config"
    COMMAND ${CMAKE_COMMAND} -E copy
        "${CMAKE_CURRENT_SOURCE_DIR}/config/logger_config.json"
        "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/Release/config/logger_config.json"
)

# 函数：添加模块测试
function(add_module_test MODULE_NAME TEST_SOURCES)
    # 设置模块测试输出目录
    set(MODULE_TEST_DIR ${TEST_ROOT_DIR}/${MODULE_NAME})
    
    # 添加测试可执行文件
    add_executable(${MODULE_NAME}_test ${TEST_SOURCES})
    
    # 链接必要的库
    target_link_libraries(${MODULE_NAME}_test PRIVATE logger)
    
    # 设置输出目录
    set_target_properties(${MODULE_NAME}_test PROPERTIES
        RUNTIME_OUTPUT_DIRECTORY ${MODULE_TEST_DIR}
        RUNTIME_OUTPUT_DIRECTORY_DEBUG ${MODULE_TEST_DIR}
        RUNTIME_OUTPUT_DIRECTORY_RELEASE ${MODULE_TEST_DIR}
    )
    
    # 创建输出目录
    add_custom_command(TARGET ${MODULE_NAME}_test POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E make_directory ${MODULE_TEST_DIR}
    )
    
    # 复制配置文件到测试目录
    add_custom_command(TARGET ${MODULE_NAME}_test POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy
            ${CMAKE_CURRENT_SOURCE_DIR}/config/logger_config.json
            ${MODULE_TEST_DIR}/logger_config.json
    )
endfunction()

# 添加日志模块测试
add_module_test(logger "tests/test_logger.cpp")

# 示例：如何添加其他模块的测试
# add_module_test(somemodule "tests/test_somemodule.cpp")