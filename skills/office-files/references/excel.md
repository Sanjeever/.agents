# Excel（openpyxl / pandas）模板

依赖选择：
- 单元格级读写、保留样式与公式：`openpyxl`
- 表格数据分析、与 csv/parquet/dataframe 互转：`pandas`（默认随 `openpyxl` 自动用作 xlsx 引擎）

阅读本文件前，请确保已阅读 `../SKILL.md` 的通用工作流。

> **输入路径处理（必读）**：本文件示例多用 `WORKDIR = Path.cwd()` + `glob("*.xlsx")` 来发现文件。**仅在用户没有指定具体文件名时**才照搬这种写法。
> 若用户指定了具体文件，把 `WORKDIR + glob` 那两行换成 `INPUT_PATH = Path(r"C:\...\具体文件.xlsx")`，详见 `../SKILL.md` 的 Script Requirements。

---

## 1. 读取工作表列表和前 20 行

```python
# /// script
# requires-python = ">=3.10"
# dependencies = ["openpyxl"]
# ///
from pathlib import Path
import openpyxl

WORKDIR = Path.cwd()

def value_to_text(value) -> str:
    return "" if value is None else str(value)

def main():
    files = list(WORKDIR.glob("*.xlsx"))
    if not files:
        raise FileNotFoundError("未找到 xlsx 文件")

    input_path = files[0]

    wb = openpyxl.load_workbook(str(input_path), data_only=True, read_only=True)

    print(f"文件：{input_path.name}")
    print(f"工作表：{wb.sheetnames}")

    ws = wb.active
    print(f"当前工作表：{ws.title}")
    print(f"尺寸：{ws.dimensions}")
    print(f"最大行：{ws.max_row}，最大列：{ws.max_column}")

    for ri, row in enumerate(ws.iter_rows(values_only=True), start=1):
        print(f"R{ri}: " + " | ".join(value_to_text(c) for c in row))
        if ri >= 20:
            break

    wb.close()

if __name__ == "__main__":
    main()
```

## 2. 读取指定单元格和区域

```python
# /// script
# requires-python = ">=3.10"
# dependencies = ["openpyxl"]
# ///
from pathlib import Path
import openpyxl

WORKDIR = Path.cwd()
SHEET_NAME = None  # 例如："Sheet1"；为 None 时使用活动工作表

def main():
    files = list(WORKDIR.glob("*.xlsx"))
    if not files:
        raise FileNotFoundError("未找到 xlsx 文件")

    input_path = files[0]
    wb = openpyxl.load_workbook(str(input_path), data_only=True)
    ws = wb[SHEET_NAME] if SHEET_NAME else wb.active

    print(f"A1 = {ws['A1'].value}")

    for row in ws["A1:C10"]:
        print([cell.value for cell in row])

    wb.close()

if __name__ == "__main__":
    main()
```

## 3. 大文件流式读取

```python
# /// script
# requires-python = ">=3.10"
# dependencies = ["openpyxl"]
# ///
from pathlib import Path
import openpyxl

WORKDIR = Path.cwd()
MAX_ROWS = 100

def value_to_text(value) -> str:
    return "" if value is None else str(value)

def main():
    files = list(WORKDIR.glob("*.xlsx"))
    if not files:
        raise FileNotFoundError("未找到 xlsx 文件")

    input_path = files[0]
    wb = openpyxl.load_workbook(str(input_path), data_only=True, read_only=True)
    ws = wb.active

    count = 0
    for row in ws.iter_rows(values_only=True):
        print(" | ".join(value_to_text(c) for c in row))
        count += 1

        if count >= MAX_ROWS:
            print(f"... 已截断，仅显示前 {MAX_ROWS} 行；工作表最大行数：{ws.max_row}")
            break

    wb.close()

if __name__ == "__main__":
    main()
```

## 4. 写入单元格并另存

```python
# /// script
# requires-python = ">=3.10"
# dependencies = ["openpyxl"]
# ///
from pathlib import Path
import openpyxl

WORKDIR = Path.cwd()

def main():
    files = list(WORKDIR.glob("*.xlsx"))
    if not files:
        raise FileNotFoundError("未找到 xlsx 文件")

    input_path = files[0]
    output_path = input_path.with_name(input_path.stem + "_modified.xlsx")

    wb = openpyxl.load_workbook(str(input_path))
    ws = wb.active

    ws["A1"] = "Hello"
    ws.cell(row=2, column=1, value="World")

    values = ["A", "B", "C"]
    for ci, value in enumerate(values, start=1):
        ws.cell(row=3, column=ci, value=value)

    wb.save(str(output_path))
    wb.close()

    print(f"输出文件：{output_path}")

if __name__ == "__main__":
    main()
```

## 5. 添加公式

`openpyxl` 只写入公式，不计算结果。用户用 Excel 打开并保存后，公式缓存才会更新。

```python
# /// script
# requires-python = ">=3.10"
# dependencies = ["openpyxl"]
# ///
from pathlib import Path
import openpyxl

WORKDIR = Path.cwd()

def main():
    files = list(WORKDIR.glob("*.xlsx"))
    if not files:
        raise FileNotFoundError("未找到 xlsx 文件")

    input_path = files[0]
    output_path = input_path.with_name(input_path.stem + "_formula.xlsx")

    wb = openpyxl.load_workbook(str(input_path))
    ws = wb.active

    ws["C1"] = "=A1+B1"
    ws["D1"] = "=SUM(A1:A10)"

    wb.save(str(output_path))
    wb.close()

    print(f"已写入公式：{output_path}")
    print("注意：openpyxl 不计算公式结果，请用 Excel 打开并保存以刷新公式缓存。")

if __name__ == "__main__":
    main()
```

## 6. 创建新 Excel 文件

```python
# /// script
# requires-python = ">=3.10"
# dependencies = ["openpyxl"]
# ///
from pathlib import Path
import openpyxl
from openpyxl.styles import Font, Alignment

WORKDIR = Path.cwd()

def main():
    output_path = WORKDIR / "output.xlsx"

    wb = openpyxl.Workbook()
    ws = wb.active
    ws.title = "数据表"

    headers = ["名称", "数值", "备注"]
    for ci, header in enumerate(headers, start=1):
        cell = ws.cell(row=1, column=ci, value=header)
        cell.font = Font(bold=True)
        cell.alignment = Alignment(horizontal="center")

    data = [
        ["事项 A", 100, "说明 A"],
        ["事项 B", 200, "说明 B"],
    ]

    for ri, row_data in enumerate(data, start=2):
        for ci, value in enumerate(row_data, start=1):
            ws.cell(row=ri, column=ci, value=value)

    ws2 = wb.create_sheet("说明")
    ws2["A1"] = "这里是说明页。"

    wb.save(str(output_path))
    wb.close()

    print(f"已创建：{output_path}")

if __name__ == "__main__":
    main()
```

## 7. xlsx ↔ pandas（推荐用于数据分析）

pandas 读 xlsx 需要 `openpyxl` 作为引擎。返回的是 DataFrame，可以直接做筛选、分组、统计、与 csv/parquet 互转。

```python
# /// script
# requires-python = ">=3.10"
# dependencies = ["pandas", "openpyxl"]
# ///
from pathlib import Path
import pandas as pd

WORKDIR = Path.cwd()

def main():
    files = list(WORKDIR.glob("*.xlsx"))
    if not files:
        raise FileNotFoundError("未找到 xlsx 文件")

    input_path = files[0]

    # 读取（默认读第一个工作表，sheet_name=None 读全部）
    df = pd.read_excel(input_path, sheet_name=0)
    print(f"形状：{df.shape}")
    print(df.head())

    # 简单分析示例
    print("\n数值列描述：")
    print(df.describe())

    # 写回 xlsx（多 sheet 用 ExcelWriter）
    output_path = input_path.with_name(input_path.stem + "_analyzed.xlsx")
    with pd.ExcelWriter(output_path, engine="openpyxl") as writer:
        df.to_excel(writer, sheet_name="原始数据", index=False)
        df.describe().to_excel(writer, sheet_name="统计")
    print(f"\n输出：{output_path}")

if __name__ == "__main__":
    main()
```

## 8. xlsx → csv（每个工作表导出一个 csv）

```python
# /// script
# requires-python = ">=3.10"
# dependencies = ["pandas", "openpyxl"]
# ///
from pathlib import Path
import pandas as pd

WORKDIR = Path.cwd()

def main():
    files = list(WORKDIR.glob("*.xlsx"))
    if not files:
        raise FileNotFoundError("未找到 xlsx 文件")

    input_path = files[0]
    sheets = pd.read_excel(input_path, sheet_name=None)  # dict[str, DataFrame]

    for name, df in sheets.items():
        out = WORKDIR / f"{input_path.stem}__{name}.csv"
        df.to_csv(out, index=False, encoding="utf-8-sig")  # utf-8-sig 保证 Excel 双击不乱码
        print(f"导出：{out}（{len(df)} 行）")

if __name__ == "__main__":
    main()
```

## 9. csv → xlsx

```python
# /// script
# requires-python = ">=3.10"
# dependencies = ["pandas", "openpyxl"]
# ///
from pathlib import Path
import pandas as pd

WORKDIR = Path.cwd()

def main():
    files = list(WORKDIR.glob("*.csv"))
    if not files:
        raise FileNotFoundError("未找到 csv 文件")

    input_path = files[0]
    output_path = input_path.with_suffix(".xlsx")

    df = pd.read_csv(input_path)
    df.to_excel(output_path, index=False, engine="openpyxl")
    print(f"输出：{output_path}（{len(df)} 行）")

if __name__ == "__main__":
    main()
```

## 10. 参数与库选择速查

| 参数 | 推荐值 | 说明 |
|---|---|---|
| `data_only=True` | 读取展示值时 | 返回公式缓存值 |
| `data_only=False` | 需要查看公式时 | 返回公式字符串 |
| `read_only=True` | 大文件读取时 | 流式，低内存，不能修改 |
| `read_only=False` | 需要修改时 | 加载完整工作簿 |

| 场景 | 用哪个库 |
|---|---|
| 单元格级精确读写、保留格式 / 公式 | openpyxl |
| 表格数据分析（筛选、分组、统计） | pandas |
| 与 csv / parquet / json 互转 | pandas |
| 创建带复杂样式的报表 | openpyxl（pandas 写出后可再用 openpyxl 微调） |
| 超大文件流式遍历 | openpyxl `read_only=True` |
