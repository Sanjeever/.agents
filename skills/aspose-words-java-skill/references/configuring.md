# Aspose.Words for Java - Configuring

---

**URL:** https://docs.aspose.com/words/java/working-with-hyphenation.md

**Contents:**
- Hyphenation Dictionaries
- Loading Hyphenation Dictionaries

---
title: "Hyphenation"
---

Hyphenation is a way to arrange text in a document more compactly. However, it is important to remember that hyphenation rules may vary depending on the language.

To ensure correct word hyphenation, language-specific hyphenation dictionaries are used.

**Hyphenation Dictionaries**

Different languages use different norms and rules for word hyphenation.The optimal solution for correct hyphenation is to use special dictionaries. Aspose.Words uses OpenOffice dictionaries.

For spell checking, OpenOffice uses the [Hunspell library](https://hunspell.github.io/). Hunspell uses the [Hyphen](https://github.com/hunspell/hyphen) for hyphenation.

Hyphenation dictionaries can be taken from the [LibreOffice dictionaries GitHub](https://github.com/LibreOffice/dictionaries). For example, [en-US hyphenation dictionary](https://github.com/LibreOffice/dictionaries/blob/master/en/hyph_en_US.dic).

As Microsoft Word uses dictionaries other than OpenOffice dictionaries to perform hyphenation, sometimes we have to advise customers to add the necessary patterns to their dictionaries in order to fix the hyphenation of particular words.

**Loading Hyphenation Dictionaries**

To use the hyphenation feature, first register a hyphenation dictionary.

Load hyphenation dictionaries for the specified languages from a file:

```java
Document doc = new Document("German text.docx");

Hyphenation.registerDictionary("en-US", "hyph_en_US.dic");
Hyphenation.registerDictionary("de-CH", "hyph_de_CH.dic");

doc.save("WorkingWithHyphenation.HyphenateWords.pdf");
```

Load hyphenation dictionaries for the specified language from a stream:

```java
Document doc = new Document("German text.docx");

FileInputStream stream = new FileInputStream("hyph_de_CH.dic");
Hyphenation.registerDictionary("de-CH", stream);

doc.save("WorkingWithHyphenation.LoadHyphenationDictionary.pdf");
```

Register only required hyphenation dictionaries “by request”. To achieve that, implement the [IHyphenationCallback](https://reference.aspose.com/words/java/com.aspose.words/ihyphenationcallback/) interface and use the static callback [Callback](https://reference.aspose.com/words/java/com.aspose.words/hyphenation/#getCallback):

```java
public void hyphenationCallback() throws Exception {
    try {
        // Register hyphenation callback.
        Hyphenation.setCallback(new CustomHyphenationCallback());

        Document document = new Document("German text.docx");
        document.save("WorkingWithHyphenation.HyphenationCallback.pdf");
    } catch (Exception e) {
        if (e.getMessage().startsWith("Missing hyphenation dictionary")) {
            System.out.println(e.getMessage());
        }

    } finally {
        Hyphenation.setCallback(null);
    }
}

public static class CustomHyphenationCallback implements IHyphenationCallback {
    public void requestDictionary(String language) throws Exception {
        String dictionaryFolder = getMyDir();
        String dictionaryFullFileName;
        switch (language) {
            case "en-US":
                dictionaryFullFileName = Paths.get(dictionaryFolder, "hyph_en_US.dic").toString();
                break;
            case "de-CH":
                dictionaryFullFileName = Paths.get(dictionaryFolder, "hyph_de_CH.dic").toString();
                break;
            default:
                throw new Exception(MessageFormat.format("Missing hyphenation dictionary for {0}.", language));
        }
        // Register dictionary for requested language.
        Hyphenation.registerDictionary(language, dictionaryFullFileName);
    }
}
```

---

**URL:** https://docs.aspose.com/words/java/working-with-asian-typography.md

**Contents:**
- Automatically Adjust Space between Asian and Latin Text or Numbers
- Set Line Break Options

---
title: "Asian Typography"
---

Asian Typography is a set of options for text paragraphs in documents written in Asian languages.

**Automatically Adjust Space between Asian and Latin Text or Numbers**

If you are designing a template with both East Asian and Latin text and  want to enhance the appearance of your form template by controlling the spaces between both types of text, you can configure your form template to automatically adjust the spaces between these two types of text. Use [AddSpaceBetweenFarEastAndAlpha](https://reference.aspose.com/words/java/com.aspose.words/paragraphformat/#getAddSpaceBetweenFarEastAndAlpha) and [AddSpaceBetweenFarEastAndDigit](https://reference.aspose.com/words/java/com.aspose.words/paragraphformat/#getAddSpaceBetweenFarEastAndDigit) properties of the [ParagraphFormat](https://reference.aspose.com/words/java/com.aspose.words/paragraphformat/) class:

```java
Document doc = new Document();
DocumentBuilder builder = new DocumentBuilder(doc);

ParagraphFormat paragraphFormat = builder.getParagraphFormat();
paragraphFormat.setAddSpaceBetweenFarEastAndAlpha(true);
paragraphFormat.setAddSpaceBetweenFarEastAndDigit(true);

builder.writeln("Automatically adjust space between Asian and Latin text");
builder.writeln("Automatically adjust space between Asian text and numbers");

doc.save("DocumentFormatting.SpaceBetweenAsianAndLatinText.docx");
```

**Set Line Break Options**

The Asian Typography tab of the paragraph properties dialog box in Microsoft Word has line break group. The options of this group can be set using the [FarEastLineBreakControl](https://reference.aspose.com/words/java/com.aspose.words/paragraphformat/#getFarEastLineBreakControl), [WordWrap](https://reference.aspose.com/words/java/com.aspose.words/paragraphformat/#getWordWrap), [HangingPunctuation](https://reference.aspose.com/words/java/com.aspose.words/paragraphformat/#getHangingPunctuation):

```java
Document doc = new Document("Asian typography.docx");

ParagraphFormat format = doc.getFirstSection().getBody().getParagraphs().get(0).getParagraphFormat();
format.setFarEastLineBreakControl(false);
format.setWordWrap(true);
format.setHangingPunctuation(false);

doc.save("DocumentFormatting.AsianTypographyLineBreakGroup.docx");
```

---