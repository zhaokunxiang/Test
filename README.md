# Pytest 教程

本教程将带你快速了解并上手使用 [pytest](https://docs.pytest.org/)。内容涵盖安装、编写与组织测试、运行测试、使用夹具（fixtures）、参数化、常见插件等主题，可帮助你在真实项目中建立稳健的自动化测试体系。

## 1. 准备环境

1. 安装 Python 3.8 及以上版本。
2. 建议创建虚拟环境隔离依赖：

   ```bash
   python -m venv .venv
   source .venv/bin/activate  # Windows 使用 .venv\Scripts\activate
   ```

3. 安装 pytest：

   ```bash
   pip install pytest
   ```

## 2. 编写第一个测试

在项目根目录创建 `src/calculator.py`：

```python
def add(a: int, b: int) -> int:
    return a + b
```

再创建测试文件 `tests/test_calculator.py`：

```python
from src.calculator import add


def test_add_returns_sum():
    assert add(1, 2) == 3
```

运行测试：

```bash
pytest
```

若一切顺利，你会看到输出 `1 passed`。

## 3. 断言与失败示例

pytest 直接使用 Python 的 `assert` 语句。当断言失败时，pytest 会自动显示对比信息：

```python
def test_add_handles_negative_numbers():
    assert add(-1, 1) == 0
    assert add(-2, -3) == -5
```

如果实际结果不符，你会看到详细的断言差异，便于定位问题。

## 4. 组织测试文件与类

- 测试文件默认以 `test_*.py` 或 `*_test.py` 命名。
- 测试函数以 `test_` 开头。
- 可以使用类组织测试用例，类名以 `Test` 开头且不需要继承任何基类：

```python
class TestCalculator:
    def test_add_zero(self):
        assert add(5, 0) == 5
```

pytest 会自动发现并运行符合命名约定的测试。

## 5. 使用 Fixtures 提供测试数据

Fixtures 用于在测试运行前准备数据或状态。定义在 `conftest.py` 或测试模块中，并使用 `@pytest.fixture` 装饰器：

```python
# tests/conftest.py
import pytest


@pytest.fixture
def numbers():
    return [1, 2, 3]
```

在测试中通过参数注入使用：

```python
def test_sum(numbers):
    assert sum(numbers) == 6
```

### Fixture 的作用域

可以通过 `scope` 参数控制 fixture 作用域：`function`、`class`、`module`、`package`、`session`。例如：

```python
@pytest.fixture(scope="module")
def db_connection():
    ...
```

## 6. 参数化测试

使用 `@pytest.mark.parametrize` 可以用不同输入多次运行同一个测试：

```python
import pytest


@pytest.mark.parametrize("a,b,expected", [
    (2, 3, 5),
    (-1, 1, 0),
    (0, 0, 0),
])
def test_add_multiple_cases(a, b, expected):
    assert add(a, b) == expected
```

## 7. 跳过与预期失败

- 使用 `pytest.skip()` 或 `@pytest.mark.skip` 跳过测试。
- 使用 `@pytest.mark.xfail` 标记已知会失败的测试。

```python
import sys
import pytest


@pytest.mark.skipif(sys.platform == "win32", reason="特定平台暂不支持")
def test_only_on_unix():
    ...


@pytest.mark.xfail(reason="尚未实现")
def test_future_feature():
    assert False
```

## 8. 常用命令行选项

- `-k <表达式>`：按关键字选择测试。
- `-m <标记>`：运行带有特定 `@pytest.mark` 标记的测试。
- `-x`：遇到第一个失败立即停止。
- `--maxfail=2`：失败达到指定次数后停止。
- `-vv`：显示更详细的输出。

示例：

```bash
pytest -k "calculator and not slow" -vv
```

## 9. 使用插件扩展能力

pytest 拥有丰富的插件生态。常见插件包括：

- `pytest-cov`：生成覆盖率报告。
- `pytest-mock`：提供 `mocker` fixture，简化打桩与断言。
- `pytest-django`、`pytest-flask`：为 Web 框架提供专用夹具。

安装插件后即可在测试中直接使用。例如使用覆盖率：

```bash
pip install pytest-cov
pytest --cov=src
```

## 10. 集成持续集成（CI）

在 CI 中运行 pytest 可以及时发现问题。以 GitHub Actions 为例，新建 `.github/workflows/tests.yml`：

```yaml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: pip install -r requirements.txt
      - run: pytest
```

## 11. 调试失败的测试

- 使用 `-x` 或 `--maxfail=1` 聚焦第一个失败。
- `pytest --pdb` 在失败时自动进入调试器。
- `pytest -vv` 查看更详细的断言信息。
- 配合 `print()` 或日志输出定位问题。

## 12. 下一步

- 阅读官方文档以深入了解高级特性，如钩子（hooks）、自定义标记、插件开发等。
- 在真实项目中逐步补充测试，保持持续的反馈循环。

祝你测试愉快！
