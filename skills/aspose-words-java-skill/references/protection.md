
# Aspose.Words for Java - Document Protection

---

**URL:** https://docs.aspose.com/words/java/protect-or-encrypt-a-document.md

**Contents:**
- Document Protection Options
- Make a Document Read-Only
- Encrypt a Document
- Restrict Document Editing

---
title: "Protect or Encrypt a Document"
---

**Document Protection Options**

Aspose.Words currently provides the document features listed in the table below. You can find the details on each of the features in the corresponding child article in the current section.

| Aspose.Words protection feature | Corresponding child article    | Corresponding MS Word feature                                |
| ------------------------------- | ------------------------------ | ------------------------------------------------------------ |
| Read-Only                       | “Make a Document Read-Only”    | Always Open Read-Only (File → Info → Protect Document)<br />Alternative feature: "Password to modify" (Save As → Tools → General Options → Password) |
| Encrypt a Document              | “Encrypt a Document”           | Encrypt with Password (File → Info → Protect Document)<br />Alternative feature: "Password to open" (Save As → Tools → General Options → Password) |
| Restrict Editing                | “Restrict Document Editing”    | Restrict Editing (File – Info – Protect Document)<br />Alternative feature: "Restrict Editing" (Review → Protect → Restrict Editing) |
| Digital Signatures              | “Work with Digital Signatures” | Add a Digital Signature (File → Info → Protect Document)     |

**Make a Document Read-Only**

Aspose.Words allows you to make a document read-only to restrict editing by using the [ReadOnlyRecommended](https://reference.aspose.com/words/java/com.aspose.words/writeprotection/?_gl=1*1v42osf*_ga*MTMyMTk1NTQ3Mi4xNjM5Mzk2MTk1*_ga_W0DG8XJWKL*czE3ODgxNTAyMTQkbzIwODkkZzEkdDE3ODgxNTIzNzMkajYwJGwwJGgxNDA5MjcyNDk5*_gcl_au*MTM3OTg4MTQyNi4xNzg3NTQ3MDc3#getReadOnlyRecommended) property and the [SetPassword](https://reference.aspose.com/words/java/com.aspose.words/writeprotection/?_gl=1*vsrhyv*_ga*MTMyMTk1NTQ3Mi4xNjM5Mzk2MTk1*_ga_W0DG8XJWKL*czE3ODgxNTAyMTQkbzIwODkkZzEkdDE3ODgxNTIzNzMkajYwJGwwJGgxNDA5MjcyNDk5*_gcl_au*MTM3OTg4MTQyNi4xNzg3NTQ3MDc3#setPassword-java.lang.String) method.

```java
Document doc = new Document();
DocumentBuilder builder = new DocumentBuilder(doc);

builder.write("Open document as read-only");

// Enter a password that's up to 15 characters long.
doc.getWriteProtection().setPassword("MyPassword");

// Make the document as read-only.
doc.getWriteProtection().setReadOnlyRecommended(true);

// Apply write protection as read-only.
doc.protect(ProtectionType.READ_ONLY);
doc.save("DocumentProtection.ReadOnlyProtection.docx");
```

**Encrypt a Document**

To encrypt a document, use the **Password** property to provide a password that functions as an encryption key. This will modify the content of your document and make it unreadable. The encrypted document will require to have this password entered before it can be opened.

You can find the appropriate **Password** property for the required format. For example, the [Password](https://reference.aspose.com/words/java/com.aspose.words/docsaveoptions/#getPassword) property in the [DocSaveOptions](https://reference.aspose.com/words/java/com.aspose.words/docsaveoptions/) class for DOC, or the [Password](https://reference.aspose.com/words/java/com.aspose.words/ooxmlsaveoptions/#getPassword) property in the [OoxmlSaveOptions](https://reference.aspose.com/words/java/com.aspose.words/ooxmlsaveoptions/) class for DOCX, DOCM, DOTX, DOTM, and FlatOpc.

```java
Document doc = new Document();
DocumentBuilder builder = new DocumentBuilder(doc);

builder.write("Hello world!");

DocSaveOptions saveOptions = new DocSaveOptions();
saveOptions.setPassword("password");

doc.save("WorkingWithDocSaveOptions.EncryptDocumentWithPassword.docx", saveOptions);
```

**Restrict Document Editing**

Aspose.Words allows you to control the way you restrict the content using the [ProtectionType](https://reference.aspose.com/words/java/com.aspose.words/protectiontype/) enumeration parameter. This will enable you to select an exact type of protection such as the following:

* AllowOnlyComments
* AllowOnlyFormFields
* AllowOnlyRevisions
* ReadOnly
* NoProtection

Aspose.Words allows you to protect your documents from changes using the [Protect](https://reference.aspose.com/words/java/com.aspose.words/document/#protect-int) method. This method is not a security feature and does not encrypt a document.

```java
Document doc = new Document();

// Apply document protection.
doc.protect(ProtectionType.NO_PROTECTION, "password");

doc.save("DocumentProtection.PasswordProtection.docx");
```

The following code example shows how to remove protection from your document:

```java
Document doc = new Document();
DocumentBuilder builder = new DocumentBuilder(doc);
builder.writeln("Text added to a document.");

// A document protection only works when document protection is turned and only editing in form fields is allowed.
doc.protect(ProtectionType.ALLOW_ONLY_FORM_FIELDS, "password");

// Save the protected document.
doc.save("DocumentProtection.AllowOnlyFormFieldsProtect.docx");
```

---