# Aspose.Words for Java - Save a Document

---

**URL:** https://docs.aspose.com/words/net/save-a-document.md

**Contents:**
- Save to a File
- Save to a Stream
- Send a Document to a Client Browser

---
title: "Save a Document"
---

To save a document Aspose.Words provides the [Save](https://reference.aspose.com/words/java/com.aspose.words/document/#save-java.lang.String) method. There are overloads that allow saving a document to a file, stream, or ASPJava HttpResponse object for sending to a client browser. The document can be saved in any [save format](https://reference.aspose.com/words/java/com.aspose.words/saveformat/) supported by Aspose.Words.

**Save to a File**

Simply use the [Save](https://reference.aspose.com/words/java/com.aspose.words/document/#save-java.lang.String) method with a file name. Aspose.Words will determine the save format from the file extension that you specify.

```java
Document doc = new Document("Document.doc");

doc.save("BaseConversions.DocToDocx.docx");
```

**Save to a Stream**

Pass a stream object to the [Save](https://reference.aspose.com/words/java/com.aspose.words/document/#save-java.lang.String) method. It's necessary to specify the save format explicitly when saving to a stream.

```java
FileInputStream stream = new FileInputStream("Document.docx");

Document doc = new Document(stream);
// You can close the stream now, it is no longer needed because the document is in memory.
stream.close();

// ... do something with the document.

// Convert the document to a different format and save to stream.
ByteArrayOutputStream dstStream = new ByteArrayOutputStream();
doc.save(dstStream, SaveFormat.RTF);
```

**Save to PCL**

Aspose.Words supports saving a document into PCL (Printer Command Language). Aspose.Words can save documents into PCL 6 (PCL 6 Enhanced or PCL XL) format. The `PclSaveOptions` class can be used to specify additional options when saving a document into the PCL format.

The following code example shows how to save a document to PCL using save options:

```java
Document doc = new Document("Rendering.docx");

PclSaveOptions saveOptions = new PclSaveOptions();
saveOptions.setSaveFormat(SaveFormat.PCL);
saveOptions.setRasterizeTransformedElements(false);

doc.save("WorkingWithPclSaveOptions.RasterizeTransformedElements.pcl", saveOptions);
```

---


**URL:** https://docs.aspose.com/words/java/specify-save-options.md

---
title: "Specify Save Options"
---

When saving a document, you can set some advanced properties. Aspose.Words provides you with the [SaveOptions](https://reference.aspose.com/words/java/com.aspose.words/saveoptions/) class, which allows more precise control of the save process. There are overloads of the **Save** method that accept a **SaveOptions** object – it should be an object of a class derived from the **SaveOptions** class.

Each save format has a corresponding class that holds save options for this save format, for example, there is [PdfSaveOptions](https://reference.aspose.com/words/java/com.aspose.words/pdfsaveoptions/) for saving to PDF format, [MarkdownSaveOptions](https://reference.aspose.com/words/java/com.aspose.words/markdownsaveoptions/) for saving to Markdown format, or [ImageSaveOptions](https://reference.aspose.com/words/java/com.aspose.words/imagesaveoptions/) for saving to an image. This article provides examples of working with some options classes derived from **SaveOptions**.

Set the save options before saving the document into HTML:

```java
Document doc = new Document("Rendering.docx");

File imagesDir = new File(Paths.get(getArtifactsDir(), "Images").toString());

// The folder specified needs to exist and should be empty.
if (imagesDir.exists())
    imagesDir.delete();

imagesDir.mkdir();

// Set an option to export form fields as plain text, not as HTML input elements.
HtmlSaveOptions saveOptions = new HtmlSaveOptions(SaveFormat.HTML);
saveOptions.setExportTextInputFormFieldAsText(true);
saveOptions.setImagesFolder(imagesDir.getPath());

doc.save("WorkingWithHtmlSaveOptions.ExportTextInputFormFieldAsText.html", saveOptions);
```

Set a password to encrypt a document using the RC4 encryption method:

```java
Document doc = new Document();
DocumentBuilder builder = new DocumentBuilder(doc);

builder.write("Hello world!");

DocSaveOptions saveOptions = new DocSaveOptions();
saveOptions.setPassword("password");

doc.save("WorkingWithDocSaveOptions.EncryptDocumentWithPassword.docx", saveOptions);
```

Load and save OpenDocument encrypted with a password:

```java
Document doc = new Document("Encrypted.docx", new LoadOptions("docPassword"));

doc.save("WorkingWithLoadOptions.LoadAndSaveEncryptedOdt.odt", new OdtSaveOptions("newPassword"));
```

Not all formats support encryption and the use of **Password** property.

Update the document creation time:

```java
Document doc = new Document("Rendering.docx");

PdfSaveOptions saveOptions = new PdfSaveOptions();
saveOptions.setUpdateLastPrintedProperty(true);

doc.save("WorkingWithPdfSaveOptions.UpdateLastPrinted.pdf", saveOptions);
```

Save a black and white image with one bit per pixel format:

```java
Document doc = new Document("Rendering.docx");

ImageSaveOptions saveOptions = new ImageSaveOptions(SaveFormat.PNG);
saveOptions.setPageSet(new PageSet(1));
saveOptions.setImageColorMode(ImageColorMode.BLACK_AND_WHITE);
saveOptions.setPixelFormat(ImagePixelFormat.FORMAT_1_BPP_INDEXED);

doc.save("WorkingWithImageSaveOptions.Format1BppIndexed.Png", saveOptions);
```

---