# Aspose.Words for Java - Split Large Documents

---

**URL:** https://docs.aspose.com/words/java/split-a-document.md

**Contents:**
- Split a Document by Headings
- Split by Sections
- Split a Document Page by Page
- Split a Document by Page Ranges
- Callback Option to Save a Document

---
title: "Split Large Documents"
---

Aspose.Words provides you with an efficient way to split one document into multiple documents by headings or sections. You can also split a document by pages or by page ranges. Both splitting options will be described in this article.

To split a document into smaller files using Aspose.Words, you need to follow these steps:

1. Load the document in any supported format.
1. Split the document.
1. Save the output documents.

You can specify criteria using the [DocumentSplitCriteria](https://reference.aspose.com/words/java/com.aspose.words/documentsplitcriteria/) enumeration. So you can divide a document into chapters using one of the following criteria or combine more than one criteria together:

- heading paragraph,
- section break,
- column break,
- page break.

**Split a Document by Headings**

To split a document into chapters by headings, use the **HeadingParagraph** value of the **DocumentSplitCriteria** property.

If you need to split a document by a specific level of heading paragraphs, such as headings 1, 2, and 3, use also the [DocumentSplitHeadingLevel](https://reference.aspose.com/words/java/com.aspose.words/htmlsaveoptions/#getDocumentSplitHeadingLevel) property. The output will be divided by paragraphs formatted with the specified heading level.

```java
Document doc = new Document("Rendering.docx");

HtmlSaveOptions options = new HtmlSaveOptions();
// Split a document into smaller parts, in this instance split by heading.
options.setDocumentSplitCriteria(DocumentSplitCriteria.HEADING_PARAGRAPH);

doc.save("SplitDocument.ByHeadings.epub", options);
```

Please note that for this criteria, Aspose.Words only supports saving to HTML format when splitting.

When saving to EPUB, the document is not split into several files, and there will be only one output file.

**Split a Document by Sections**

Aspose.Words also enables you to use section breaks to split documents and save them to HTML. For this purpose, use **SectionBreak** as the **DocumentSplitCriteria**:

```java
HtmlSaveOptions options = new HtmlSaveOptions();
options.setDocumentSplitCriteria(DocumentSplitCriteria.SECTION_BREAK);
```

There is another way to split the source document into multiple output documents, and you can choose any output format supported by Aspose.Words:

```java
Document doc = new Document("Big document.docx");

for (int i = 0; i < doc.getSections().getCount(); i++) {
    // Split a document into smaller parts, in this instance, split by section.
    Section section = doc.getSections().get(i).deepClone();

    Document newDoc = new Document();
    newDoc.getSections().clear();

    Section newSection = (Section) newDoc.importNode(section, true);
    newDoc.getSections().add(newSection);

    // Save each section as a separate document.
    newDoc.save(MessageFormat.format("SplitDocument.BySections_{0}.docx", i));
```

**Split a Document Page by Page**

Aspose.Words enables you to split a multi-page document page by page using the [ExtractPages](https://reference.aspose.com/words/java/com.aspose.words/document/#extractPages-int-int) method.

```java
Document doc = new Document("Big document.docx");

int pageCount = doc.getPageCount();

for (int page = 0; page < pageCount; page++) {
    // Save each page as a separate document.
    Document extractedPage = doc.extractPages(page, 1);
    extractedPage.save(MessageFormat.format("SplitDocument.PageByPage_{0}.docx", page + 1));
}
```

**Split a Document by Page Ranges**

Aspose.Words allows splitting a multi-page document by page ranges. You can split one file into multiple files with various page ranges or just select one range and save only this part of the source document. Note that you can choose the page range according to the maximum and minimum page number of a document.

```java
Document doc = new Document("Big document.docx");

// Get part of the document.
Document extractedPages = doc.extractPages(3, 6);
extractedPages.save("SplitDocument.ByPageRange.docx");
```

**Callback Option to Save a Document**

You can use the [DocumentPartSavingCallback](https://reference.aspose.com/words/java/com.aspose.words/htmlsaveoptions/#getDocumentPartSavingCallback) property to control how Aspose.Words saves document parts when this document is exported into HTML format. This property allows you to rename output files or even to redirect them into custom streams.

Please note that this callback is not useful when saving to EPUB because all output parts must be saved into a single container – the .epub file. So, stream redirection is not supported, and the effect of renaming is not visible since files are renamed inside the container.

**Merge the Split Document with Another Document**

Aspose.Words enables you to merge the output split document with another document to form a new document. This can be called document merging.

```java
private void mergeDocuments() throws Exception {
    // Find documents using for merge.
    File directory = new File(getArtifactsDir());
    Collection<File> documentPaths = FileUtils.listFiles(directory, new WildcardFileFilter("SplitDocument.PageByPage_*.docx"), null);

    String sourceDocumentPath =
            FileUtils.getFile(getArtifactsDir(), "SplitDocument.PageByPage_1.docx").getPath();

    // Open the first part of the resulting document.
    Document sourceDoc = new Document(sourceDocumentPath);

    // Create a new resulting document.
    Document mergedDoc = new Document();
    DocumentBuilder mergedDocBuilder = new DocumentBuilder(mergedDoc);

    // Merge document parts one by one.
    for (File documentPath : documentPaths) {
        if (documentPath.getName().equals(sourceDocumentPath))
            continue;

        mergedDocBuilder.moveToDocumentEnd();
        mergedDocBuilder.insertDocument(sourceDoc, ImportFormatMode.KEEP_SOURCE_FORMATTING);
        sourceDoc = new Document(documentPath.getPath());
    }

    mergedDoc.save("SplitDocument.MergeDocuments.docx");
}
```

---