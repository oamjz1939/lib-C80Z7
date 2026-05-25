---
name: python-build
description: Write and modify Python code. Only used when the user explicitly states such as: "Use python-build".
---

## Overview

Help users create or modify Python scripts. Prioritize clear logic, minimal changes, verifiable results, and code suitable for stable execution.

## Core Instructions

1. **Clarify Before Implementing**
   - State key assumptions clearly; if multiple reasonable interpretations exist, present them first.
   - If the requirement is unclear or affects the implementation direction, stop and ask the user.

3. **Minimal Changes**
   
   - Change only the code required to satisfy the request.
   - Do not perform unrelated refactoring, renaming, formatting, or style cleanup.
   
4. **Python Code Standards**
   
   - Extract critical configuration variables into top-level constants using `UPPER_SNAKE_CASE`; path sources must be confirmed item by item according to the “File and Directory Path Handling” rules.
   - Add a brief Chinese comment explaining the purpose of each constant.
   - Immediately after every function definition, add a multi-line Chinese docstring explaining every parameter and its role.
   - Avoid clever syntax. Prefer clear `if-else` blocks, standard loops, and explicit logic.
   
5. **File and Directory Path Handling**
   
   - For every file or directory accessed by the script, ask the user which path handling method should be used. Do not decide silently.
   - Each path constant must have a brief Chinese comment explaining its purpose.
   - Only three path handling methods are allowed:
   
   **Method 1: Environment Variable Path**
   
   - Use the fixed environment variable name: `ENV_VAR_NAME = "my_script_env_code79"`.
   - Read the base path with `os.environ.get(ENV_VAR_NAME)`.
   - If the environment variable is missing, empty, or the target file/directory cannot be found, raise a clear error.
   - Convert the path to an absolute path with `Path(env_path).expanduser().resolve()`.
   - Use this when the user wants the path to be externally configurable and the script to run across machines or directories.
   
   **Method 2: Script Directory Path**
   - Use `Path(__file__).resolve().parent` to get the script directory.
   - Access the target file or directory under the script directory.
   - Do not rely on the runtime working directory.
   - Use this when the target file or directory is distributed with the script or fixed near the script location.
   
   **Method 3: Hardcoded Raw String Path**
   
   - Define the full path as a top-level `UPPER_SNAKE_CASE` constant using `Path` with a Python raw string literal, such as `INPUT_FILE_PATH = Path(r"C:\path\to\input.csv")`.
   - Path-related constants must still be defined at the top of the script using `UPPER_SNAKE_CASE`.
   
6. **LLM Access Convention**
   - When the script needs to access an LLM, use the OpenAI Python library.
   - `base_url` and `api_key` must be read from environment variables and must not be hardcoded.
   - Use the following fixed environment variable names:
     - `CODE79_BASE_URL_ENV = "code79_base_url"`
     - `CODE79_API_KEY_ENV = "code79_api_key"`
   - Hardcode the fixed model name as a top-level constant:
     - `LLM_MODEL = "deepseek-v4-flash"`
   - If an environment variable is missing or empty, raise a clear error.
   
7. **Language Requirements**
   - All code comments, docstrings, parameter explanations, and descriptive text inside generated code must be written in professional, concise Chinese.

## Code Examples

### File Path Handling Examples

```python
from pathlib import Path
import os

# 基础路径环境变量名
ENV_VAR_NAME = "my_script_env_code79"

# 输入文件名称
INPUT_FILE_NAME = "input.csv"

# 脚本所在目录
SCRIPT_DIR = Path(__file__).resolve().parent

# 输入文件完整路径
INPUT_FILE_PATH = Path(r"C:\data\input.csv")


def validate_input_file_path(input_file_path):
    """
    参数说明：
    - input_file_path：需要验证是否存在的输入文件路径。
    """
    if not input_file_path.exists():
        raise FileNotFoundError(f"未找到输入文件：{input_file_path}")

    return input_file_path


def get_input_file_path_from_env():
    """
    参数说明：
    - 无参数：从环境变量读取基础路径，并返回输入文件路径。
    """
    env_path = os.environ.get(ENV_VAR_NAME)

    if not env_path:
        raise ValueError(f"未找到环境变量：{ENV_VAR_NAME}")

    base_path = Path(env_path).expanduser().resolve()
    input_file_path = base_path / INPUT_FILE_NAME

    return validate_input_file_path(input_file_path)


def get_input_file_path_from_script_dir():
    """
    参数说明：
    - 无参数：根据脚本所在目录返回输入文件路径。
    """
    input_file_path = SCRIPT_DIR / INPUT_FILE_NAME

    return validate_input_file_path(input_file_path)


def get_input_file_path_from_hardcoded_path():
    """
    参数说明：
    - 无参数：根据代码中固定的 raw string 路径返回输入文件路径。
    """
    input_file_path = INPUT_FILE_PATH

    return validate_input_file_path(input_file_path)
```

### LLM Access Example

```python
import os
from openai import OpenAI

# LLM 服务地址环境变量名
CODE79_BASE_URL_ENV = "code79_base_url"

# LLM API 密钥环境变量名
CODE79_API_KEY_ENV = "code79_api_key"

# 固定使用的 LLM 模型名称
LLM_MODEL = "deepseek-v4-flash"


def create_llm_client():
    """
    参数说明：
    - 无参数：从环境变量读取 LLM 服务地址和 API 密钥，并创建 OpenAI 客户端。
    """
    base_url = os.environ.get(CODE79_BASE_URL_ENV)
    api_key = os.environ.get(CODE79_API_KEY_ENV)

    if not base_url:
        raise ValueError(f"未找到环境变量：{CODE79_BASE_URL_ENV}")

    if not api_key:
        raise ValueError(f"未找到环境变量：{CODE79_API_KEY_ENV}")

    return OpenAI(base_url=base_url, api_key=api_key)
```
