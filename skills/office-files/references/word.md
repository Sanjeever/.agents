# Word（python-docx）模板

依赖：`python-docx`（Python 中 `import docx`）

阅读本文件前，请确保已阅读 `../SKILL.md` 的通用工作流。

> **输入路径处理（必读）**：本文件示例多用 `WORKDIR = Path.cwd()` + `glob("*.docx")` 来发现文件。**仅在用户没有指定具体文件名时**才照搬这种写法。
> 若用户指定了具体文件，把 `WORKDIR + glob` 那两行换成 `INPUT_PATH = Path(r"C:\...\具体文件.docx")`，详见 `../SKILL.md` 的 Script Requirements。

---

## 1. 读取所有段落

```python
# /// script
# requires-python = ">=3.10"
# dependencies = ["python-docx"]
# ///
from pathlib import Path
import docx

WORKDIR = Path.cwd()

def main():
    files = list(WORKDIR.glob("*.docx"))
    if not files:
        raise FileNotFoundError("未找到 docx 文件")

    input_path = files[0]
    doc = docx.Document(str(input_path))

    print(f"文件：{input_path.name}")
    for i, p in enumerate(doc.paragraphs, start=1):
        text = p.text.strip()
        if text:
            print(f"[P{i}] {text}")

if __name__ == "__main__":
    main()
```

## 2. 读取所有表格

```python
# /// script
# requires-python = ">=3.10"
# dependencies = ["python-docx"]
# ///
from pathlib import Path
import docx

WORKDIR = Path.cwd()

def main():
    files = list(WORKDIR.glob("*.docx"))
    if not files:
        raise FileNotFoundError("未找到 docx 文件")

    input_path = files[0]
    doc = docx.Document(str(input_path))

    print(f"文件：{input_path.name}")

    for ti, table in enumerate(doc.tables, start=1):
        print(f"\n=== 表格 {ti}：{len(table.rows)} 行 x {len(table.columns)} 列 ===")
        for ri, row in enumerate(table.rows, start=1):
            cells = [cell.text.replace("\n", " ").strip() for cell in row.cells]
            print(f"R{ri}: " + " | ".join(cells))

if __name__ == "__main__":
    main()
```

## 3. 读取标题结构

```python
# /// script
# requires-python = ">=3.10"
# dependencies = ["python-docx"]
# ///
from pathlib import Path
import docx

WORKDIR = Path.cwd()

def main():
    files = list(WORKDIR.glob("*.docx"))
    if not files:
        raise FileNotFoundError("未找到 docx 文件")

    input_path = files[0]
    doc = docx.Document(str(input_path))

    print(f"文件：{input_path.name}")

    for p in doc.paragraphs:
        style_name = p.style.name or ""
        text = p.text.strip()
        if not text:
            continue

        if style_name.startswith("Heading") or style_name.startswith("标题"):
            print(f"[{style_name}] {text}")

if __name__ == "__main__":
    main()
```

## 4. 文本替换（同 run 内）

最简单的情况：替换发生在单个 run 内部。Word 经常把一段文字拆成多个 run（例如部分加粗、不同字体），这种简单替换匹配不到跨 run 的文本。跨 run 替换见下一节。

```python
# /// script
# requires-python = ">=3.10"
# dependencies = ["python-docx"]
# ///
from pathlib import Path
import docx

WORKDIR = Path.cwd()
OLD_TEXT = "旧文本"
NEW_TEXT = "新文本"

def replace_in_paragraphs(paragraphs, old: str, new: str) -> int:
    count = 0
    for p in paragraphs:
        for run in p.runs:
            if old in run.text:
                run.text = run.text.replace(old, new)
                count += 1
    return count

def main():
    files = list(WORKDIR.glob("*.docx"))
    if not files:
        raise FileNotFoundError("未找到 docx 文件")

    input_path = files[0]
    output_path = input_path.with_name(input_path.stem + "_modified.docx")

    doc = docx.Document(str(input_path))

    count = 0
    count += replace_in_paragraphs(doc.paragraphs, OLD_TEXT, NEW_TEXT)

    for table in doc.tables:
        for row in table.rows:
            for cell in row.cells:
                count += replace_in_paragraphs(cell.paragraphs, OLD_TEXT, NEW_TEXT)

    doc.save(str(output_path))

    print(f"输入文件：{input_path}")
    print(f"输出文件：{output_path}")
    print(f"替换次数：{count}")

if __name__ == "__main__":
    main()
```

## 5. 跨 run 文本替换

思路：把一个 paragraph 的所有 run 拼成完整字符串做替换，然后把替换后的文本写回第一个 run，把其余 run 清空。这样会丢失原文本内部的格式差异（例如部分加粗），但能保证替换成功，且 paragraph 级别的格式（对齐、缩进、段前段后）保留。

适用：替换出现在多个 run 拼接处时使用；如果原文本格式简单（同一字体、同一字号），效果通常可接受。

```python
# /// script
# requires-python = ">=3.10"
# dependencies = ["python-docx"]
# ///
from pathlib import Path
import docx

WORKDIR = Path.cwd()
OLD_TEXT = "旧文本"
NEW_TEXT = "新文本"

def replace_in_paragraph_merged(paragraph, old: str, new: str) -> int:
    """合并所有 run 文本后替换，写回首 run，清空其余 run。"""
    if not paragraph.runs:
        return 0

    full_text = "".join(run.text for run in paragraph.runs)
    if old not in full_text:
        return 0

    count = full_text.count(old)
    replaced = full_text.replace(old, new)

    paragraph.runs[0].text = replaced
    for run in paragraph.runs[1:]:
        run.text = ""

    return count

def replace_in_paragraphs(paragraphs, old: str, new: str) -> int:
    return sum(replace_in_paragraph_merged(p, old, new) for p in paragraphs)

def main():
    files = list(WORKDIR.glob("*.docx"))
    if not files:
        raise FileNotFoundError("未找到 docx 文件")

    input_path = files[0]
    output_path = input_path.with_name(input_path.stem + "_replaced.docx")

    doc = docx.Document(str(input_path))

    count = 0
    count += replace_in_paragraphs(doc.paragraphs, OLD_TEXT, NEW_TEXT)

    for table in doc.tables:
        for row in table.rows:
            for cell in row.cells:
                count += replace_in_paragraphs(cell.paragraphs, OLD_TEXT, NEW_TEXT)

    doc.save(str(output_path))
    print(f"替换次数：{count}，输出：{output_path}")

if __name__ == "__main__":
    main()
```

## 6. 提取文档中的图片

`.docx` 本质是 zip。图片以原始字节存储在 `word/media/` 目录下。直接用 `zipfile` 提取最简单可靠，无需依赖 `python-docx`。

```python
# /// script
# requires-python = ">=3.10"
# dependencies = []
# ///
from pathlib import Path
import zipfile

WORKDIR = Path.cwd()

def main():
    files = list(WORKDIR.glob("*.docx"))
    if not files:
        raise FileNotFoundError("未找到 docx 文件")

    input_path = files[0]
    out_dir = WORKDIR / f"{input_path.stem}_images"
    out_dir.mkdir(exist_ok=True)

    with zipfile.ZipFile(input_path) as zf:
        media = [n for n in zf.namelist() if n.startswith("word/media/")]
        for name in media:
            target = out_dir / Path(name).name
            target.write_bytes(zf.read(name))
            print(f"导出：{target}")

    print(f"\n共导出 {len(media)} 张图片到：{out_dir}")

if __name__ == "__main__":
    main()
```

## 7. 创建新 Word 文档

```python
# /// script
# requires-python = ">=3.10"
# dependencies = ["python-docx"]
# ///
from pathlib import Path
import docx

WORKDIR = Path.cwd()

def main():
    output_path = WORKDIR / "output.docx"

    doc = docx.Document()
    doc.add_heading("项目汇报", level=1)
    doc.add_paragraph("这里是正文第一段。")

    doc.add_heading("一、项目概况", level=2)
    doc.add_paragraph("这里是项目概况内容。")

    table = doc.add_table(rows=3, cols=2)
    table.style = "Table Grid"

    data = [
        ["名称", "数值"],
        ["事项 A", "100"],
        ["事项 B", "200"],
    ]

    for ri, row_data in enumerate(data):
        for ci, value in enumerate(row_data):
            table.rows[ri].cells[ci].text = value

    doc.save(str(output_path))
    print(f"已创建：{output_path}")

if __name__ == "__main__":
    main()
```

## 8. 限制速查

| 项目 | 说明 |
|---|---|
| 页眉页脚 | 通过 `doc.sections[i].header / footer` 访问，支持有限 |
| 批注 | python-docx 不直接支持，需操作 XML |
| 文本框 | 部分文本框无法用 `doc.paragraphs` 遍历到，需扫描 shapes |
| 域代码 | 复杂域（目录、超链接等）写入后可能需要在 Word 中按 F9 刷新 |
| 跨 run 文本 | 见第 5 节 |
