# Aspose.Words for Java - Use AI-powered Features

---

**URL:** https://docs.aspose.com/words/java/ai-powered-features.md

**Contents:**
- Configure AI models
- Translate Document Using AI
- Summarize Document Using AI
- Check Grammar Using AI

---
title: "Use AI-powered Features"
---

Aspose.Words for Java AI enables operations such as text summarization, grammar checking, and document translation using cutting edge models from Anthropic, Google, and OpenAI.

**Configure AI models**

The Aspose.Words for Java can be connected to Anthropic, Google, and OpenAI models. To do this, just use the appropriate AiModel class and specify the [AiModelType](https://reference.aspose.com/words/java/com.aspose.words/aimodeltype/).

```java
public static AiModel create(int modelType)
```

In addition to the standard models, you can use an LLM deployed on your server. Just select the appropriate AiModel class and specify the URL, name, and ApiKey.

```java
public void selfHostedModel() throws Exception
{
    Document doc = new Document("Big document.docx");

    String apiKey = System.getenv("API_KEY");
    // Use OpenAI generative language models.
    AiModel model = new CustomAiModel().withApiKey(apiKey);

    Document translatedDoc = model.translate(doc, Language.RUSSIAN);
    translatedDoc.save("AI.SelfHostedModel.docx");
    }

// Custom self-hosted AI model.
static class CustomAiModel extends OpenAiModel
{
    protected String getUrl()
    {
        return "https://localhost/";
    }

    protected String getName()
    {
        return "my-model-24b";
    }
}
```

**Translate Document Using AI**

Translating documents supports multiple languages and formats, making it easy to work with a wide range of content.

To translate a document, use one of the [Translate](https://reference.aspose.com/words/java/com.aspose.words/aimodel/#translate-com.aspose.words.Document-int) methods and one of the supported languages listed in the [Language](https://reference.aspose.com/words/java/com.aspose.words/language/) enumeration:

```java
Document doc = new Document("Document.docx");

String apiKey = System.getenv("API_KEY");
 IAiModelText model = (IAiModelText)AiModel.create(AiModelType.GEMINI_15_FLASH).withApiKey(apiKey);

Document translatedDoc = model.translate(doc, Language.ARABIC);
translatedDoc.save("AI.AiTranslate.docx");
```

**Summarize Document Using AI**

To summarize a document, use one of the [Summarize](https://reference.aspose.com/words/java/com.aspose.words/aimodel/#summarize-com.aspose.words.Document-com.aspose.words.SummarizeOptions) methods

You can also optionally specify [SummarizeOptions](https://reference.aspose.com/words/java/com.aspose.words/summarizeoptions/). Use the following values from the [SummaryLength](https://reference.aspose.com/words/java/com.aspose.words/summarylength/) enumeration to set the summary length:

* `VeryShort` - generates 1-2 sentences
* `Short` - generates 3-4 sentences
* `Medium` - generates 5-6 sentences
* `Long` - generates 7-10 sentences
* `VeryLong` - generates 11-20 sentences

Pass one or more documents to the Summarizer to produce a concise summary that accurately represents the content:

```java
Document firstDoc = new Document("Big document.docx");
Document secondDoc = new Document("Document.docx");

String apiKey = System.getenv("API_KEY");
// Use OpenAI or Google generative language models.
IAiModelText model = (IAiModelText)AiModel.create(AiModelType.GPT_4_O_MINI).withApiKey(apiKey);

Document oneDocumentSummary = model.summarize(firstDoc, new SummarizeOptions(); { oneDocumentSummary.setSummaryLength(SummaryLength.SHORT); });
oneDocumentSummary.save("AI.AiSummarize.One.docx");

Document multiDocumentSummary = model.summarize(new Document[] { firstDoc, secondDoc }, new SummarizeOptions(); { multiDocumentSummary.setSummaryLength(SummaryLength.LONG); });
multiDocumentSummary.save("AI.AiSummarize.Multi.docx");
```

**Check Grammar Using AI**

Use one of the [CheckGrammar](https://reference.aspose.com/words/java/com.aspose.words/aimodel/#checkGrammar-com.aspose.words.Document-com.aspose.words.CheckGrammarOptions) methods to correct spelling, fix grammatical errors and typos, improve stylistic quality, and optionally track changes using revisions for later review:

```java
Document doc = new Document("Big document.docx");

String apiKey = System.getenv("API_KEY");
// Use OpenAI generative language models.
IAiModelText model = (OpenAiModel)AiModel.create(AiModelType.GPT_4_O_MINI).withApiKey(apiKey);

CheckGrammarOptions grammarOptions = new CheckGrammarOptions();
grammarOptions.setImproveStylistics(true);

Document proofedDoc = model.checkGrammar(doc, grammarOptions);
proofedDoc.save("AI.AiGrammar.docx");
```

---