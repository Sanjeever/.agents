# Aspose.Words for Java - Create or Load a Document

---

**URL:** https://docs.aspose.com/words/java/create-or-load-a-document.md

**Contents:**
- Create a New Document
- Load from a File
- Load from a Stream

---
title: "Create or Load a Document"
---

The [Document](https://reference.aspose.com/words/java/com.aspose.words/document/) class represents a document loaded into memory. The document has several overloaded constructors allowing you to create a blank document or to load it from a file or stream. The document can be loaded in any [LoadFormat](https://reference.aspose.com/words/java/com.aspose.words/loadformat/) supported by Aspose.Words.

**Create a New Document**

To create a new blank document call the [Document](https://reference.aspose.com/words/java/com.aspose.words/document/) constructor without parameters. To generate a document programmatically, use the [DocumentBuilder](https://reference.aspose.com/words/java/com.aspose.words/documentbuilder) class to add document contents.

```java
Document doc = new Document();

// Use a document builder to add content to the document.
DocumentBuilder builder = new DocumentBuilder(doc);
builder.writeln("Hello World!");

doc.save("AddContentUsingDocumentBuilder.CreateNewDocument.docx");
```

Default values:

- A blank document contains one section with default parameters, one empty paragraph, some document styles. Actually this document is the same as the result of creating the “New document” in Microsoft Word.
- The document paper size is [PaperSize](https://reference.aspose.com/words/java/com.aspose.words/papersize/).**Letter**.

**Load from a File**

Pass a file name as string to the Document constructor to open an existing document from a file.

```java
Document doc = new Document("Document.docx");
```

**Load from a Stream**

To open a document from a stream, simply pass a stream object that contains the document into the Document constructor.

```java
FileInputStream stream = new FileInputStream("Document.docx");

Document doc = new Document(stream);
// You can close the stream now, it is no longer needed because the document is in memory.
stream.close();
```

---


**URL:** https://docs.aspose.com/words/java/specify-load-options.md

**Contents:**
- Set Microsoft Word Version
- Set Language Preferences
- Use WarningCallback to Control Problems While Loading a Document
- Use ResourceLoadingCallback to Control the External Resources Loading
- Use TempFolder to Avoid a Memory Exception
- Set the Encoding Explicitly
- Load Encrypted Documents

---
title: "Specify Load Options"
---

When loading a document, you can set some advanced properties with the [LoadOptions](https://reference.aspose.com/words/java/com.aspose.words/loadoptions/) class. Some load formats have a corresponding class that holds load options for this load format, for example, there is [PdfLoadOptions](https://reference.aspose.com/words/java/com.aspose.words/pdfloadoptions/) for loading to PDF format or [TxtLoadOptions](https://reference.aspose.com/words/java/com.aspose.words/txtloadoptions/) for loading to TXT.

**Set Microsoft Word Version**

Different versions of Microsoft Word application can display documents in differently. Essential document markup elements may be missing or may be interpreted differently causing Microsoft Word 2019 to show such a document differently compared to Microsoft Word 2010.

By default Aspose.Words opens documents using Microsoft Word 2019 rules. You can explicitly specify the desired version using the [MswVersion](https://reference.aspose.com/words/java/com.aspose.words/loadoptions/#getMswVersion) property:

```java
LoadOptions loadOptions = new LoadOptions();
loadOptions.setMswVersion(MsWordVersion.WORD_2010);

Document doc = new Document("Document.docx", loadOptions);

doc.save("WorkingWithLoadOptions.SetMsWordVersion.docx");
```

**Set Language Preferences**

Microsoft Word may show documents differently depending on the "Office Language Preferences" dialog settings, that can be found in "File → Options → Languаge". Aspose.Words provides the [LanguagePreferences](https://reference.aspose.com/words/java/com.aspose.words/loadoptions/#getLanguagePreferences) property as the equivalent of this dialog. If Aspose.Words output differs from the Microsoft Word output, set the appropriate value for **EditingLanguage** – this can improve the output document:

```java
LoadOptions loadOptions = new LoadOptions();
// Set language preferences that will be used when document is loading.
loadOptions.getLanguagePreferences().addEditingLanguage(EditingLanguage.JAPANESE);

Document doc = new Document("No default editing language.docx", loadOptions);
```

**Use WarningCallback to Control Problems While Loading a Document**

 If you want to know about problems that occurred while loading a document, Aspose.Words provides the [IWarningCallback](https://reference.aspose.com/words/java/com.aspose.words/iwarningcallback/) interface.

```java
public static class DocumentLoadingWarningCallback implements IWarningCallback {
    public void warning(WarningInfo info) {
        // Prints warnings and their details as they arise during document loading.
        System.out.println(MessageFormat.format("WARNING: {0}, source: {1}", info.getWarningType(), info.getSource()));
        System.out.println(MessageFormat.format("\tDescription: {0}", info.getDescription()));
    }
}
```

To get information about all problems throughout the load time, use the [WarningCallback](https://reference.aspose.com/words/java/com.aspose.words/loadoptions/#getWarningCallback) property.

```java
LoadOptions loadOptions = new LoadOptions();
loadOptions.setWarningCallback(new DocumentLoadingWarningCallback());

Document doc = new Document("Document.docx", loadOptions);
```

**Use ResourceLoadingCallback to Control the External Resources Loading**

A document may contain external links to images located somewhere on a local disk, network, or Internet. Aspose.Words automatically loads such images into a document, but there are situations when this process needs to be controlled. The [ResourceLoadingCallback](https://reference.aspose.com/words/java/com.aspose.words/loadoptions/#getResourceLoadingCallback) load option allows you to control this.

Implementthe [IResourceLoadingCallback](https://reference.aspose.com/words/java/com.aspose.words/iresourceloadingcallback/) interface:

```java
private static class HtmlLinkedResourceLoadingCallback implements IResourceLoadingCallback {
    public int resourceLoading(ResourceLoadingArgs args) throws Exception {
        switch (args.getResourceType()) {
            case ResourceType.CSS_STYLE_SHEET:
                System.out.println("External CSS Stylesheet found upon loading: " + args.getOriginalUri());
                // CSS file will don't used in the document.
                return ResourceLoadingAction.SKIP;

            case ResourceType.IMAGE:
                // Replaces all images with a substitute.
                BufferedImage newImage = ImageIO.read(new File(getImagesDir() + "Logo.jpg"));

                try (ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
                    ImageIO.write(newImage, "jpg", baos);
                    byte[] imageBytes = baos.toByteArray();
                    args.setData(imageBytes);
                }

                // New images will be used instead of presented in the document.
                return ResourceLoadingAction.USER_PROVIDED;

            case ResourceType.DOCUMENT:
                System.out.println("External document found upon loading: " + args.getOriginalUri());
                // Will be used as usual.
                return ResourceLoadingAction.DEFAULT;

            default:
                throw new IllegalArgumentException("Unexpected ResourceType value.");
        }
    }
}
```

Use the **ResourceLoadingCallback** property:

```java
LoadOptions loadOptions = new LoadOptions();
loadOptions.setResourceLoadingCallback(new HtmlLinkedResourceLoadingCallback());

Document doc = new Document("Images.html", loadOptions);
doc.save("WorkingWithLoadOptions.ResourceLoadingCallback.pdf");
```

**Use TempFolder to Avoid a Memory Exception**

Aspose.Words supports extremely large documents that have thousands of pages full of rich content. If you have a problem with Out of Memory exception while loading a document, try to use the [TempFolder](https://reference.aspose.com/words/java/com.aspose.words/loadoptions/#getTempFolder) property:

```java
LoadOptions loadOptions = new LoadOptions();
loadOptions.setTempFolder(getArtifactsDir());

Document doc = new Document("Document.docx", loadOptions);
```

**Set the Encoding Explicitly**

Aspose.Words tries to automatically detect the appropriate encoding by default, but in a rare case you may need to use an encoding different from the one detected by our encoding recognition algorithm. Use the [Encoding](https://reference.aspose.com/words/java/com.aspose.words/loadoptions/#getEncoding) property to get or set the encoding:

```java
LoadOptions loadOptions = new LoadOptions();
loadOptions.setEncoding(Charset.forName("US-ASCII"));

Document doc = new Document("English text.txt", loadOptions);
```

**Load Encrypted Documents**

You can load Word documents encrypted with a password. Use a special constructor overload, which accepts a [LoadOptions](https://reference.aspose.com/words/java/com.aspose.words/loadoptions/) object. This object contains the [Password](https://reference.aspose.com/words/java/com.aspose.words/loadoptions/#getPassword) property, which specifies the password string:

```java
Document doc = new Document("Encrypted.docx", new LoadOptions("docPassword"));
```

If you do not know in advance whether the file is encrypted, you can use the [FileFormatUtil](https://reference.aspose.com/words/java/com.aspose.words/fileformatutil/) class and the [IsEncrypted](https://reference.aspose.com/words/java/com.aspose.words/fileformatinfo/#isEncrypted) property:

```java
FileFormatInfo info = FileFormatUtil.detectFileFormat("Encrypted.docx");
System.out.println(info.isEncrypted());
```

---