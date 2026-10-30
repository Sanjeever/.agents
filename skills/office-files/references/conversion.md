# Office 文件转换模板

阅读本文件前，请确保已阅读 `../SKILL.md` 的通用工作流。

> **输入路径处理（必读）**：本文件示例多用 `WORKDIR = Path.cwd()` + `glob()` 来发现文件。**仅在用户没有指定具体文件名时**才照搬这种写法。
> 若用户指定了具体文件，把 `WORKDIR + glob` 那两行换成 `INPUT_PATH = Path(r"C:\...\具体文件.扩展名")`，详见 `../SKILL.md` 的 Script Requirements。

---

## 一、转换方案选择

| 转换 | 推荐方案 | 备注 |
|---|---|---|
| docx → pdf | `docx2pdf`（Windows / macOS） | 调用本机 Word / Pages，保真度最高 |
| pptx → pdf | `comtypes`（仅 Windows） | 调用本机 PowerPoint COM 接口 |
| xlsx → pdf | `comtypes`（仅 Windows） | 调用本机 Excel COM 接口 |
| 任意 → pdf（无 Office 环境） | LibreOffice `soffice --headless --convert-to pdf` | 跨平台；需用户已安装 LibreOffice |
| xlsx ↔ csv | `pandas` | 见 `excel.md` 第 8、9 节 |

**重要前提**：`docx2pdf` 和 `comtypes` 方案都需要本机安装 Microsoft Office。运行前先确认；不确认就先询问用户。

---

## 二、docx → pdf（推荐：docx2pdf）

```python
# /// script
# requires-python = ">=3.10"
# dependencies = ["docx2pdf"]
# ///
from pathlib import Path
from docx2pdf import convert

WORKDIR = Path.cwd()

def main():
    files = list(WORKDIR.glob("*.docx"))
    if not files:
        raise FileNotFoundError("未找到 docx 文件")

    for input_path in files:
        output_path = input_path.with_suffix(".pdf")
        convert(str(input_path), str(output_path))
        print(f"已转换：{output_path}")

if __name__ == "__main__":
    main()
```

注意事项：
- 首次运行会启动 Word 进程，耗时 5-10 秒；批量转换时 Word 进程会被复用。
- Word 正在前台被用户打开同一个文件时会失败，需先关闭。
- macOS 上调用 Pages，对复杂格式（域、公式、文本框）的保真度低于 Windows 上的 Word。

---

## 三、pptx / xlsx → pdf（Windows + Office COM）

```python
# /// script
# requires-python = ">=3.10"
# dependencies = ["comtypes"]
# ///
from pathlib import Path
import comtypes.client

WORKDIR = Path.cwd()

# 17 = ppSaveAsPDF，0 = xlTypePDF
PPT_PDF_FORMAT = 32   # ppSaveAsPDF 在 PowerPoint 2010+ 为 32
XLS_PDF_FORMAT = 0    # xlTypePDF

def pptx_to_pdf(input_path: Path) -> Path:
    output_path = input_path.with_suffix(".pdf")
    powerpoint = comtypes.client.CreateObject("PowerPoint.Application")
    try:
        deck = powerpoint.Presentations.Open(str(input_path), WithWindow=False)
        deck.SaveAs(str(output_path), PPT_PDF_FORMAT)
        deck.Close()
    finally:
        powerpoint.Quit()
    return output_path

def xlsx_to_pdf(input_path: Path) -> Path:
    output_path = input_path.with_suffix(".pdf")
    excel = comtypes.client.CreateObject("Excel.Application")
    excel.Visible = False
    try:
        wb = excel.Workbooks.Open(str(input_path))
        wb.ExportAsFixedFormat(XLS_PDF_FORMAT, str(output_path))
        wb.Close(SaveChanges=False)
    finally:
        excel.Quit()
    return output_path

def main():
    for path in WORKDIR.glob("*.pptx"):
        print(f"PPT → PDF：{pptx_to_pdf(path)}")
    for path in WORKDIR.glob("*.xlsx"):
        print(f"XLSX → PDF：{xlsx_to_pdf(path)}")

if __name__ == "__main__":
    main()
```

---

## 四、跨平台回退：LibreOffice 命令行

不依赖 Microsoft Office，需要用户已安装 LibreOffice。

```bash
# Linux / macOS
soffice --headless --convert-to pdf input.docx --outdir ./

# Windows（默认安装路径）
"C:\Program Files\LibreOffice\program\soffice.exe" --headless --convert-to pdf input.docx --outdir .\
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
import sys

WORKDIR = Path.cwd()

def find_soffice() -> str:
    candidate = shutil.which("soffice")
    if candidate:
        return candidate
    if sys.platform == "win32":
        win_path = Path(r"C:\Program Files\LibreOffice\program\soffice.exe")
        if win_path.exists():
            return str(win_path)
    raise FileNotFoundError("未找到 LibreOffice (soffice)，请先安装。")

def main():
    soffice = find_soffice()
    patterns = ("*.docx", "*.pptx", "*.xlsx")

    for pattern in patterns:
        for path in WORKDIR.glob(pattern):
            print(f"转换：{path.name}")
            subprocess.run(
                [soffice, "--headless", "--convert-to", "pdf",
                 str(path), "--outdir", str(WORKDIR)],
                check=True,
            )

if __name__ == "__main__":
    main()
```

---

## 五、何时不转换、转告用户手动操作

| 场景 | 建议 |
|---|---|
| 本机未安装 Office 且未装 LibreOffice | 提示用户：在 Word/PowerPoint/Excel 中"文件 → 另存为 PDF"，或安装 LibreOffice |
| 文档含密码保护 | 不要尝试绕过；让用户先解除保护 |
| .doc / .ppt / .xls 旧格式 | 先让用户在 Office 中另存为 .docx / .pptx / .xlsx |
| 含动画的 PPT 要求"保留动画" | PDF 无动画；用户需要的可能是导出为视频，本 skill 不涵盖 |
