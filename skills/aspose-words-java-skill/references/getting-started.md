# Aspose.Words for Java - Getting started

---

**URL:** https://docs.aspose.com/words/java/system-requirements.md

**Contents:**
- Supported Operating Systems
- Supported Cloud Platforms
- Supported Implementations

---
title: "System Requirements"
---

**Supported Operating Systems**

Aspose.Words for Java is compatible with the following operating systems:

* Windows:
    * Windows 2003 Server (x64, x86)
    * Windows 2008 Server (x64, x86)
    * Windows 2012 Server (x64, x86)
    * Windows 2012 R2 Server (x64, x86)
    * Windows 2016 Server (x64, x86)
    * Windows 2019 Server (x64, x86)
    * Windows XP (x64, x86)
    * Windows Vista (x64, x86)
    * Windows 7 (x64, x86)
    * Windows 8, 8.1 (x64, x86)
    * Windows 10 (x64, x86)
    * Windows 11 (x64, x86)
* Linux (any modern distribution supporting Java)
* macOS version 10.9 (Mavericks) and later

**Supported Cloud Platforms**

Aspose.Words for Java supports any cloud platforms that can run Java applications.

---

**URL:** https://docs.aspose.com/words/java/installation.md

**Contents:**
- Installing via NuGet Package Manager
- Installing via Installer

---
title: "Installation"
---

**Installing via NuGet Package Manager**

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

---