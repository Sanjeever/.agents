# Aspose.Words for Java - Find and Replace

---

**URL:** https://docs.aspose.com/words/java/find-and-replace.md

**Contents:**
- Find and Replace Text Using Simple String Replacement
- Find and Replace Text Using Regular Expressions
- Find and Replace String Using Metacharacters
- Find and Replace String in Header/Footer of a Document
- Ignore Text During Find and Replace
- Customize Find and Replace Operation

---
title: "Find and Replace"
---

Aspose.Words allows you to find a specific string or regular expression pattern in your document and replace it with an alternative without installing and using additional applications such as Microsoft Word.

You can work with many options during the find and replace process using [FindReplaceOptions](https://reference.aspose.com/words/java/com.aspose.words/findreplaceoptions/) class.

You can use one of the [Replace](https://reference.aspose.com/words/java/com.aspose.words/range/?#replace-java.lang.String-java.lang.String) methods to find or replace in various scenarios.

**Find and Replace Text Using Simple String Replacement**

In this case, you can specify a string to be replaced, a string that will replace all its occurrences, whether the replacement is case-sensitive, and whether only stand-alone words will be affected.

The following code example shows how to find the string “_CustomerName_” and replace it with the string *“James Bond”*:

```java
Document doc = new Document();
DocumentBuilder builder = new DocumentBuilder(doc);

builder.writeln("Hello _CustomerName_,");

doc.getRange().replace("_CustomerName_", "James Bond", new FindReplaceOptions(FindReplaceDirection.FORWARD));

doc.save("FindAndReplace.ReplaceWithString.docx");
```

**Find and Replace Text Using Regular Expressions**

A regular expression (regex) is a pattern that describes a certain sequence of text. Suppose you want to replace all double occurrences of a word with a single word occurrence. Then you can apply the following regular expression to specify the double-word pattern: `([a-zA-Z]+) \1`.

The following code example shows how to replace strings that match a regular expression pattern with a specified replacement string:

```java
Document doc = new Document();
DocumentBuilder builder = new DocumentBuilder(doc);

builder.writeln("sad mad bad");

FindReplaceOptions options = new FindReplaceOptions();

doc.getRange().replace(Pattern.compile("[s|m]ad"), "bad", options);

doc.save("FindAndReplace.ReplaceWithRegex.docx");
```

**Find and Replace String Using Metacharacters**

You can use metacharacters in the search string or the replacement string if a particular text or phrase is composed of multiple paragraphs, sections, or pages. Some of the metacharacters include **&p** for a paragraph break, **&b** for a section break, **&m** for a page break, and **&l** for a line break.

Note that the metacharacter **&&** equals to **&**. For example, if you need to find text for **&p** that is not a paragraph break, then you can use **&&p**.

The following code example shows how to replace text with paragraph and page break:

```java
Document doc = new Document();
DocumentBuilder builder = new DocumentBuilder(doc);

builder.getFont().setName("Arial");
builder.writeln("First section");
builder.writeln("  1st paragraph");
builder.writeln("  2nd paragraph");
builder.writeln("{insert-section}");
builder.writeln("Second section");
builder.writeln("  1st paragraph");

FindReplaceOptions findReplaceOptions = new FindReplaceOptions();
findReplaceOptions.getApplyParagraphFormat().setAlignment(ParagraphAlignment.CENTER);

// Double each paragraph break after word "section", add kind of underline and make it centered.
int count = doc.getRange().replace("section&p", "section&p----------------------&p", findReplaceOptions);

// Insert section break instead of custom text tag.
count = doc.getRange().replace("{insert-section}", "&b", findReplaceOptions);

doc.save("FindAndReplace.ReplaceTextContainingMetaCharacters.docx");
```

**Find and Replace String in Header/Footer of a Document**

You can find and replace text in the header/footer section of a Word document using the [HeaderFooter](https://reference.aspose.com/words/java/com.aspose.words/headerfooter/) class:

```java
Document doc = new Document("Footer.docx");

HeaderFooterCollection headersFooters = doc.getFirstSection().getHeadersFooters();
HeaderFooter footer = headersFooters.getByHeaderFooterType(HeaderFooterType.FOOTER_PRIMARY);

FindReplaceOptions options = new FindReplaceOptions();
options.setMatchCase(false);
options.setFindWholeWordsOnly(false);

footer.getRange().replace("(C) 2006 Aspose Pty Ltd.", "Copyright (C) 2020 by Aspose Pty Ltd.", options);

doc.save("FindAndReplace.ReplaceTextInFooter.docx");
```

**Ignore Text During Find and Replace**

While applying the find and replace operation, you can ignore certain segments of the text. So, certain parts of the text can be excluded from the search, and the find and replace can be applied only to the remaining parts.

Aspose.Words provides many find and replace properties for ignoring text such as [IgnoreDeleted](https://reference.aspose.com/words/java/com.aspose.words/findreplaceoptions/#getIgnoreDeleted), [IgnoreFieldCodes](https://reference.aspose.com/words/java/com.aspose.words/findreplaceoptions/#getIgnoreFieldCodes), [IgnoreFields](https://reference.aspose.com/words/java/com.aspose.words/findreplaceoptions/#getIgnoreFields), [IgnoreFootnotes](https://reference.aspose.com/words/java/com.aspose.words/findreplaceoptions/#getIgnoreFootnotes), and [IgnoreInserted](https://reference.aspose.com/words/java/com.aspose.words/findreplaceoptions/#getIgnoreInserted).

The following code example shows how to ignore text inside delete revisions:

```java
Document doc = new Document();
DocumentBuilder builder = new DocumentBuilder(doc);

// Insert non-revised text.
builder.writeln("Deleted");
builder.write("Text");

// Remove first paragraph with tracking revisions.
doc.startTrackRevisions("author", new Date());
doc.getFirstSection().getBody().getFirstParagraph().remove();
doc.stopTrackRevisions();

FindReplaceOptions options = new FindReplaceOptions();
options.setIgnoreDeleted(true);

Pattern regex = Pattern.compile("e");
doc.getRange().replace(regex, "*", options);

System.out.println(doc.getText());

options.setIgnoreDeleted(false);
doc.getRange().replace(regex, "*", options);

System.out.println(doc.getText());
```

**Customize Find and Replace Operation**

Aspose.Words provides many different [properties](https://reference.aspose.com/words/java/com.aspose.words/findreplaceoptions/) to find and replace text such as applying specific format with [ApplyFont](https://reference.aspose.com/words/java/com.aspose.words/findreplaceoptions/#getApplyFont) and [ApplyParagraphFormats](https://reference.aspose.com/words/java/com.aspose.words/findreplaceoptions/#getApplyParagraphFormat) properties, using substitutions in replacement patterns with [UseSubstitutions](https://reference.aspose.com/words/java/com.aspose.words/findreplaceoptions/#getUseSubstitutions) property, and others.

The following code example shows how to highlight a specific word in your document:

```java
Document doc = new Document("Footer.docx");

FindReplaceOptions options = new FindReplaceOptions();
options.getApplyFont().setHighlightColor(Color.orange);

Pattern regex = Pattern.compile("(header|footer)");
doc.getRange().replace(regex, "", options);
```

Aspose.Words allows you to use the [IReplacingCallback](https://reference.aspose.com/words/java/com.aspose.words/ireplacingcallback/) interface to create and call a custom method during a replace operation.

If you need to replace a string with an HTML tag, apply the **IReplacingCallback** interface to customize the find and replace operation so the match starts at the beginning of a run with the match node of your document. Let us provide several examples of using **IReplacingCallback**.

The following code example shows how to replace text specified with HTML:

```java
public void replaceWithHtml() throws Exception {
    Document doc = new Document();
    DocumentBuilder builder = new DocumentBuilder(doc);

    builder.writeln("Hello <CustomerName>,");

    FindReplaceOptions options = new FindReplaceOptions();
    options.setReplacingCallback(new ReplaceWithHtmlEvaluator(options));

    doc.getRange().replace(Pattern.compile(" <CustomerName>,"), "", options);

    doc.save("FindAndReplace.ReplaceWithHtml.docx");
}

private static class ReplaceWithHtmlEvaluator implements IReplacingCallback {
    ReplaceWithHtmlEvaluator(FindReplaceOptions options) {
        mOptions = options;
    }

    /// <summary>
    /// NOTE: This is a simplistic method that will only work well when the match
    /// starts at the beginning of a run.
    /// </summary>
    public int replacing(ReplacingArgs args) throws Exception {
        DocumentBuilder builder = new DocumentBuilder((Document) args.getMatchNode().getDocument());
        builder.moveTo(args.getMatchNode());

        // Replace '<CustomerName>' text with a red bold name.
        builder.insertHtml("<b><font color='red'>James Bond, </font></b>");
        args.setReplacement("");

        return ReplaceAction.REPLACE;
    }

    private FindReplaceOptions mOptions;
}
```

The following code example shows how to prepend a line number to each line:

```java
public void lineCounter() throws Exception {
    Document doc = new Document();
    DocumentBuilder builder = new DocumentBuilder(doc);

    builder.writeln("This is first line");
    builder.writeln("Second line");
    builder.writeln("And last line");

    // Prepend each line with line number.
    FindReplaceOptions options = new FindReplaceOptions();
    options.setReplacingCallback(new LineCounterCallback());

    Pattern regex = Pattern.compile("[^&p]*&p");
    doc.getRange().replace(regex, "", options);

    doc.save("FindAndReplace.LineCounter.docx");
}

static class LineCounterCallback implements IReplacingCallback {
    public int replacing(ReplacingArgs args) {
        String value = args.getMatch().group(0);
        System.out.println(value);

        args.setReplacement(MessageFormat.format("{0} {1}", mCounter++, value));
        return ReplaceAction.REPLACE;
    }

    private int mCounter = 1;
}
```

---