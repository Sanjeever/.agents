# Aspose.Words for Java - Sign with Digital Signature

---

**URL:** https://docs.aspose.com/words/java/working-with-digital-signatures.md

**Contents:**
- Supported Formats
- Create a Digital Signature
- Sign a Document
- Add a Signature Line
- Sign a Generated PDF Document
- Remove a Digital Signature

---
title: "Sign with Digital Signature"
---

A digital signature is a technological implementation of electronic signatures to sign documents and authenticate the signer to guarantee that a document has not been modified since it was signed. Use the [DigitalSignatureUtil](https://reference.aspose.com/words/java/com.aspose.words/digitalsignatureutil/) class to work with digital signatures.

**Supported Formats**

Aspose.Words allows you to work with digital signatures on DOC, OOXML, and ODT documents and to sign the generated document in PDF or XPS format.

**Create a Digital Signature**

Aspose.Words allows you to create X.509 certificate, a digital certificate that uses the internationally accepted X.509 PKI standard to verify that a public key belongs to the signer included inside the certificate. To do this, use the [Create](https://reference.aspose.com/words/java/com.aspose.words/certificateholder/#create-byte---java.lang.String) method within the [CertificateHolder](https://reference.aspose.com/words/java/com.aspose.words/certificateholder/) class.

**Sign a Document**

Aspose.Words allows you to sign a DOC, DOCX, XPS, or ODT document digitally using the [Sign](https://reference.aspose.com/words/java/com.aspose.words/digitalsignatureutil/#sign-java.io.InputStream-java.io.OutputStream-com.aspose.words.CertificateHolder) method and [SignOptions](https://reference.aspose.com/words/java/com.aspose.words/signoptions/) properties.

```java
CertificateHolder certHolder = CertificateHolder.create("morzal.pfx", "aw");
        
DigitalSignatureUtil.sign("Digitally signed.docx", "Document.Signed.docx",
    certHolder);
```

**Add a Signature Line**

Aspose.Words allows you to insert a signature line using the [DocumentBuilder.InsertSignatureLine](https://reference.aspose.com/words/java/com.aspose.words/documentbuilder/#insertSignatureLine-com.aspose.words.SignatureLineOptions) method.  You can also set the parameters for this representation using the [SignatureLineOptions](https://reference.aspose.com/words/java/com.aspose.words/signaturelineoptions/) class.

```java
Document doc = new Document();
DocumentBuilder builder = new DocumentBuilder(doc);

SignatureLineOptions signatureLineOptions = new SignatureLineOptions();
{
    signatureLineOptions.setSigner("yourname");
    signatureLineOptions.setSignerTitle("Worker");
    signatureLineOptions.setEmail("yourname@aspose.com");
    signatureLineOptions.setShowDate(true);
    signatureLineOptions.setDefaultInstructions(false);
    signatureLineOptions.setInstructions("Please sign here.");
    signatureLineOptions.setAllowComments(true);
}

SignatureLine signatureLine = builder.insertSignatureLine(signatureLineOptions).getSignatureLine();
signatureLine.setProviderId(UUID.fromString("CF5A7BB4-8F3C-4756-9DF6-BEF7F13259A2"));
        
doc.save("SignDocuments.SignatureLineProviderId.docx");

SignOptions signOptions = new SignOptions();
{
    signOptions.setSignatureLineId(signatureLine.getId());
    signOptions.setProviderId(signatureLine.getProviderId());
    signOptions.setComments("Document was signed by Aspose");
    signOptions.setSignTime(new Date());
}

CertificateHolder certHolder = CertificateHolder.create("morzal.pfx", "aw");

DigitalSignatureUtil.sign("SignDocuments.SignatureLineProviderId.docx", 
    "SignDocuments.CreateNewSignatureLineAndSetProviderId.docx", certHolder, signOptions);
```

**Sign a Generated PDF Document**

Aspose.Words allows you to sign and get all details of a PDF document using the [PdfDigitalSignatureDetails](https://reference.aspose.com/words/java/com.aspose.words/pdfdigitalsignaturedetails/) properties.

```java
Document doc = new Document();
DocumentBuilder builder = new DocumentBuilder(doc);

builder.writeln("Test Signed PDF.");

PdfSaveOptions saveOptions = new PdfSaveOptions();
saveOptions.setDigitalSignatureDetails(new PdfDigitalSignatureDetails(
        CertificateHolder.create("morzal.pfx", "aw"), "reason", "location",
        new Date()));

doc.save("WorkingWithPdfSaveOptions.DigitallySignedPdfUsingCertificateHolder.pdf", saveOptions);
```

**Remove Digital Signatures**

Aspose.Words allows you to remove all digital signatures from a signed document using the [RemoveAllSignatures](https://reference.aspose.com/words/java/com.aspose.words/digitalsignatureutil/#removeAllSignatures-java.io.InputStream-java.io.OutputStream) method.

```java
DigitalSignatureUtil.removeAllSignatures("Digitally signed.docx",
        "DigitalSignatureUtil.LoadAndRemove.FromString.docx");

// 2 - Determine the locations of both the signed document and the unsigned copy by file streams:
try (FileInputStream streamIn = new FileInputStream("Digitally signed.docx"))
{
    try (FileOutputStream streamOut = new FileOutputStream("DigitalSignatureUtil.LoadAndRemove.FromStream.docx"))
    {
        DigitalSignatureUtil.removeAllSignatures(streamIn, streamOut);
    }
}

// Verify that both our output documents have no digital signatures.
Assert.assertEquals(IterableUtils.size(DigitalSignatureUtil.loadSignatures("DigitalSignatureUtil.LoadAndRemove.FromString.docx")), 0);
Assert.assertEquals(IterableUtils.size(DigitalSignatureUtil.loadSignatures("DigitalSignatureUtil.LoadAndRemove.FromStream.docx")), 0);
```

You can not remove only one digital signature within your document.

---