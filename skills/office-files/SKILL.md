---
name: office-files
description: "读取、提取、修改、创建或转换 Office 文件，包括 Word 文档、PowerPoint 演示文稿、Excel 表格、CSV/PDF 转换。适用于用户明确提到 docx、pptx、xlsx、csv、pdf、Word、PowerPoint、Excel、spreadsheet、presentation、document、文档、表格、演示、幻灯片、办公文件、Office 文件等任务。"
---

# Office 文件处理

使用一次性 Python 脚本处理 `.docx`、`.pptx`、`.xlsx`、`.csv` 与常见 Office 到 PDF 转换。先按任务类型读取对应参考文件，再执行脚本。

## 参考文件路由

- Word `.docx`：读取 `references/word.md`
- PowerPoint `.pptx`：读取 `references/powerpoint.md`
- Excel `.xlsx`、CSV、数据分析：读取 `references/excel.md`
- Office ↔ PDF、批量转换：读取 `references/conversion.md`

## 核心工作流

1. 确认输入文件类型和用户目标：读取、提取、修改、创建、转换或分析。
2. 读取对应 `references/*.md`，只加载当前任务需要的参考文件。
3. 编写一次性 Python 脚本；脚本顶部包含 PEP 723 内联依赖声明。
4. 用户给出具体文件路径时，把完整路径作为 `Path(r"...")` 字符串字面量写入脚本；用户只给目录或要求批量处理时，用 `Path.glob()` 发现文件。
5. 修改已有 Office 文件时默认另存为新文件，除非用户明确要求覆盖。
6. 使用 `PYTHONUTF8=1 uv run script.py` 运行脚本。
7. 成功后报告输出文件；失败时保留脚本并报告错误。

## 库选择

| 任务 | 库 | PEP 723 依赖 | 导入 |
|---|---|---|---|
| Word `.docx` | python-docx | `python-docx` | `import docx` |
| PowerPoint `.pptx` | python-pptx | `python-pptx` | `import pptx` |
| Excel cell-level read/write | openpyxl | `openpyxl` | `import openpyxl` |
| Excel/CSV analysis and conversion | pandas | `pandas`, `openpyxl` | `import pandas as pd` |
| DOCX to PDF | docx2pdf | `docx2pdf` | `from docx2pdf import convert` |
| PPTX/XLSX to PDF on Windows | comtypes | `comtypes` | `import comtypes.client` |

## 脚本要求

Use PEP 723 metadata at the top of every generated script:

```python
# /// script
# requires-python = ">=3.10"
# dependencies = ["python-docx"]
# ///
```

Run the script directly with `uv` so it can read the PEP 723 dependencies:

```bash
PYTHONUTF8=1 uv run script.py
```

Do not use command-line arguments for Chinese file paths. Encode paths in the script:

```python
from pathlib import Path

INPUT_PATH = Path(r"C:\Users\Sanjeev\Documents\服务内容清单.docx")
if not INPUT_PATH.exists():
    raise FileNotFoundError(INPUT_PATH)
```

目录或批量任务使用 `glob()` 显式发现文件；找不到文件时直接失败并报告：

```python
from pathlib import Path

WORKDIR = Path(r"C:\Users\Sanjeev\Documents")
files = sorted(WORKDIR.glob("*.docx"))
if not files:
    raise FileNotFoundError("未找到 docx 文件")
```

## 输出策略

- 少量文本结果直接 `print()`。
- 表格、长文本或结构化结果写入 `result.txt`、`result.json`、`result.csv` 或新的 Office 文件。
- 一次性读取/查询脚本执行成功后可以删除。
- 产生 Office、PDF 或结果文件的脚本保留，并在回复中说明脚本路径。
- 执行失败的脚本保留，便于排查。

## 边界条件

- 不处理 `.doc`、`.xls`、`.ppt` 旧格式；要求用户先另存为 `.docx`、`.xlsx`、`.pptx`。
- 不绕过加密或密码保护文件；要求用户先解除保护。
- 不执行宏；处理 `.docm`、`.xlsm`、`.pptm` 时明确提示宏可能受影响。
- 不强行修复损坏文件；报告读取或转换失败。
- 超大 Excel 必须使用 `openpyxl.load_workbook(..., read_only=True)` 流式读取。

## 执行前检查

1. 已按任务类型读取对应 `references/*.md`。
2. 脚本包含 PEP 723 头，依赖列表完整。
3. 具体文件用路径字面量；目录或批量任务才使用 `glob()`。
4. 修改类任务默认另存。
5. 运行方式为 `PYTHONUTF8=1 uv run script.py`。
6. 失败时保留脚本并报告异常。
