# Aspose.Words for Java - Add Watermark

---

**URL:** https://docs.aspose.com/words/java/working-with-watermark.md

**Contents:**
- Adding an Image Watermark
- Adding a Text Watermark
- Remove Watermark from a Document

---
title: "Add Watermark"
---

A watermark is a background image that displays behind the text in a document. A watermark can contain a text or an image represented by the [Watermark](https://reference.aspose.com/words/java/com.aspose.words/watermark/) class.

**Add a Watermark to a Document**

Aspose.Words provides the [WatermarkType](https://reference.aspose.com/words/java/com.aspose.words/watermarktype/)enumeration defining three possible types of watermarks (Text, Image, and None) to work with. 

**Insert a text watermark** in a document by defining [TextWatermarkOptions](https://reference.aspose.com/words/java/com.aspose.words/textwatermarkoptions/) using the [SetText](https://reference.aspose.com/words/java/com.aspose.words/watermark/#setText-java.lang.String) method:

```java
Document doc = new Document("Document.docx");

TextWatermarkOptions options = new TextWatermarkOptions();
options.setFontFamily("Arial");
options.setFontSize(36f);
options.setColor(Color.BLACK);
options.setLayout(WatermarkLayout.HORIZONTAL);
options.isSemitrasparent(false);

doc.getWatermark().setText("Test", options);

doc.save("WorkWithWatermark.AddTextWatermark.docx");
```

**Insert an image watermark** in a document by defining [ImageWatermarkOptions](https://reference.aspose.com/words/java/com.aspose.words/imagewatermarkoptions/) using the [SetImage](https://reference.aspose.com/words/java/com.aspose.words/watermark/#setImage-java.awt.image.BufferedImage) method:

```java
Document doc = new Document("Document.docx");

ImageWatermarkOptions options = new ImageWatermarkOptions();
{
    options.setScale(5.0);
    options.isWashout(false);
}

doc.getWatermark().setImage(getImagesDir() + "Transparent background logo.png", options);

doc.save("WorkWithWatermark.AddImageWatermark.docx");
```

Image watermark can be inserted as image, string, or stream.

The watermark can also be inserted using shape class as well. It is very easy to insert any shape or image into a header or footer and thus create a watermark of any imaginable type.

**Remove Watermark from a Document**

The [Watermark](https://reference.aspose.com/words/java/com.aspose.words/watermark/) class provides the remove method to remove the watermark from a document.

```java
Document doc = new Document();

// Add a plain text watermark.
doc.getWatermark().setText("Aspose Watermark");

// If we wish to edit the text formatting using it as a watermark,
// we can do so by passing a TextWatermarkOptions object when creating the watermark.
TextWatermarkOptions textWatermarkOptions = new TextWatermarkOptions();
textWatermarkOptions.setFontFamily("Arial");
textWatermarkOptions.setFontSize(36f);
textWatermarkOptions.setColor(Color.BLACK);
textWatermarkOptions.setLayout(WatermarkLayout.DIAGONAL);
textWatermarkOptions.isSemitrasparent(false);

doc.getWatermark().setText("Aspose Watermark", textWatermarkOptions);

doc.save("Document.TextWatermark.docx");

// We can remove a watermark from a document like this.
if (doc.getWatermark().getType() == WatermarkType.TEXT)
    doc.getWatermark().remove();

doc.save("WorkWithWatermark.RemoveDocumentWatermark.docx");
```

---