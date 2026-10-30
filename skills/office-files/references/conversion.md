# Office 文件转换模板

阅读本文件前，请确保已阅读 `../SKILL.md` 的通用工作流。

> **输入路径处理（必读）**：本文件示例多用 `WORKDIR = Path.cwd()` + `glob()` 来发现文件。**仅在用户没有指定具体文件名时**才照搬这种写法。
> 若用户指定了具体文件，按其实际路径设置 `INPUT_PATH` 并直接处理该文件，替换目录扫描逻辑；详见 `../SKILL.md` 的脚本要求。

---

## 一、转换方案选择

| 转换 | 推荐方案 | 备注 |
|---|---|---|
| docx / pptx / xlsx → pdf | 已安装办公软件提供的转换接口，或 LibreOffice | 根据可用能力和版式保真要求选择 |
| xlsx ↔ csv | `pandas` | 见 `excel.md` 第 8、9 节 |

先通过只读检查确认当前环境已有的转换工具、版本和所需字体，按所选工具的实际接口执行。可执行文件不在 PATH 时，按需查找实际安装位置，不将本机路径固化进技能。

要求保持原版式时，优先使用原格式对应办公软件提供的可用接口。使用其他转换引擎前应判断版式差异是否可接受；转换失败后明确报告，不自动切换引擎降低保真要求。缺少必要工具时说明缺失项，安装依据已有授权。

---

## 二、LibreOffice 转换示例

适用于已确认 LibreOffice 可用且转换效果满足要求的任务。命令名可用时执行：

```text
soffice --headless --convert-to pdf input.docx --outdir .
```

Python 封装：

```python
# /// script
# requires-python = ">=3.10"
# dependencies = []
# ///
from pathlib import Path
import shutil
import subprocess

WORKDIR = Path.cwd()

SOFFICE = shutil.which("soffice") or shutil.which("libreoffice")

def main():
    if SOFFICE is None:
        raise FileNotFoundError("PATH 中未找到 LibreOffice；请先定位可执行文件并据实设置 SOFFICE。")
    patterns = ("*.docx", "*.pptx", "*.xlsx")

    for pattern in patterns:
        for path in WORKDIR.glob(pattern):
            print(f"转换：{path.name}")
            subprocess.run(
                [SOFFICE, "--headless", "--convert-to", "pdf",
                 str(path), "--outdir", str(WORKDIR)],
                check=True,
            )

if __name__ == "__main__":
    main()
```

---

## 三、验证与限制

转换后确认目标文件存在，并检查页数、版式和所需字体。无法实际渲染检查时，明确说明验证限制。

| 场景 | 建议 |
|---|---|
| 未找到满足要求的转换工具 | 报告缺少的能力，说明可行的安装或手动导出方式 |
| 文档含密码保护 | 不要尝试绕过；让用户先解除保护 |
| .doc / .ppt / .xls 旧格式 | 先让用户在 Office 中另存为 .docx / .pptx / .xlsx |
| 含动画的 PPT 要求"保留动画" | PDF 无动画；用户需要的可能是导出为视频，本 skill 不涵盖 |
