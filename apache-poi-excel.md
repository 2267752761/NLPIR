好的 👍，我们单独聚焦 **Apache POI 的 Excel 处理功能**，做一个深入的讲解，涵盖从 **读写核心对象 → 样式 → 公式 → 图表 → 大文件处理 → 高级特性**。

---

# 📝 Apache POI Excel 处理功能详解

## 1. Excel 工作簿模型

Excel 的核心对象层次结构如下：

* **Workbook（工作簿）**

  * `HSSFWorkbook` → 处理 **.xls** (Excel 97-2003)
  * `XSSFWorkbook` → 处理 **.xlsx** (Excel 2007+)
  * `SXSSFWorkbook` → 流式写出大数据量 .xlsx

* **Sheet（工作表）**

  * `Workbook.createSheet("Sheet1")`
  * 提供行列管理、冻结窗口、分页符、自动筛选等功能

* **Row（行）**

  * `Sheet.createRow(int rowIndex)`
  * 可设置高度、行样式

* **Cell（单元格）**

  * `Row.createCell(int columnIndex)`
  * 支持的数据类型：

    * `STRING` → 文本
    * `NUMERIC` → 数字（整型/浮点型/日期）
    * `BOOLEAN` → 布尔值
    * `FORMULA` → 公式
    * `BLANK` → 空白

---

## 2. 样式与字体（CellStyle + Font）

Excel 的格式化能力非常强，POI 提供完整 API：

* **字体**

  * 字体类型、大小、粗体、斜体、下划线、颜色
  * `Font font = workbook.createFont(); font.setBold(true);`

* **单元格样式**

  * 背景色 / 填充模式
  * 边框（上/下/左/右，粗细、颜色）
  * 水平/垂直对齐方式
  * 自动换行、缩进、旋转

* **数据格式**

  * 内置：`#,##0`, `0.00%`, `yyyy-MM-dd`
  * 自定义：`"￥"#,##0.00`

```java
CellStyle style = workbook.createCellStyle();
style.setDataFormat(workbook.createDataFormat().getFormat("yyyy-MM-dd"));
```

---

## 3. 公式与计算

POI 支持 Excel 公式，核心类是 **FormulaEvaluator**。

* **设置公式**

  ```java
  cell.setCellFormula("SUM(A1:A10)");
  ```

* **计算公式**

  ```java
  FormulaEvaluator evaluator = workbook.getCreationHelper().createFormulaEvaluator();
  evaluator.evaluateFormulaCell(cell);
  ```

* **常见支持公式**
  SUM、IF、VLOOKUP、CONCATENATE、ROUND、NOW、TODAY …（Excel 原生大部分公式都支持）

---

## 4. 批注与超链接

* **批注（Comment）**

  ```java
  CreationHelper factory = workbook.getCreationHelper();
  Drawing<?> drawing = sheet.createDrawingPatriarch();
  ClientAnchor anchor = factory.createClientAnchor();
  Comment comment = drawing.createCellComment(anchor);
  comment.setString(factory.createRichTextString("这是一条批注"));
  cell.setCellComment(comment);
  ```

* **超链接**

  ```java
  Hyperlink link = factory.createHyperlink(HyperlinkType.URL);
  link.setAddress("https://apache.org");
  cell.setHyperlink(link);
  ```

---

## 5. 图表（Charts, XSSF + XDDF）

Excel 图表处理主要依赖 **XSSF** (xlsx) + **XDDF API**。

* 支持类型：

  * 柱状图（BarChart）、折线图（LineChart）、饼图（PieChart）、散点图（ScatterChart）、面积图（AreaChart）
* API：`org.apache.poi.xddf.usermodel.chart.*`

示例（折线图）：

```java
XSSFDrawing drawing = sheet.createDrawingPatriarch();
XSSFClientAnchor anchor = drawing.createAnchor(0, 0, 0, 0, 0, 5, 10, 20);
XSSFChart chart = drawing.createChart(anchor);
XDDFChartLegend legend = chart.getOrAddLegend();
legend.setPosition(LegendPosition.BOTTOM);
```

---

## 6. 数据透视表（Pivot Table, XSSF only）

* 仅支持 `.xlsx` 格式
* 使用 `XSSFPivotTable` API 创建

```java
XSSFPivotTable pivotTable = sheet.createPivotTable(source, position);
pivotTable.addRowLabel(0); // 行字段
pivotTable.addColumnLabel(DataConsolidateFunction.SUM, 1); // 数据汇总
```

---

## 7. 大文件处理（Streaming）

处理几十 MB 或百万行数据时，内存占用会爆炸。POI 提供流式处理：

* **SXSSFWorkbook** → 写入大数据量（只保留窗口内行，其他 flush 到磁盘）

  * 适合导出百万行报表
* **StreamingReader（第三方扩展）** → 事件驱动读取 `.xlsx`（基于 StAX）

  * 内存占用极低，适合导入大文件

---

## 8. 其它功能

* **合并单元格** → `sheet.addMergedRegion(new CellRangeAddress(...))`
* **冻结窗格** → `sheet.createFreezePane(colSplit, rowSplit)`
* **分页符** → `sheet.setRowBreak(rowIndex)`
* **打印设置** → 页边距、页眉页脚、缩放比例

---

## 9. 使用场景总结

1. **企业报表导出**（带样式、公式、图表、透视表）
2. **批量数据导入导出**（对接数据库）
3. **Excel 自动化操作**（计算、格式化、报表生成）
4. **大数据量 Excel 文件处理**（SXSSF/StreamingReader）

---

👉 总结一句话：
**POI Excel 模块提供了从“简单数据导入导出”到“复杂报表生成（含公式、图表、透视表）”的全链路功能，唯一的限制就是性能与内存占用，需要在大文件场景下结合 SXSSF 或 StreamingReader。**

---
