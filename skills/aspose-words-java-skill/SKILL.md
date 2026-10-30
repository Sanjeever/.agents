---
name: aspose.words
description: "Develop applications with Aspose.Words for Java APIs to create, edit, convert, and render Word documents. Automate document comparison, reporting with LINQ Reporting Engine, mail merge, text search and replacement, document merging and splitting, digital signatures, watermark management, and AI-powered tasks such as translation, summarization, and grammar checking. Use this documentation when building, integrating, or troubleshooting document processing solutions."
---

# Aspose.Words for Java Skill

This skill provides AI assistants with structured guidance for working with the Aspose.Words for Java API, including documentation, code examples, and implementation best practices.

Load this skill whenever a request involves Aspose.Words APIs, document processing operations, or Word document automation.

## When to Use This Skill

Use this skill when you want to:

- automate document creation, editing, and conversion
- convert documents between supported file formats
- compare document content and identify differences
- generate documents using LINQ Reporting Engine or Mail Merge
- combine multiple documents or split a document into separate files
- add or remove watermarks and apply digital signatures
- leverage AI features such as translation, summarization, or grammar checking
- find information about supported file formats and API capabilities
- integrate Aspose.Words for Java into your applications
- troubleshoot API usage or document processing behavior

## Example User Requests

- Convert DOCX to PDF with Aspose.Words for Java
- Compare two Word documents
- Generate invoices using the LINQ Reporting Engine
- Perform a Mail Merge with data from a JSON or database source
- Replace text or insert content into a Word document
- Merge multiple DOCX files into a single document
- Split a document into separate files by section or page
- Add a watermark to a document
- Apply a digital signature to a document
- Translate a document using the Aspose.Words AI
- Summarize a document using the Aspose.Words AI
- Check grammar in a document using the Aspose.Words AI
- Which document formats can Aspose.Words for Java read, write, and convert?

## Installation

Install Aspose.Words for Java via Maven Repository:

1. Specify Aspose Maven Repository configuration/location in your Maven pom.xml:

```java
<repositories>
	<repository>
		<id>AsposeJavaAPI</id>
		<name>Aspose Java API</name>
		<url>https://releases.aspose.com/java/repo/</url>
	</repository>
</repositories>
```

2. Define the Aspose.Words for Java API dependency in your pom.xml:

```java
<dependencies>
	<dependency>
		<groupId>com.aspose</groupId>
		<artifactId>aspose-words</artifactId>
		<version>22.11</version>
		<classifier>jdk17</classifier>
	</dependency>
	<dependency>
		<groupId>com.aspose</groupId>
		<artifactId>aspose-words</artifactId>
		<version>22.11</version>
		<classifier>javadoc</classifier>
	</dependency>
</dependencies>
```

## Licensing

You can apply a license using a file or stream object.

How to apply a license from a file:

```java
License license = new License();

try
{
    license.setLicense("Aspose.Words.lic");
    
    System.out.println("License set successfully.");
}
catch (Exception e)
{
    System.out.println("\nThere was an error setting the license: " + e.getMessage());
}
```

How to apply a license from a stream:

```java
License license = new License();

try
{
    license.setLicense(new FileInputStream(new File("Aspose.Words.lic")));
    
    System.out.println("License set successfully.");
}
catch (Exception e)
{
    System.out.println("\nThere was an error setting the license: " + e.getMessage());
}
```

## Quick Reference

### Create or Load a Document

Create a document from scratch:

```java
Document doc = new Document();

DocumentBuilder builder = new DocumentBuilder(doc);
builder.writeln("Hello World!");

doc.save("AddContentUsingDocumentBuilder.CreateNewDocument.docx");
```

Load a document:

```java
Document doc = new Document("Document.docx");
```

### Save a Document

Load DOC and save it to DOCX to a file:

```java
Document doc = new Document("Document.doc");

doc.save("BaseConversions.DocToDocx.docx");
```

### Document Conversion

Convert documents between supported formats:

```java
Document doc = new Document("Document.docx");
doc.save("BaseConversions.DocxToPdf.pdf");
```
### Compare Documents

Check if two documents are equal or not:

```java
Document docA = new Document("Document.docx");
Document docB = docA.deepClone();

// DocA now contains changes as revisions.
docA.compare(docB, "user", new Date());

System.out.println(docA.getRevisions().getCount() == 0 ? "Documents are equal" : "Documents are not equal");
```

### LINQ Reporting

To build a report from the template, you can use the following source code:

```java
String dataDir = Utils.getDataDir(HelloWorld.class);

String fileName = "HelloWorld.doc";
// Load the template document.
Document doc = new Document(dataDir + fileName);

// Create an instance of sender class to set it's properties.
Sender sender = new Sender();
sender.setName("LINQ Reporting Engine");
sender.setMessage("Hello World");

// Create a Reporting Engine.
ReportingEngine engine = new ReportingEngine();

// Execute the build report.
engine.buildReport(doc, sender, "sender");

dataDir = dataDir + Utils.GetOutputFilePath(fileName);

// Save the finished document to disk.
doc.save(dataDir);
```

### Mail Merge

Execute a simple Mail Merge operation:

```java
Document doc = new Document();
DocumentBuilder builder = new DocumentBuilder(doc);

// Create Merge Fields.
builder.insertField(" MERGEFIELD CustomerName ");
builder.insertParagraph();
builder.insertField(" MERGEFIELD Item ");
builder.insertParagraph();
builder.insertField(" MERGEFIELD Quantity ");

// Fill the fields in the document with user data.
doc.getMailMerge().execute(new String[]{"CustomerName", "Item", "Quantity"},
        new Object[]{"John Doe", "Hawaiian", "2"});

doc.save("BaseOperations.SimpleMailMerge.docx");
```

### Search and Replace

Find the string “CustomerName” and replace it with the string “James Bond”:

```java
Document doc = new Document();
DocumentBuilder builder = new DocumentBuilder(doc);

builder.writeln("Hello _CustomerName_,");

doc.getRange().replace("_CustomerName_", "James Bond", new FindReplaceOptions(FindReplaceDirection.FORWARD));

doc.save("FindAndReplace.ReplaceWithString.docx");
```

### Insert and Append Documents

Merge documents, for example, append a document to the end of another document:

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

### Split Large Documents

Split a document into smaller parts by heading:

```java
Document doc = new Document("Rendering.docx");

HtmlSaveOptions options = new HtmlSaveOptions();
// Split a document into smaller parts, in this instance split by heading.
options.setDocumentSplitCriteria(DocumentSplitCriteria.HEADING_PARAGRAPH);

doc.save("SplitDocument.ByHeadings.epub", options);
```

### Digital Signature

Sign documents using a certificate holder and sign options:

```java
CertificateHolder certHolder = CertificateHolder.create("morzal.pfx", "aw");
        
DigitalSignatureUtil.sign("Digitally signed.docx", "Document.Signed.docx", certHolder);
```

### Add Watermark

Add a text watermark:

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

Add an image watermark:

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

### AI-powered Features

Use AI-powered capabilities (if enabled) to translate or summarize documents, or to check grammar:

```java
public static AiModel create(int modelType)
```

### Document Protection

Encrypt a document with a password:

```java
Document doc = new Document();
DocumentBuilder builder = new DocumentBuilder(doc);

builder.write("Hello world!");

DocSaveOptions saveOptions = new DocSaveOptions();
saveOptions.setPassword("password");

doc.save("WorkingWithDocSaveOptions.EncryptDocumentWithPassword.docx", saveOptions);
```

### Print a Document

Print a document via the Print preview and Settings dialogs:

```java
Document doc = new Document(dataDir + "TestFile.doc");

PrinterJob pj = PrinterJob.getPrinterJob();

// Initialize the Print Dialog with the number of pages in the document.
PrintRequestAttributeSet attributes = new HashPrintRequestAttributeSet();
attributes.add(new PageRanges(1, doc.getPageCount()));

if (!pj.printDialog(attributes)) {
	return;
}

AsposeWordsPrintDocument awPrintDoc = new AsposeWordsPrintDocument(doc);
// Pass our document as pageable to the printer job.
pj.setPageable(awPrintDoc);

PrintPreviewDialog previewDlg = new PrintPreviewDialog(awPrintDoc);
// Pass the desired page range attributes to the print preview dialog.
previewDlg.setPrinterAttributes(attributes);

// Proceed with printing if the user accepts the print preview.
if (previewDlg.display())
	pj.print(attributes);
```

## Reference Files

This skill includes comprehensive documentation in `references/`:

- **supported-formats.md** – supported document formats
- **licensing.md** – license types and license applying options
- **getting-started.md** – guides and tutorials for beginners
- **configuring.md** - configuring fonts, hyphenation, text shaping
- **create-or-load.md** – creating or loading a document
- **saving.md** – saving a document
- **conversion.md** – converting a document from one format to another
- **comparison.md** – comparison documents to detected changes: insertions, deletions, and modifications
- **reporting.md** – LINQ reporting engine
- **mail-merge.md** – creating personalized documents through a combination of a document template with data
- **replacement.md** – search and replace functionality
- **merge.md** – merging multiple documents into one
- **splitting.md** – splitting a large document by various split criteria
- **signature.md** – digital signing of documents
- **watermark.md** – adding or removing watermarks in documents
- **ai.md** – summarization, grammar checking, and document translation using AI models
- **protection.md** – document protection
- **printing.md** – document printing

Open the relevant reference file when detailed documentation is needed.

## Resources

- Official Aspose.Words for Java documentation: https://docs.aspose.com/words/java/
- API Reference: https://reference.aspose.com/words/java

Official Aspose.Words for Java documentation pages are AI-friendly and support the .md format. For example, the page `https://docs.aspose.com/words/java/convert-a-document-to-pdf/` has a version `https://docs.aspose.com/words/java/convert-a-document-to-pdf.md`, the page `https://docs.aspose.com/words/java/split-a-document/` has a version `https://docs.aspose.com/words/java/split-a-document.md`, and so on.

## Notes for AI Agents

- Prefer examples that use the official Aspose.Words for Java APIs.
- Provide short and clear code snippets when possible.
- Use the references directory to retrieve detailed documentation.
- If a feature is not covered in this file, check the corresponding reference file.