# Aspose.Words for  - Reporting

---

**URL:** https://docs.aspose.com/words/java/linq-reporting-engine.md

**Contents:**
- How To Build a Report
- Create a Template
- Create a Data Source
- Build a Report

---
title: "Reporting"
---

Aspose.Words for  offers a powerful LINQ reporting engine that enables developers to generate dynamic, data-driven documents efficiently.

**How To Build a Report**

LINQ is a  feature that allows to query data from various sources (like collections, databases, XML, and more) using a syntax integrated directly into the language. It provides a consistent and readable way to filter, sort, group, and manipulate data.

Using Aspose.Words for  reporting featires, you can extract and structure data using LINQ, then pass that data to a template document for generating dynamic reports.

**Create a Template**

A typical LINQ template consists of common document content and simple text placeholders of the following format: <<[field_name_]>>.

This is an example for a document with the common text “Name:” and “Age:” and their corresponding placeholders:

Name: <<[manager.Name]>>
Age: <<[manager.Age]>>

You can also create placeholders not only for text data but also for images.

**Create a Data Source**

To build reports with Aspose.Words for , you can use CsvDataSource, JsonDataSource, and XmlDataSource.

The following code examples show how to create a data source using:

- CSV:
`CsvDataSource dataSource = new CsvDataSource(..., options);`

- Json:
`JsonDataSource dataSource = new JsonDataSource(..., options);`

- Xml:
`XmlDataSource dataSource = new XmlDataSource(..., options);;`

**Build a Report**

To build a report with LINQ syntax, use the one of the following JSON, XML, or Class data source as an example, and one of the BuildReport methods:

- JSON:

{
  manager: {
    Photo: "Photo.png",
    Name: "John Smith",
    Age: 37
  }
}

- XML:

<manager>
    <Photo>Photo.png</Photo>
    <Name>John Smith</Name>
    <Age>37</Age>
</manager>

- Class:

public class Manager
{
    public string Photo {  get; set; }
    public string Name { get; set; }
    public int Age { get; set; }
}

Load data from JSON and then build a report:

```java
Document doc = ...              // Loading a template document.
JsonDataSource dataSource = ... // Loading JSON.

ReportingEngine engine = new ReportingEngine();
engine.BuildReport(doc, dataSource, "managers");
```

---