# Aspose.Words for Java - Merge Documents

---

**URL:** https://docs.aspose.com/words/java/insert-and-append-documents.md

**Contents:**
- Insert a Document
- Insert a Document During Find and Replace Operation
- Insert a Document During Mail Merge Operation
- Insert a Document at Bookmark
- Append a Document

---
title: "Insert and Append Documents"
---

The insert operation allows you to insert the content of previously created documents into a new or existing one.

In turn, the append feature allows you to add a document only at the end of another document.

**Insert a Document**

You can insert documents in a variety of locations in different ways. For example, you can insert a document through a replace operation, a merge field during a merge operation, or via a bookmark.

You can also use the [InsertDocument](https://reference.aspose.com/words/java/com.aspose.words/documentbuilder/#insertDocument-com.aspose.words.Document-int) or the [InsertDocumentInline](https://reference.aspose.com/words/java/com.aspose.words/documentbuilder/#insertDocumentInline-com.aspose.words.Document-int-com.aspose.words.ImportFormatOptions) method, which is similar to inserting a document in Microsoft Word, to insert a whole document at the current cursor position without any previous importing.

```java
Document srcDoc = new Document("Document source.docx");
Document dstDoc = new Document("Northwind traders.docx");
DocumentBuilder builder = new DocumentBuilder(dstDoc);

builder.moveToDocumentEnd();
builder.insertBreak(BreakType.PAGE_BREAK);

builder.insertDocument(srcDoc, ImportFormatMode.KEEP_SOURCE_FORMATTING);
builder.getDocument().save("JoinAndAppendDocuments.insertDocument.docx");
```

```java
DocumentBuilder srcDoc = new DocumentBuilder();
srcDoc.write("[src content]");

// Create destination document.
DocumentBuilder dstDoc = new DocumentBuilder();
dstDoc.write("Before ");
dstDoc.insertNode(new BookmarkStart(dstDoc.getDocument(), "src_place"));
dstDoc.insertNode(new BookmarkEnd(dstDoc.getDocument(), "src_place"));
dstDoc.write(" after");

Assert.assertEquals("Before  after", dstDoc.getDocument().getText().trim());

// Insert source document into destination inline.
dstDoc.moveToBookmark("src_place");
dstDoc.insertDocumentInline(srcDoc.getDocument(), ImportFormatMode.USE_DESTINATION_STYLES, new ImportFormatOptions());

Assert.assertEquals("Before [src content] after", dstDoc.getDocument().getText().trim());
```

**Insert a Document During Find and Replace Operation**

You can insert documents while performing find and replace operations. For example, a document can contain paragraphs with the text `INTRODUCTION` and `CONCLUSION]`. But in the final document, you need to replace those paragraphs with the content obtained from another external document. To achieve that, you will need to create a handler for the replace event.

```java
private static class InsertDocumentAtReplaceHandler implements IReplacingCallback {
    public int replacing(ReplacingArgs args) throws Exception {
        Document subDoc = new Document("Document insertion 2.docx");

        // Insert a document after the paragraph, containing the match text.
        Paragraph para = (Paragraph) args.getMatchNode().getParentNode();
        insertDocument(para, subDoc);

        // Remove the paragraph with the match text.
        para.remove();
        return ReplaceAction.SKIP;
    }
}
```

```java
Document mainDoc = new Document("Document insertion 1.docx");

FindReplaceOptions options = new FindReplaceOptions();
options.setDirection(FindReplaceDirection.BACKWARD);
options.setReplacingCallback(new InsertDocumentAtReplaceHandler());

mainDoc.getRange().replace(Pattern.compile("\\[MY_DOCUMENT\\]"), "", options);
mainDoc.save("CloneAndCombineDocuments.InsertDocumentAtReplace.docx");
```

**Insert a Document During Mail Merge Operation**

You can insert a document into a merge field during a Mail Merge operation. For example, a Mail Merge template can contain a merge field such as `Summary`. But in the final document, you need to insert content obtained from another external document into this merge field. To achieve that, you will need to create a handler for the merge event.

```java
private static class InsertDocumentAtMailMergeHandler implements IFieldMergingCallback {
    // This handler makes special processing for the "Document_1" field.
    // The field value contains the path to load the document. 
    // We load the document and insert it into the current merge field.
    public void fieldMerging(FieldMergingArgs args) throws Exception {
        if ("Document_1".equals(args.getDocumentFieldName())) {
            // Use document builder to navigate to the merge field with the specified name.
            DocumentBuilder builder = new DocumentBuilder(args.getDocument());
            builder.moveToMergeField(args.getDocumentFieldName());

            // The name of the document to load and insert is stored in the field value.
            Document subDoc = new Document((String) args.getFieldValue());

            insertDocument(builder.getCurrentParagraph(), subDoc);

            // The paragraph that contained the merge field might be empty now, and you probably want to delete it.
            if (!builder.getCurrentParagraph().hasChildNodes())
                builder.getCurrentParagraph().remove();

            // Indicate to the mail merge engine that we have inserted what we wanted.
            args.setText(null);
        }
    }

    public void imageFieldMerging(ImageFieldMergingArgs args) {
        // Do nothing.
    }
}
```

```java
Document mainDoc = new Document("Document insertion 1.docx");

mainDoc.getMailMerge().setFieldMergingCallback(new InsertDocumentAtMailMergeHandler());
// The main document has a merge field in it called "Document_1".
// The corresponding data for this field contains a fully qualified path to the document.
// That should be inserted to this field.
mainDoc.getMailMerge().execute(new String[]{"Document_1"}, new Object[]{"Document insertion 2.docx"});

mainDoc.save("CloneAndCombineDocuments.InsertDocumentAtMailMerge.doc");
```

**Insert a Document at Bookmark**

You can import a text file into a document and insert it right after a bookmark that you have defined in the document. To do this, create a bookmarked paragraph where you want the document to be inserted.

```java
Document mainDoc = new Document("Document insertion 1.docx");
Document subDoc = new Document("Document insertion 2.docx");

Bookmark bookmark = mainDoc.getRange().getBookmarks().get("insertionPlace");
insertDocument(bookmark.getBookmarkStart().getParentNode(), subDoc);

mainDoc.save("CloneAndCombineDocuments.InsertDocumentAtBookmark.docx");
```

Note that the bookmark should not enclose multiple paragraphs or text that you want them to appear in your final resulting document.

**Append a Document**

You may have a use case where you need to include additional pages from a document to the end of an existing document. To do this, you just need to call the [AppendDocument](https://reference.aspose.com/words/java/com.aspose.words/document/#appendDocument-com.aspose.words.Document-int) method to add a document to the end of another one.

```java
Document dstDoc = new Document();
dstDoc.getFirstSection().getBody().appendParagraph("Destination document text. ");

Document srcDoc = new Document();
srcDoc.getFirstSection().getBody().appendParagraph("Source document text. ");
// Append the source document to the destination document.
// Pass format mode to retain the original formatting of the source document when importing it.
dstDoc.appendDocument(srcDoc, ImportFormatMode.KEEP_SOURCE_FORMATTING);

dstDoc.save("JoinAndAppendDocuments.KeepSourceFormatting.docx");
```

---