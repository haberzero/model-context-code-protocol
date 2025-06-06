# MCCP 配置文件 (`mccp_config.json`) 规范

`mccp_config.json` 文件是 MCCP 项目中的核心组成部分，作为项目特定的配置清单。它定义了指导 MCCP 工具链处理项目的行为描述层 (`.mcbc`) 和符号描述层 (`.mcpc`) 文件以及生成目标源代码 (`src_target`) 所必需的基本参数。

该文件指定了项目元数据、目标编程语言、构建参数、依赖项，以及将 MCCP 层文件映射到目标文件系统结构的规则，包括重要的命名约定。

## 文件位置

根据 MCCP 项目结构约定，`mccp_config.json` 文件应存放在项目的配置文件夹中。标准存放路径建议为：

`config/mccp_config.json`

这是一个相对于项目根目录的路径，与 `mccp_symbols.json` 存放于同一目录下，便于集中管理项目配置。

```
/ 项目根目录
├── config/
│   └── mccp_config.json  <-- 文件位置
├── src_mcbc/
├── src_mcpc/
└── src_target/
```

对于包含多个独立模块/应用的更大型项目，或者像本仓库中的示例项目那样，可能在每个模块/应用自己的目录下包含一个 `config/` 文件夹及其对应的 `mccp_config.json` 和 `mccp_symbols.json`，以实现模块级别的配置管理。例如：`demo_projects/demo1_todo_app/config/mccp_config.json`。

## 字段定义

以下是 `mccp_config.json` 中包含的字段：

*   **`projectName`**:
    *   类型: `string`
    *   描述: MCCP 项目的人类可读名称。用于标识和文档生成。
    *   示例: `"basic_calculator_app"`

*   **`mccpVersion`**:
    *   类型: `string`
    *   描述: 指定项目所基于的 MCCP 框架或协议版本。有助于确保与工具链的兼容性。
    *   示例: `"0.0.2"`

*   **`targetLanguage`**:
    *   类型: `string`
    *   描述: 定义生成的目标源代码的编程语言。此设置是**区分大小写**的，并决定了工具链应用的语言特定规则。
    *   示例: `"C"`

*   **`api-url`**:
    *   类型: `string`
    *   描述: 可选字段，指向外部 API 的服务地址（如模型调用接口）。
    *   示例: `"https://api.example.com/v1"`

*   **`api-key`**:
    *   类型: `string`
    *   描述: 可选字段，用于访问外部 API 的认证密钥（需保密存储）。
    *   示例: `"sk-123xxxxxxxxxxxxx"`

*   **`llmModel`**:
    *   类型: `string`
    *   描述: 指定项目使用的大语言模型名称，影响代码生成的上下文理解。
    *   示例: `"Qwen/Qwen3-8B"`

*   **`dependentLib`**:
    *   类型: `array of strings`
    *   描述: 列出项目依赖的外部库/模块（替代原 `dependencies` 字段）。
    *   示例: `["numpy", "scipy"]`

*   **`buildSettings`**:
    *   类型: `object`
    *   描述: 包含与构建目标源代码相关的设置（支持跨语言扩展）。
    *   示例: 
    ```json
    {
      "installCommand": ""  // 可选，该项定义待未来完善
    }
    ```

## 文件系统映射和命名约定 (`fileSystemMapping` 对象)

这是一个可选但强烈推荐的对象，它定义了 `src_mbh` 和 `src_mpc` 中的结构如何映射到 `src_target`
如果省略，将应用默认的工程结构。本项参数的设计目的是为了适应不同的历史代码结构和项目需求。
例如，在一个大型项目中，开发者只希望在某个小模块中使用 MCCP 技术，而不希望对整个项目进行全面改造。
此时，可以通过 `fileSystemMapping` 来指定该小模块的 MCCP 层文件的目录结构，以及目标代码的目录结构，从而实现对特定模块的 MCCP 支持，同时又不过多影响原始项目的文件目录结构

```json
"fileSystemMapping": {
  "mcbcDir": "src_mcbc/", // 行为描述层目录，允许存在不同的工程结构，便于适配不同的历史性代码
  "mcpcDir": "src_mcpc/",   // 符号描述层目录，允许存在不同的工程结构，便于适配不同的历史性代码
  "targetDir": "src_target/",  // 目标代码目录，允许存在不同的工程结构，便于适配不同的历史性代码
  "isExtraSuffix": false // 默认值，与文件命名有关，见下方规则
}
```

*   **`behavioralLayerDir`**:
    *   类型: `string`
    *   描述: 行为描述层文件的根目录。
    *   标准值: `"src_mbh/"`

*   **`symbolicLayerDir`**:
    *   类型: `string`
    *   描述: 符号-伪代码层文件的根目录。
    *   标准值: `"src_mpc/"`

*   **`targetLayerDir`**:
    *   类型: `string`
    *   描述: 目标源代码文件的根目录。
    *   标准值: `"src_target/"`

*   **`is_extra_suffix`**:
    *   类型: `boolean`
    *   描述: 此字段控制是否在对应的 `.mcbc` 和 `.mcpc` 文件的基本文件名之后添加一个派生自原始目标文件扩展名的额外后缀。这对于区分对应不同目标文件类型（例如 C 语言中的 `.c` 和 `.h`）的 MCCP 文件至关重要。事实上，此字段主要是针对C/C++进行设计的。

    *   **通用行为 (应用时):** 如果为 `true`，则在 MCCP 层扩展名 (`.mcbc` 或 `.mcpc`) 之前，会在基本文件名后附加一个下划线 (`_`) 跟着原始目标文件的扩展名（不包含开头的点）。
        *   示例: 目标文件 `file.ext` -> MCCP 文件 `file_ext.mcbc`, `file_ext.mcpc`。

    *   **C 语言家族 (`C`, `C++`, 等):**
        *   对于 `targetLanguage` 设置为 C 语言家族（例如 `"C"`, `"C++"`）时，`is_extra_suffix` 的行为是**隐式为 `true`** 的，并且通常不应在配置中显式设置为 `false`，因为工具链会强制执行特定的 C 语言家族命名规则，而无论此标志在 JSON 中的值如何。
        *   额外后缀是根据 C/C++ 文件类型**固定**的：
            *   对于头文件（例如 `.h`, `.hpp`）：额外后缀是 `_h`。
                *   示例: `utils.h` -> `utils_h.mcbc`, `utils_h.mcpc`。
            *   对于源文件（例如 `.c`, `.cpp`, `.cc`, `.cxx`）：额外后缀分别是 `_c` 或 `_cpp`。
                *   示例: `main.c` -> `main_c.mcbc`, `main_c.mcpc`。
                *   示例: `module.cpp` -> `module_cpp.mcbc`, `module_cpp.mcpc`。

    *   **其他语言（例如 `Python`, `Java`, `JavaScript`）:**
        *   对于其他语言，`is_extra_suffix` 字段是**可配置的**。
        *   如果 `is_extra_suffix` 为 `true`：额外后缀直接派生自目标文件的扩展名（例如 `.py` -> `_py`）。
            *   示例: `script.py` -> `script_py.mcbc`, `script_py.mcpc`。
        *   如果 `is_extra_suffix` 为 `false` 或字段被省略（默认行为）：不添加额外后缀。
            *   示例: `script.py` -> `script.mcbc`, `script.mcpc`。

## 其他标准字段

*   **`dependencies`**:
    *   类型: `array of strings`
    *   描述: 列出了目标代码依赖的外部库、框架或模块。这些信息可用于指导代码生成（例如，建议导入）或由构建脚本使用。
    *   示例 (Python): `["json", "requests"]`
    *   示例 (C): `["math", "pthread"]` (可能因系统/构建设置而异)

*   **`buildSettings`**:
    *   类型: `object`
    *   描述: 包含与构建目标源代码相关的设置。此对象的结构和内容高度依赖于 `targetLanguage`。
    *   示例 (C/C++): `{"compilerFlags": "-Wall -Wextra -pedantic", "linkerFlags": "-lm"}`
    *   示例 (Python): `{"installCommand": "pip install -r requirements.txt"}`
    *   示例 (Java): `{"buildTool": "maven", "mavenGoals": ["clean", "install"]}`

*   **`fileSpecificSettings`**:
    *   类型: `objects`
    *   描述: 暂时作为一个占位符存在，可能在未来用于定义特定文件的额外设置。
    *   示例 (概念性): `{ "targetFile": "src_target/utils.c", "settings": { "generateHeaderGuard": true } }`

## 示例

以下是针对不同场景的 `mccp_config.json` 示例，基于 MCCP 示例项目和规范。

### 示例 1: Python 项目 (`demo1_hello_world_app`)(demo1 暂未编写，意图展示mccp工程中的源码文件映射)

```json
  等待填充
```

### 示例 2: C 项目 (基础计算器 - `demo2_calculator_app`)(demo2 暂未编写，意图展示C语言的固定is_extra_suffix)

```json
  等待填充
```

### 示例 3: Python 项目，但设定 `is_extra_suffix: true`(demo3 暂未编写，意图展示py下的is_extra_suffix的作用)

这是一个假设的 Python 项目示例，其中显式地将文件扩展名添加到 MCCP 文件名中以增加清晰度。

```json
{
    "projectName": "basic_statistics_app",
    "mccpVersion": "0.0.2",
    "targetLanguage": "Python",
    "api-url": "",
    "api-key": "",
    "llmModel": "Qwen/Qwen3-8B",
    "dependentLib": [
        "numpy",
        "scipy"
    ],
    "buildSettings": {
        "installCommand": ""
    },
    "fileSystemMapping": {
        "behavioralLayerDir": "src_mbh/",
        "symbolicLayerDir": "src_mpc/",
        "targetLayerDir": "src_target/",
        "is_extra_suffix": true
    },
    "fileSpecificSettings":{}
}
```

在这个示例中，`is_extra_suffix` 被设置为 `true`，这意味着 `.mcbc` 和 `.mcpc` 文件的基本文件名后会附加一个下划线和原始目标文件的扩展名。例如，对于 `main.py`，对应的 MCCP 文件将是 `main_py.mcbc` 和 `main_py.mcpc`。
