#[[
杨小兵-2024-09-20
  设置构建当前cmakelists.txt的cmake版本最低是3.5
]]
cmake_minimum_required(VERSION 3.5)


#[[
杨小兵-2024-09-20
  CMake 策略 `CMP0077` ：这个策略的设置影响了如何处理 `option()` 命令和变量，特别是当一个 CMake 项目被作为子项目包含在另一个项目中时。
在实际应用中，这个策略允许更灵活地管理多项目构建配置，使顶层项目能够控制子项目的配置选项，而无需修改子项目本身的 CMake 脚本。这在大型项目
或多个项目依赖相同库的情况下尤其有用，可以统一配置策略，避免重复设置问题。通过在你的 CMake 脚本中设置 `CMP0077` 策略为 `NEW`，你可以确保
项目的配置更加一致和可控。

### CMake 策略 CMP0077 详解
- **策略描述**：在 CMake 中，策略 `CMP0077` 允许 `option()` 命令在项目被作为子项目包含时使用已经设置的变量值。这意味着，如果在包含子项
目之前某个选项变量已经被设置，那么 `option()` 命令不会重置这个变量的值。
- **旧行为**（OLD）：如果使用 `option()` 命令定义了一个选项，并且在包含该子项目之前已经设置了这个选项的值，那么原来的值会被 `option()` 
命令中指定的默认值覆盖。
- **新行为**（NEW）：`option()` 命令会保留现有的值，即使这个值是在包含该子项目之前设置的。这意味着外部项目可以预先设置一些选项，这些选项
将被保留下来，不会被子项目中的默认设置覆盖。
]]
# See https://cmake.org/cmake/help/latest/policy/CMP0077.html
# This allows for setting option variables externally, when this project
# is included in another CMake project.
cmake_policy(SET CMP0077 NEW)


#[[
杨小兵-2024-09-20
  设置当前项目名称为inja，inja这个项目使用的语言是C++，当前inja的版本信息是3.4.0
]]
project(inja LANGUAGES CXX VERSION 3.4.0)

#[[
杨小兵-2024-09-20
  下列是一些使用 `option()` 命令定义的 CMake 选项。这些选项用于在 CMake 配置过程中设置不同的构建参数和功能开关。每个 `option()` 命令定义了一个
布尔选项，可以通过 `-D` 标志在 CMake 命令行中设置，或者通过 CMake GUI 工具进行配置。下面是对每个选项的详细解释：

1. **INJA_USE_EMBEDDED_JSON**:
   - 描述：如果系统上不可用，则使用随附的 JSON 头文件。
   - 默认值：`ON`
   - 作用：此选项允许用户决定是否使用 Inja 库附带的 `nlohmann/json.hpp`。如果设置为 `ON`，即使系统上安装了 nlohmann JSON 库，Inja 构建系统也会
   使用内嵌的版本。这对于确保版本兼容性或在缺少系统级库安装权限的环境中特别有用。

2. **INJA_INSTALL**:
   - 描述：为 inja 生成安装目标。
   - 默认值：`ON`
   - 作用：此选项控制是否创建安装目标（即安装命令），使得 Inja 库可以被安装到系统的标准位置。这对于希望将 Inja 作为系统库使用的用户来说很有用。

3. **INJA_EXPORT**:
   - 描述：将当前构建树导出到包注册表。
   - 默认值：`ON`
   - 作用：启用此选项会将构建的 Inja 库及其相关文件导出到 CMake 包注册表中，这样其他项目就可以找到并链接到这个构建的库。

4. **BUILD_TESTING**:
   - 描述：构建单元测试。
   - 默认值：`ON`
   - 作用：此选项控制是否构建库的单元测试。对于开发者来说，这是一个检查库功能是否正常的重要工具。

5. **INJA_BUILD_TESTS**:
   - 描述：当 BUILD_TESTING 启用时，构建单元测试。
   - 默认值：`ON`
   - 作用：这个选项是对 `BUILD_TESTING` 的进一步细化，确保只有在测试构建被激活时才构建 Inja 的单元测试。

6. **BUILD_BENCHMARK**:
   - 描述：构建基准测试。
   - 默认值：`ON`
   - 作用：控制是否构建性能基准测试。这对于评估库的性能和进行性能优化非常重要。

7. **COVERALLS**:
   - 描述：生成 coveralls 数据。
   - 默认值：`OFF`
   - 作用：此选项用于控制是否生成适用于 [Coveralls](https://coveralls.io/) 的代码覆盖率数据。Coveralls 是一个在线服务，用于追踪项目的代码
   覆盖率变化和历史数据。

  这些选项使得构建过程具有很高的灵活性，允许开发者根据自己的需要开启或关闭特定的功能和测试。通过合理配置这些选项，可以确保库按预期方式构建，并且
可以根据项目需求进行适当的调整和优化。
]]
option(INJA_USE_EMBEDDED_JSON "Use the shipped json header if not available on the system" ON)
option(INJA_INSTALL "Generate install targets for inja" ON)
option(INJA_EXPORT "Export the current build tree to the package registry" ON)
option(BUILD_TESTING "Build unit tests" ON)
option(INJA_BUILD_TESTS "Build unit tests when BUILD_TESTING is enabled." ON)
option(BUILD_BENCHMARK "Build benchmark" ON)
option(COVERALLS "Generate coveralls data" OFF)


#[[
杨小兵-2024-09-20
  下列 CMake 脚本片段中包含了三个 `set` 命令，这些命令用于设置不同的变量，每个变量都对应于构建过程的不同方面。这里逐一解释每个命令的作用和意义：

1. **set(INJA_INSTALL_INCLUDE_DIR "include")**
   - **作用**：此命令设置了一个变量 `INJA_INSTALL_INCLUDE_DIR`，其值为 `"include"`。这个变量通常用于指定 Inja 库的头文件在安装时应放置的目录。
   - **详解**：在 CMake 的安装（`install()`）命令中，这个变量可以被用来指定头文件的安装路径。例如，可以通过这样的命令
    `install(DIRECTORY include/ DESTINATION ${INJA_INSTALL_INCLUDE_DIR})` 将库的公共头文件安装到系统的 `include` 目录下。

2. **set(INJA_PACKAGE_USE_EMBEDDED_JSON OFF)**
   - **作用**：此命令设置了一个变量 `INJA_PACKAGE_USE_EMBEDDED_JSON`，其值为 `OFF`。这意味着不使用 Inja 库自带的 JSON 头文件。
   - **详解**：如果 Inja 配置为不使用内嵌的 JSON 库，那么它将依赖于系统已安装的 `nlohmann/json.hpp` 库。这个设置通常用于确保项目使用的是最新
   版本的 JSON 库，或者当项目需要与系统上其他组件共享同一个 JSON 库版本时。

3. **set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR}/cmake)**
   - **作用**：此命令更新了 `CMAKE_MODULE_PATH` 变量，添加了一个新的路径 `${PROJECT_SOURCE_DIR}/cmake`。
   - **详解**：`CMAKE_MODULE_PATH` 变量指定了 CMake 查找模块（如 `find_package`, `include` 等命令使用的文件）的路径。通过添加 
   `${PROJECT_SOURCE_DIR}/cmake` 到这个列表，CMake 能够在这个目录下查找自定义的模块脚本或者配置文件。这通常用于组织项目特定的 CMake 模块，如
   查找依赖库、设置编译器选项等。

  这三个设置共同影响着项目的构建配置和行为，使得构建过程可以根据项目的具体需求进行定制。通过适当配置这些变量，开发者可以控制库的安装位置、依赖项
以及模块搜索路径，从而优化构建过程和项目结构。
]]
set(INJA_INSTALL_INCLUDE_DIR "include")
set(INJA_PACKAGE_USE_EMBEDDED_JSON OFF)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR}/cmake)


#[[
杨小兵-2024-09-20
  这段 CMake 脚本片段用于为使用 Microsoft Visual Studio 编译器（MSVC）构建项目时设置特定的编译选项。这里详细解释每个部分：

### if(MSVC)
  这行检查当前使用的编译器是否是 MSVC（Microsoft Visual Studio 的编译器）。`MSVC` 是 CMake 中预定义的变量，当 CMake 检测到项目正在使用
 Microsoft Visual Studio 的编译器进行构建时，该变量被设置为真（TRUE）。这个条件判断确保接下来的设置只在使用 MSVC 编译器时应用，避免对其他
 编译器产生影响。

### add_compile_options(/utf-8 /Zc:__cplusplus)
此命令为 MSVC 编译器添加了两个编译选项：
- **/utf-8**：这个选项确保所有源文件和所有执行阶段都使用 UTF-8 编码。这对于支持多语言或国际化的项目特别重要，确保字符编码的统一，避免因编码
不一致导致的问题。例如，如果源代码中包含非 ASCII 字符（如中文、日文等），使用此选项可以保证这些字符在编译过程中被正确处理。

- **/Zc:__cplusplus**：这个选项告诉 MSVC 编译器遵循更严格的 C++ 标准，正确定义 `__cplusplus` 预处理器宏。通常情况下，MSVC 的 `__cplusplus` 、
宏默认未按照 C++ 标准更新（直到 Visual Studio 2017 15.7 版本后才有所改善），此选项确保 `__cplusplus` 宏能够正确反映编译代码的 C++ 标准版本。
例如，如果使用 C++17 编译，`__cplusplus` 将被定义为 201703L。

### 适用场景

  这些编译器选项特别适用于需要确保编译环境和代码行为一致性的项目，比如跨平台项目或需要处理多种语言字符的项目。通过在 CMake 中指定这些选项，可以
提高代码的移植性和可维护性，同时确保在不同开发环境中的一致性表现。
]]
if(MSVC)
  add_compile_options(/utf-8 /Zc:__cplusplus)
endif()



#[[
杨小兵-2024-09-20
  下列 CMake 脚本片段涉及了两个主要命令：`add_library` 和 `add_library` 的别名版本。这些命令在 CMake 中用于定义库和设置库的别名。下面是对
这两条命令的详细解释：

### add_library(inja INTERFACE)
1. **命令功能**：
   - `add_library` 是 CMake 中用来声明一个库的命令。它可以创建静态库、动态库或者在本例中的接口库。
2. **INTERFACE 关键字**：
   - 使用 `INTERFACE` 关键字创建的是一个接口库。接口库不编译成一个实际的库文件，而是用于在项目中传递使用要求（例如编译选项、定义、包含目录等），
  对于 header-only 库特别有用。
3. **库名称**：
   - 在这个命令中，库的名称被定义为 `inja`。这意味着后续可以通过这个名称来引用这个库，例如在 `target_link_libraries` 命令中指定链接依赖。

### add_library(pantor::inja ALIAS inja)

1. **命令功能**：
   - 这条命令创建了一个别名 `pantor::inja` 指向原先声明的 `inja` 库。别名用于提供一个可选的命名空间风格的引用方式。

2. **ALIAS 关键字**：
   - `ALIAS` 关键字用于为一个已存在的库指定一个别名。这使得库可以有两个名字：原始名和别名。

3. **命名空间**：
   - 使用 `pantor::inja` 作为别名，`pantor` 可以看作是一个命名空间，这不仅有助于库的组织，也避免了名称冲突。在大型项目或多库项目中，使用命名空间
  可以更清晰地管理各个组件。

### 使用场景

- 当项目中包含多个库时，通过为库设定明确的别名，尤其是带有命名空间的别名，可以提高项目的可维护性和清晰度。
- 接口库非常适合用于头文件库（如 Inja），因为它们主要通过头文件提供功能，不涉及编译链接实体代码的过程。
]]
add_library(inja INTERFACE)
add_library(pantor::inja ALIAS inja)


#[[
杨小兵-2024-09-20

  
]]
target_include_directories(inja INTERFACE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:${INJA_INSTALL_INCLUDE_DIR}>
)
target_compile_features(inja INTERFACE cxx_std_17)


#[[
杨小兵-2024-09-20

  
]]
if(INJA_USE_EMBEDDED_JSON)
  find_package(nlohmann_json QUIET)
  if(NOT nlohmann_json_FOUND)
    set(INJA_PACKAGE_USE_EMBEDDED_JSON ON)
    add_library(nlohmann_json INTERFACE)
    add_library(pantor::nlohmann_json ALIAS nlohmann_json)

    target_include_directories(nlohmann_json INTERFACE
      $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/third_party/include>
      $<INSTALL_INTERFACE:${INJA_INSTALL_INCLUDE_DIR}/nlohmann>
    )

    target_compile_features(nlohmann_json INTERFACE cxx_std_17)

    install(TARGETS nlohmann_json EXPORT injaTargets)

    set(INJA_SELECTED_JSON_LIBRARY "pantor::nlohmann_json")
  else()
    set(INJA_SELECTED_JSON_LIBRARY "nlohmann_json::nlohmann_json")
  endif()
else()
  # If target already exists, e.g. by git submodules
  if(TARGET nlohmann_json)
    set(INJA_SELECTED_JSON_LIBRARY "nlohmann_json::nlohmann_json")

    install(TARGETS nlohmann_json EXPORT injaTargets)
  else()
    find_package(nlohmann_json REQUIRED)
    set(INJA_SELECTED_JSON_LIBRARY "nlohmann_json::nlohmann_json")
  endif()
endif()


target_link_libraries(inja INTERFACE ${INJA_SELECTED_JSON_LIBRARY})


execute_process(COMMAND scripts/update_single_include.sh WORKING_DIRECTORY ${PROJECT_SOURCE_DIR})


if(BUILD_TESTING AND INJA_BUILD_TESTS)
  enable_testing()

  add_executable(inja_test test/test.cpp)
  target_link_libraries(inja_test PRIVATE inja)
  add_test(inja_test ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/inja_test)

  if(MSVC)
    target_compile_options(inja_test PRIVATE /W4 /permissive-)
  else()
    target_compile_options(inja_test PRIVATE -Wall -Wextra -Werror)
  endif()


  add_library(single_inja INTERFACE)
  target_compile_features(single_inja INTERFACE cxx_std_17)
  target_include_directories(single_inja INTERFACE single_include include third_party/include)

  add_executable(single_inja_test test/test.cpp)
  target_link_libraries(single_inja_test PRIVATE single_inja)

  add_test(single_inja_test ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/single_inja_test)
endif()


if(BUILD_BENCHMARK)
  add_executable(inja_benchmark test/benchmark.cpp)
  target_link_libraries(inja_benchmark PRIVATE inja)
endif()


if(COVERALLS)
  include(Coveralls)
  coveralls_turn_on_coverage()

  file(GLOB_RECURSE COVERAGE_SRCS include/inja/*.hpp)

  # set(COVERAGE_SRCS test/unit.cpp test/unit-renderer.cpp include/inja)

  coveralls_setup("${COVERAGE_SRCS}" OFF) # If we should upload.
endif()


include(CMakePackageConfigHelpers)

write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/injaConfigVersion.cmake"
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY SameMajorVersion
)


# build tree package config
configure_file(
  cmake/config/injaBuildConfig.cmake.in
  injaConfig.cmake
  @ONLY
)


install(TARGETS inja EXPORT injaTargets)


export(
  EXPORT injaTargets
  NAMESPACE pantor::
  FILE "${CMAKE_CURRENT_BINARY_DIR}/injaTargets.cmake"
)

# build tree package config
configure_file(
  cmake/config/injaBuildConfig.cmake.in
  injaConfig.cmake
  @ONLY
)


if(INJA_INSTALL)
  set(INJA_CONFIG_PATH "lib/cmake/inja")

  # install tree package config
  configure_package_config_file(
    cmake/config/injaConfig.cmake.in
    ${INJA_CONFIG_PATH}/injaConfig.cmake
    INSTALL_DESTINATION ${INJA_CONFIG_PATH}
    NO_CHECK_REQUIRED_COMPONENTS_MACRO
  )

  install(
    DIRECTORY include/inja
    DESTINATION ${INJA_INSTALL_INCLUDE_DIR}
    FILES_MATCHING PATTERN "*.hpp"
  )
  if(INJA_USE_EMBEDDED_JSON)
    install(
      DIRECTORY third_party/include/nlohmann
      DESTINATION ${INJA_INSTALL_INCLUDE_DIR}
      FILES_MATCHING PATTERN "*.hpp"
    )
  endif()
  install(
    FILES
      "${CMAKE_CURRENT_BINARY_DIR}/${INJA_CONFIG_PATH}/injaConfig.cmake"
      "${CMAKE_CURRENT_BINARY_DIR}/injaConfigVersion.cmake"
    DESTINATION ${INJA_CONFIG_PATH}
  )

  install(
    EXPORT injaTargets FILE injaTargets.cmake
    NAMESPACE pantor::
    DESTINATION ${INJA_CONFIG_PATH}
  )
endif()


if(INJA_EXPORT)
  export(PACKAGE inja)
endif()
