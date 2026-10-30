# PowerPoint（python-pptx）模板

依赖：`python-pptx`（Python 中 `import pptx`）

阅读本文件前，请确保已阅读 `../SKILL.md` 的通用工作流。

> **输入路径处理（必读）**：本文件示例多用 `WORKDIR = Path.cwd()` + `glob("*.pptx")` 来发现文件。**仅在用户没有指定具体文件名时**才照搬这种写法。
> 若用户指定了具体文件，按其实际路径设置 `INPUT_PATH` 并直接处理该文件，替换目录扫描逻辑；详见 `../SKILL.md` 的脚本要求。

---

## 1. 幻灯片布局索引

`prs.slide_layouts[n]` 中的 `n` 对应模板布局。不同模板可能不同，标准空白模板通常如下：

| 索引 | 布局名 | 内容 | 适用场景 |
|---|---|---|---|
| 0 | Title Slide | 标题 + 副标题 | 封面、结尾页 |
| 1 | Title and Content | 标题 + 内容 | 标准内容页 |
| 2 | Section Header | 标题 + 副标题 | 章节页 |
| 3 | Two Content | 标题 + 左右两栏 | 左右对比 |
| 4 | Comparison | 标题 + 对比内容 | 对比分析 |
| 5 | Title Only | 仅标题 | 标题 + 自由排版 |
| 6 | Blank | 空白 | 完全自由排版 |

**创建汇报 PPT 时优先使用 `slide_layouts[6]` 空白布局。** 模板布局的字体、字号、占位符在中文环境里经常出现偏差，用空白布局 + `add_textbox()` / `add_shape()` 自由排版更可控。

默认继承模板字体；需要指定字体时，确认当前环境和目标阅读环境具备该字体。

## 2. 按页读取全部文本

```python
# /// script
# requires-python = ">=3.10"
# dependencies = ["python-pptx"]
# ///
from pathlib import Path
import pptx

WORKDIR = Path.cwd()

def main():
    files = list(WORKDIR.glob("*.pptx"))
    if not files:
        raise FileNotFoundError("未找到 pptx 文件")

    input_path = files[0]
    prs = pptx.Presentation(str(input_path))

    print(f"文件：{input_path.name}")

    for si, slide in enumerate(prs.slides, start=1):
        print(f"\n=== 第 {si} 页 ===")

        for shape in slide.shapes:
            if shape.has_text_frame:
                text = shape.text_frame.text.strip()
                if text:
                    print(text)

            if shape.has_table:
                table = shape.table
                for row in table.rows:
                    cells = [cell.text.replace("\n", " ").strip() for cell in row.cells]
                    print(" | ".join(cells))

if __name__ == "__main__":
    main()
```

## 3. 扁平化提取全部文本

```python
# /// script
# requires-python = ">=3.10"
# dependencies = ["python-pptx"]
# ///
from pathlib import Path
import pptx

WORKDIR = Path.cwd()

def main():
    files = list(WORKDIR.glob("*.pptx"))
    if not files:
        raise FileNotFoundError("未找到 pptx 文件")

    input_path = files[0]
    prs = pptx.Presentation(str(input_path))

    for slide in prs.slides:
        for shape in slide.shapes:
            if shape.has_text_frame:
                text = shape.text_frame.text.strip()
                if text:
                    print(text)

if __name__ == "__main__":
    main()
```

## 4. 替换全部幻灯片文本

```python
# /// script
# requires-python = ">=3.10"
# dependencies = ["python-pptx"]
# ///
from pathlib import Path
import pptx

WORKDIR = Path.cwd()
OLD_TEXT = "旧文本"
NEW_TEXT = "新文本"

def main():
    files = list(WORKDIR.glob("*.pptx"))
    if not files:
        raise FileNotFoundError("未找到 pptx 文件")

    input_path = files[0]
    output_path = input_path.with_name(input_path.stem + "_modified.pptx")

    prs = pptx.Presentation(str(input_path))
    count = 0

    for slide in prs.slides:
        for shape in slide.shapes:
            if not shape.has_text_frame:
                continue

            for paragraph in shape.text_frame.paragraphs:
                for run in paragraph.runs:
                    if OLD_TEXT in run.text:
                        run.text = run.text.replace(OLD_TEXT, NEW_TEXT)
                        count += 1

    prs.save(str(output_path))

    print(f"输入文件：{input_path}")
    print(f"输出文件：{output_path}")
    print(f"替换次数：{count}")

if __name__ == "__main__":
    main()
```

## 5. 提取 PPT 中的图片

`.pptx` 同样是 zip。图片在 `ppt/media/`。

```python
# /// script
# requires-python = ">=3.10"
# dependencies = []
# ///
from pathlib import Path
import zipfile

WORKDIR = Path.cwd()

def main():
    files = list(WORKDIR.glob("*.pptx"))
    if not files:
        raise FileNotFoundError("未找到 pptx 文件")

    input_path = files[0]
    out_dir = WORKDIR / f"{input_path.stem}_images"
    out_dir.mkdir(exist_ok=True)

    with zipfile.ZipFile(input_path) as zf:
        media = [n for n in zf.namelist() if n.startswith("ppt/media/")]
        for name in media:
            target = out_dir / Path(name).name
            target.write_bytes(zf.read(name))
            print(f"导出：{target}")

    print(f"\n共导出 {len(media)} 张到：{out_dir}")

if __name__ == "__main__":
    main()
```

## 6. 创建 PPT：空白布局 + 自由排版

示例使用中性配色（黑 / 灰 / 白）。需要换主题时，只调整下列颜色常量即可。

```python
# /// script
# requires-python = ">=3.10"
# dependencies = ["python-pptx"]
# ///
from pathlib import Path
import pptx
from pptx.util import Cm, Pt
from pptx.dml.color import RGBColor
from pptx.enum.text import PP_ALIGN
from pptx.enum.shapes import MSO_SHAPE

WORKDIR = Path.cwd()

# 中性配色，按需替换
PRIMARY = RGBColor(0x22, 0x22, 0x22)   # 主色（标题、深背景）
ACCENT  = RGBColor(0x66, 0x66, 0x66)   # 强调色（细线、副标题）
WHITE   = RGBColor(0xFF, 0xFF, 0xFF)
TEXT    = RGBColor(0x1F, 0x1F, 0x1F)
MUTED   = RGBColor(0x6B, 0x6B, 0x6B)
CARD    = RGBColor(0xF3, 0xF3, 0xF3)

def add_textbox(slide, left, top, width, height, text, font_size=14,
                color=TEXT, bold=False, alignment=PP_ALIGN.LEFT,
                font_name=None):
    tb = slide.shapes.add_textbox(left, top, width, height)
    tf = tb.text_frame
    tf.word_wrap = True

    p = tf.paragraphs[0]
    p.text = text
    p.font.size = Pt(font_size)
    p.font.color.rgb = color
    p.font.bold = bold
    p.font.name = font_name
    p.alignment = alignment
    return tb

def add_rect(slide, left, top, width, height, color, shape_type=MSO_SHAPE.ROUNDED_RECTANGLE):
    shape = slide.shapes.add_shape(shape_type, left, top, width, height)
    shape.fill.solid()
    shape.fill.fore_color.rgb = color
    shape.line.fill.background()
    return shape

def main():
    output_path = WORKDIR / "output.pptx"

    prs = pptx.Presentation()
    prs.slide_width = Cm(33.867)   # 16:9
    prs.slide_height = Cm(19.05)

    # 封面页
    slide = prs.slides.add_slide(prs.slide_layouts[6])
    slide.background.fill.solid()
    slide.background.fill.fore_color.rgb = PRIMARY

    add_rect(slide, Cm(2), Cm(5.5), Cm(1.5), Cm(0.06), ACCENT, MSO_SHAPE.RECTANGLE)
    add_textbox(slide, Cm(2), Cm(6), Cm(28), Cm(2),
                "项目汇报", font_size=40, color=WHITE, bold=True)
    add_textbox(slide, Cm(2), Cm(8.3), Cm(24), Cm(1),
                "副标题 / 汇报单位 / 日期", font_size=16, color=WHITE)

    # 内容页
    slide = prs.slides.add_slide(prs.slide_layouts[6])
    slide.background.fill.solid()
    slide.background.fill.fore_color.rgb = WHITE

    add_rect(slide, Cm(0), Cm(0), Cm(33.867), Cm(1.2), PRIMARY, MSO_SHAPE.RECTANGLE)
    add_textbox(slide, Cm(1.5), Cm(0.18), Cm(20), Cm(0.8),
                "一、章节标题", font_size=14, color=WHITE, bold=True)

    add_rect(slide, Cm(1.5), Cm(3), Cm(14), Cm(5), CARD)
    add_textbox(slide, Cm(2.3), Cm(3.5), Cm(12.5), Cm(0.8),
                "卡片标题", font_size=18, color=PRIMARY, bold=True)
    add_textbox(slide, Cm(2.3), Cm(4.7), Cm(12.5), Cm(2.5),
                "这里填写卡片描述内容。内容应简洁，适合汇报展示。",
                font_size=12, color=MUTED)

    prs.save(str(output_path))
    print(f"已创建：{output_path}")

if __name__ == "__main__":
    main()
```

## 7. 常用单位

| 单位 | 用法 |
|---|---|
| `Cm(1)` | 厘米，适合中文汇报 PPT 排版 |
| `Inches(1)` | 英寸 |
| `Pt(12)` | 字号 |
| `RGBColor(0xFF, 0xFF, 0xFF)` | RGB 颜色 |

16:9 宽屏尺寸：

```python
prs.slide_width = Cm(33.867)
prs.slide_height = Cm(19.05)
```

## 8. 限制速查

| 项目 | 说明 |
|---|---|
| SmartArt | 不保证完整读取或修改 |
| 图表 | 修改有限，结构复杂 |
| 组合形状 | 遍历需递归处理 group shape |
| 备注页 | `slide.notes_slide.notes_text_frame.text` 读取 |
| 嵌入对象 | 不保证完整读取 |
