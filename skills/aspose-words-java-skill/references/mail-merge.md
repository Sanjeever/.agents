# Aspose.Words for Java - Mail Merge

---

**URL:** https://docs.aspose.com/words/java/mail-merge-and-reporting.md

**Contents:**
- How Mail Merge Works
- Simple Mail Merge Operation
- Mail Merge with Regions
- Mail Merge Advanced Features

---
title: "Mail Merge"
---

**How Mail Merge Works**

Mail Merge is a process that combines a document template with data to create personalized documents in bulk. It works by inserting specific fields (e.g., names, addresses) into a template, which is then filled with data from a data source, generating multiple customized outputs.

It is important to understand that the output documents will differ only in the specific values in the merge fields.

Here are a few advantages of the Aspose.Words reporting solution:

- Design reports in Microsoft Word using standard Mail Merge fields
- Define regions in the document that are growing, such as detailed order rows
- Insert images during a mail merge
- Execute any custom logic, control formatting, or insert complex content using Mail Merge event handlers
- Fill in documents with data from any type of data source

**Simple Mail Merge Operation**

**Simple Mail Merge** creates personalized documents by merging a template with data. In the template, placeholders (called merge fields) represent dynamic content like names, addresses, or dates. During the merge process, these placeholders are replaced with the corresponding data from an external data source.

Follow these steps to **execute a simple mail merge operation**:

1. Prepare a template document with merge fields (e.g. `<<FirstName>>`, `<<Location>>`, `<<Quantity>>`)
2. Prepare data in the data source (DataRow, DataTable, DataSet, array of values)
3. Execute the merge operation to produce individual documents with personalized information using one of the [Execute methods](https://reference.aspose.com/words/java/com.aspose.words/mailmerge/#execute-java.lang.String---java.lang.Object)

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

doc.save(getArtifactsDir() + "BaseOperations.SimpleMailMerge.docx");
```

**Mail Merge with Regions**

**Mail merge with regions** is used when you need to insert dynamic content (such as tables with different numbers of rows or itemized lists of different lengths) for a specific area of the document. In this case, merge regions are used instead of simple merge fields - they define sections that are repeated for each data record. This is useful for documents like invoices with multiple line items or reports with dynamic tables.

Follow these steps to **execute a mail merge with regions**:

1. Define the merge region in the template using start and end markers (e.g., `<<TableStart:Items>>` and `<<TableEnd:Items>>`)
2. Connect the template to a structured data source with nested records
3. Perform the merge operation using one of the [ExecuteWithRegions](https://reference.aspose.com/words/net/aspose.words.mailmerging/mailmerge/executewithregions/#executewithregions/) methods

```java
private DataSet createDataSet() {
    // Create the customers table.
    DataTable tableCustomers = new DataTable("Customers");
    tableCustomers.getColumns().add("CustomerID");
    tableCustomers.getColumns().add("CustomerName");
    tableCustomers.getRows().add(new Object[]{1, "John Doe"});
    tableCustomers.getRows().add(new Object[]{2, "Jane Doe"});

    // Create the orders table.
    DataTable tableOrders = new DataTable("Orders");
    tableOrders.getColumns().add("CustomerID");
    tableOrders.getColumns().add("ItemName");
    tableOrders.getColumns().add("Quantity");
    tableOrders.getRows().add(new Object[]{1, "Hawaiian", 2});
    tableOrders.getRows().add(new Object[]{2, "Pepperoni", 1});
    tableOrders.getRows().add(new Object[]{2, "Chicago", 1});

    // Add both tables to a data set.
    DataSet dataSet = new DataSet();
    dataSet.getTables().add(tableCustomers);
    dataSet.getTables().add(tableOrders);

    // The "CustomerID" column, also the primary key of the customers table is the foreign key for the Orders table.
    dataSet.getRelations().add(tableCustomers.getColumns().get("CustomerID"), tableOrders.getColumns().get("CustomerID"));

    return dataSet;
}
```

```java
Document doc = new Document();
DocumentBuilder builder = new DocumentBuilder(doc);

// The start point of mail merge with regions the dataset.
builder.insertField(" MERGEFIELD TableStart:Customers");

// Data from rows of the "CustomerName" column of the "Customers" table will go in this MERGEFIELD.
builder.write("Orders for ");
builder.insertField(" MERGEFIELD CustomerName");
builder.write(":");

// Create column headers.
builder.startTable();
builder.insertCell();
builder.write("Item");
builder.insertCell();
builder.write("Quantity");
builder.endRow();

// We have a second data table called "Orders", which has a many-to-one relationship with "Customers"
// picking up rows with the same CustomerID value.
builder.insertCell();
builder.insertField(" MERGEFIELD TableStart:Orders");
builder.insertField(" MERGEFIELD ItemName");
builder.insertCell();
builder.insertField(" MERGEFIELD Quantity");
builder.insertField(" MERGEFIELD TableEnd:Orders");
builder.endTable();

// The end point of mail merge with regions.
builder.insertField(" MERGEFIELD TableEnd:Customers");

// Pass our dataset to perform mail merge with regions.
DataSet customersAndOrders = createDataSet();
doc.getMailMerge().executeWithRegions(customersAndOrders);

doc.save(getArtifactsDir() + "BaseOperations.MailMergeWithRegions.docx");
```

**Mail Merge Advanced Features**

**Mail Merge Advanced Features** allow you to customize the behavior of the mail merge process, controlling how data is processed, formatted, and displayed in the final document. These options provide flexibility for advanced scenarios and fine-tuning output.

Mail Merge Advanced Features: Mustache syntax, Mail merge with regions, Nested mail merge, Custom data sources, Mail merge callbacks, Field merging callbacks, Image merging, HTML formatting injection, Removing unused fields, Removing empty paragraphs, Conditional merging, Mapped fields, Dynamic table generation, Progress reporting, Data throttling.

**Mail Merge Advanced Features Example**

When you are using some long template that is created by someone else, you may want to delete all the merge fields that already exist in that template before performing a Mail Merge operation. You can use the [DeleteFields](https://reference.aspose.com/words/java/com.aspose.words/mailmerge/#deleteFields) method if you want to delete all merge fields from a document without executing a Mail Merge operation. This method is not affected by any removing options of the [CleanupOptions](https://reference.aspose.com/words/java/com.aspose.words/mailmerge/#getCleanupOptions) property and executing it only removes merged fields, not any containing fields or empty paragraphs.

---