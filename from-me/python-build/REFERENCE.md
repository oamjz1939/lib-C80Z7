# Reference: Path Handling & LLM Access

On-demand patterns for specific scenarios. See [SKILL.md](SKILL.md) for core coding rules.

1. **File and Directory Path Handling**
   
   - Ask the user only when path choice changes behavior, affects portability, risks overwriting data, or cannot be inferred safely.
   - Add a brief Chinese comment for path constants when the purpose is not obvious.
   - Choose the path resolution strategy that best fits the deployment context. The following patterns are preferred starting points — pick the one that matches the scenario, or combine as needed:
   
   **Environment variable** — when paths should be externally configurable across machines:
   
   - Use a fixed environment variable name such as \`ENV_VAR_NAME = "my_script_env_code79"\`.
   - Read with \`os.environ.get(ENV_VAR_NAME)\`, raise a clear error if missing or empty.
   - Resolve to an absolute path with \`Path(env_path).expanduser().resolve()\`.
   
   **Script-relative** — when the target file is distributed alongside the script:
   - Use \`Path(__file__).resolve().parent\` as the base.
   - Do not rely on the runtime working directory.
   
   **Hardcoded absolute path** — only when the user explicitly provides a fixed local path:
   
   - Define as a top-level \`UPPER_SNAKE_CASE\` constant using a Python raw string literal, e.g. \`INPUT_FILE_PATH = Path(r"C:\path\to\input.csv")\`.
   
2. **LLM Access Convention**
   - When the script needs to access an LLM, use the OpenAI Python library.
   - `base_url` and `api_key` must be read from environment variables and must not be hardcoded.
   - Prefer the following environment variable names unless the user or existing project specifies different names:
     - `CODE79_BASE_URL_ENV = "code79_base_url"`
     - `CODE79_API_KEY_ENV = "code79_api_key"`
   - Define the model name as a top-level constant. Use the following default unless the user or existing project specifies another model:
     - `LLM_MODEL = "deepseek-v4-flash"`
   - If an environment variable is missing or empty, raise a clear error.
   
## Code Examples

### File Path Handling Examples

```python
from pathlib import Path
import os

# 基础路径环境变量名
ENV_VAR_NAME = "my_script_env_code79"

INPUT_FILE_NAME = "input.csv"
SCRIPT_DIR = Path(__file__).resolve().parent

# 用户指定的固定输入文件路径
INPUT_FILE_PATH = Path(r"C:\data\input.csv")


def validate_input_file_path(input_file_path):
    if not input_file_path.exists():
        raise FileNotFoundError(f"未找到输入文件：{input_file_path}")

    return input_file_path


def get_input_file_path_from_env():
    """
    从环境变量读取基础目录，并返回输入文件路径。
    """
    env_path = os.environ.get(ENV_VAR_NAME)

    if not env_path:
        raise ValueError(f"未找到环境变量：{ENV_VAR_NAME}")

    base_path = Path(env_path).expanduser().resolve()
    input_file_path = base_path / INPUT_FILE_NAME

    return validate_input_file_path(input_file_path)


def get_input_file_path_from_script_dir():
    input_file_path = SCRIPT_DIR / INPUT_FILE_NAME

    return validate_input_file_path(input_file_path)


def get_input_file_path_from_hardcoded_path():
    """
    返回用户明确指定的固定输入文件路径。
    """
    input_file_path = INPUT_FILE_PATH

    return validate_input_file_path(input_file_path)
```

### LLM Access Example

```python
import os
from openai import OpenAI

CODE79_BASE_URL_ENV = "code79_base_url"
CODE79_API_KEY_ENV = "code79_api_key"

# 默认模型名称，可按用户要求或项目约定调整
LLM_MODEL = "deepseek-v4-flash"


def create_llm_client():
    """
    从环境变量读取 LLM 服务地址和 API 密钥，并创建 OpenAI 客户端。
    """
    base_url = os.environ.get(CODE79_BASE_URL_ENV)
    api_key = os.environ.get(CODE79_API_KEY_ENV)

    if not base_url:
        raise ValueError(f"未找到环境变量：{CODE79_BASE_URL_ENV}")

    if not api_key:
        raise ValueError(f"未找到环境变量：{CODE79_API_KEY_ENV}")

    return OpenAI(base_url=base_url, api_key=api_key)
```
