# Aspose.Words for Java - Licensing and Subscription

---

**URL:** https://docs.aspose.com/words/java/licensing.md

**Contents:**
- Free Trial
- Temporary License
- Protecting Your Purchased License
- Load a License from a File
- Load a License from a Stream Object
- Include the License File as an Embedded Resource
- Apply Metered License
- Using Multiple Aspose Products

---
title: "Licensing and Subscription"
---

**Free Trial**

The evaluation version is the same as the purchased one – the [Trial version](https://releases.aspose.com/words/java/) simply becomes licensed when you add a few lines of code to apply the license.

The Trial version of Aspose.Words without the specified license provides full product functionality, but inserts an evaluative watermark at the top of the document upon loading and saving and limits the maximum document size to a few hundred paragraphs.

**Temporary License**

If you wish to test Aspose.Words without the limitations of the Trial version, you can also request a 30-day Temporary License. For more details, see the [Get a Temporary License](https://purchase.aspose.com/temporary-license/) page.

**Purchased License**

After purchase, you need to apply the license file or include the license file as an embedded resource. This section describes options of how this can be done, and also comments on some common questions.

You need to set the license:

* only once per application domain
* before using any other Aspose.Words classes

You can find pricing information on the [Pricing Information](https://purchase.aspose.com/pricing/words/family/) page.

**Protecting Your Purchased License**

After purchasing a license, you need to carefully read the information on page [Protecting Your Purchased License](https://purchase.aspose.com/orders/protecting-your-license-file) to protect your license file. Please note that this page is available for viewing only if you have a paid license.

**Load a License from a File**

Using the [SetLicense](https://reference.aspose.com/words/java/com.aspose.words/license/#setLicense-java.lang.String) method, you can try to find the license file in the embedded resources or assembly folders for further use.

The following code example shows how to initialize a license from a folder:

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

**Load a License from a Stream Object**

The following code example shows how to initialize a license from a stream using another [SetLicense](https://reference.aspose.com/words/java/com.aspose.words/license/#setLicense-java.io.InputStream) method:

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

**Include the License File as an Embedded Resource**

A neat way to package a license with your application and make sure it will not be lost is to include it as an embedded resource into one of the assemblies that call Aspose.Words. To include a file as an embedded resource, follow these steps:

1. In Visual Studio, include the .lic file into the project using the “**File | Add Existing Item…**” menu.
2. Select the file in the Solution Explorer and set “**Build Action to Embedded Resource**” in the Properties window.
3. In your code, invoke **SetLicense** that passes only the short name of the resource file.

**Apply Metered License**

After completing all the necessary steps to obtain this type of license, you will receive the keys, not the license file. This metered key can be applied using the [Metered](https://reference.aspose.com/words/java/com.aspose.words/metered/) class specially introduced for this purpose.

Do not call the **SetMeteredKey** method frequently so that this licensing method properly accumulates consumption and reports it to us. Just instantiate the Aspose.Words library, call **SetMeteredKey** once, then leave the library instantiated and reuse it.

```java
try
{
    Metered metered = new Metered();
    metered.setMeteredKey("*****", "*****");

    Document doc = new Document(getMyDir() + "Document.docx");

    System.out.println(doc.getPageCount());
}
catch (Exception e)
{
    System.out.println("\nThere was an error setting the license: " + e.getMessage());
}
```

Normally it is enough to apply the metered license once on application start. However, if the metered licensing mechanism fails to communicate with the Aspose servers for 24 hours, Aspose.Words will exit licensed mode and switch to evaluation mode. To avoid such case, you can use the [IsMeteredLicensed](https://reference.aspose.com/words/java/com.aspose.words/metered/#isMeteredLicensed) method to check the license status and reapply the metered license if necessary.

**Using Multiple Aspose Products**

If you use multiple Aspose products in your application, such as **Aspose.Words** and **Aspose.Cells**, here are a few useful tips:

* Set the License for each Aspose product separately. Even if you have a single license file for all components, for example, "Aspose.Total.lic", you still need to call **SetLicense** separately for each Aspose product that you use in your application.
* Use the Fully Qualified License Class Name. Each Aspose product has a **License** class in its own namespace. For example, Aspose.Words has [Aspose.Words.License](https://reference.aspose.com/words/java/com.aspose.words/license/) and `Aspose.Cells` has `Aspose.Cells`.License class. Using the fully qualified class name allows you to avoid confusion as to which license applies to which product.

---