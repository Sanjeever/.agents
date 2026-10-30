# Aspose.Words for Java - Conversion

---

**URL:** https://docs.aspose.com/words/java/convert-a-document-to-pdf.md

**Contents:**
- What is Document Conversion?
  - Convert DOCX to PDF
  - Convert DOCX to HTML
  - Convert DOCX to PNG
  - Convert DOCX to XLSX

---
title: "Convert From One Format to Another"
---

**What is Document Conversion?**

Aspose.Words can convert a document from any supported [LoadFormat](https://reference.aspose.com/words/java/com.aspose.words/loadformat/) into any supported [SaveFormat](https://reference.aspose.com/words/java/com.aspose.words/saveformat/). Conversion is the process of loading a document and then saving it in the target format.

**Convert DOCX to PDF**

```csharp
Document doc = new Document("Document.docx");
doc.save("BaseConversions.DocxToPdf.pdf");
```

**Convert DOCX to HTML**

```csharp
Document doc = new Document("Document.docx");

doc.save("BaseConversions.DocxToHtml.html");
```

**Convert DOCX to PNG**

```csharp
Document doc = new Document("Document.docx");
doc.save("BaseConversions.DocxToPng.png");
```

**Convert DOCX to XLSX**

```csharp
Document doc = new Document("Document.docx");
doc.save("BaseConversions.DocxToXlsx.xlsx");
```

---

**URL:** https://docs.aspose.com/words/java/convert-a-multi-page-document-to-an-image.md

---
title: "Convert a Multi-page Document to an Image"
---

Aspose.Words for Java allows users to export multi-page documents to raster images using the [MultiPageLayout](https://reference.aspose.com/words/java/com.aspose.words/multipagelayout/) class. This can be useful for generating previews, archives, or visual representations of documents for non-editable use.

Aspose.Words supports multi-page export to the following raster image formats:

* Jpeg
* Gif
* Png
* Bmp
* Tiff
* WebP

You can specify how the pages should be organized when saving to an image:

* [SinglePage](https://reference.aspose.com/words/java/com.aspose.words/multipagelayout/#singlePage) – save only the first page
* [Grid](https://reference.aspose.com/words/java/com.aspose.words/multipagelayout/#grid-int-float-float) – arrange the pages in a grid, left-to-right and top-to-bottom, while specifying the number of columns
* [Horizontal](https://reference.aspose.com/words/java/com.aspose.words/multipagelayout/#horizontal-float) – arrange the pages horizontally side-by-side, left-to-right, in a single output
* [Vertical](https://reference.aspose.com/words/java/com.aspose.words/multipagelayout/#vertical-float) – arrange the pages vertically one below the other in a single output
* [TiffFrames](https://reference.aspose.com/words/java/com.aspose.words/multipagelayout/#tiffFrames) – arrange each page as a separate frame in a multi-frame TIFF image, applies only to TIFF image formats

Save a multi-page DOCX document as JPEG image with Horizontal layout:

```csharp
Document doc = new Document("Rendering.docx");

ImageSaveOptions options = new ImageSaveOptions(SaveFormat.JPEG);
options.setPageLayout(MultiPageLayout.horizontal(10));

doc.save("WorkingWithImageSaveOptions.HorizontalLayout.jpg", options);
```

You can also customize the output file page appearance – specify [BackColor](https://reference.aspose.com/words/java/com.aspose.words/multipagelayout/#getBackColor), [BorderColor](https://reference.aspose.com/words/java/com.aspose.words/multipagelayout/#getBorderColor), and [BorderWidth](https://reference.aspose.com/words/java/com.aspose.words/multipagelayout/#getBorderWidth):

```csharp
Document doc = new Document("Rendering.docx");

ImageSaveOptions options = new ImageSaveOptions(SaveFormat.JPEG);
// Set up a grid layout with:
// - 3 columns per row.
// - 10pts spacing between pages (horizontal and vertical).
options.setPageLayout(MultiPageLayout.grid(3, 10, 10));

// Customize the background and border.
options.getPageLayout().setBackColor(Color.lightGray);
options.getPageLayout().setBorderColor(Color.blue);
options.getPageLayout().setBorderWidth(2);

doc.save("ImageSaveOptions.GridLayout.jpg", options);
```

---