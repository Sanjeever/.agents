# Aspose.Words for Java - Comparison

---

**URL:** https://docs.aspose.com/words/java/compare-documents.md

**Contents:**
- Limitations and Supported File Formats
- How to Compare Two Documents
- Specify Advanced Comparison Options

---
title: "Compare Documents"
---

Comparing documents is a process that identifies changes between two documents and contains the changes as revisions. This process compares any two documents, including versions of one specific document, then the changes between both documents will be shown as revisions in the first document.

The comparison method is achieved by comparing words at character level or at word level. If a word contains a change of at least one character, in the result, the difference will be displayed as a change of the entire word, not a character. This process of comparison is a usual task in the legal and financial industries.

**Limitations and Supported File Formats**

The general limitation for two documents being compared is that they must not have revisions before calling the compare method as this limitation exists in Microsoft Word.

You can compare any two documents within the [supported file formats](/words/java/supported-document-formats/). Basically, you can compare document objects and even you can create those objects from scratch without having any specific format.

**How to Compare Two Documents**

When you compare documents, differences of the latter document from the former show up as revisions to the former. When you modify a document, each edit will have its own revision after running the compare method.

Aspose.Words allows you to identify documents differences using the [Compare](https://reference.aspose.com/words/java/com.aspose.words/document/#compare-com.aspose.words.Document-java.lang.String-java.util.Date) method. It allows you to check documents or document versions to find differences and changes, including formatting modifications such as font changes, spacing changes, the addition of words and paragraphs.

As a result of comparison, documents can be determined as equal or not equal. The term “equal” documents means that the comparison method is not able to represent changes as revisions. This means that both document text and text formatting are the same. But there can be other differences between documents. For example, Microsoft Word supports only format revisions for styles, and you cannot represent style insertion/deletion. So documents can have a different set of styles, and the **Compare** method still produces no revisions.

```java
Document docA = new Document("Document.docx");
Document docB = docA.deepClone();

// DocA now contains changes as revisions.
docA.compare(docB, "user", new Date());

System.out.println(docA.getRevisions().getCount() == 0 ? "Documents are equal" : "Documents are not equal");
```

**Specify Advanced Comparison Options**

There are many different properties of the [CompareOptions](https://reference.aspose.com/words/java/com.aspose.words/compareoptions/) class which you can apply when you want to compare documents.

```java
Document docA = new Document("Document.docx");
Document docB = docA.deepClone();

CompareOptions options = new CompareOptions();
options.setIgnoreFormatting(true);
options.setIgnoreHeadersAndFooters(true);
options.setIgnoreCaseChanges(true);
options.setIgnoreTables(true);
options.setIgnoreFields(true);
options.setIgnoreComments(true);
options.setIgnoreTextboxes(true);
options.setIgnoreFootnotes(true);

docA.compare(docB, "user", new Date(), options);

System.out.println(docA.getRevisions().getCount() == 0 ? "Documents are equal" : "Documents are not equal");
```

---