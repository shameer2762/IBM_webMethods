# 🧾 Creating Flat File Document in webMethods Designer

This note explains **each minute detail** involved in creating a **Flat File Document**, including **Flat File Dictionary** and **Flat File Schema**, in webMethods Designer. These are crucial concepts for data exchange using structured text-based formats like `.csv`, `.txt`, or `.dat` files.

---

## 🧠 1. Understanding the Basics

Before you begin, understand what each component means:

### 📘 Flat File Dictionary
A **Flat File Dictionary** defines the *structure* of your flat file — the layout, field definitions, and data hierarchy (record, composite, and field levels).  
It acts as a **blueprint** that defines how the Integration Server should interpret or generate flat files.

### 📗 Flat File Schema
A **Flat File Schema** uses the dictionary to describe how a flat file is parsed or generated. It includes:
- The delimiter or fixed-length structure
- Record identification logic
- Record parsing rules (repeating, optional, composite, etc.)

### ⚙️ Purpose
The schema and dictionary together tell webMethods **how to parse incoming flat files** or **generate output flat files** from IS documents.

---

## IBM Documentation:: https://www.ibm.com/docs/en/webmethods-integration/wm-designer/11.1.0?topic=files-creating-flat-file-schemas

## 🧩 2. Steps to Create a Flat File Dictionary

1. **Open Designer → Package Navigator → Choose your Package**
2. Right-click on your folder → **New → Flat File Dictionary**
3. Enter a name → Click **Finish**.

### Inside the Dictionary Editor
The Flat File Dictionary has three main elements:
- **Record Definition**
- **Composite Definition**
- **Field Definition**

Let's understand each one.

#### 🔹 Record Definition
A **Record Definition** defines a group of fields that represent one line (or a section) of data in a flat file.  
It contains field definitions and can also have nested composites or child records.

Example: In an invoice flat file, `Header`, `Detail`, and `Trailer` could each be separate **Record Definitions**.

#### 🔹 Composite Definition
A **Composite Definition** groups multiple related record or field definitions together into a single structure.  
It is often used to define **repeating record groups**, such as multiple invoice line items under one header.

#### 🔹 Field Definition
A **Field Definition** defines the smallest unit of data — a single field within a record.  
Each field has:
- **Name**
- **Type** (string, integer, decimal, etc.)
- **Length** (for fixed-width files)
- **Position / Delimiter**
- **Optional/Required** property

#### 🔹 Record Identifier
A **Record Identifier** is a value or token used to **distinguish different record types** within a flat file.  
For example, in a flat file with mixed record types:
```
HDR|Invoice123|2025-10-20
DTL|Item1|100|5
DTL|Item2|200|3
TRL|END|2
```
Here:
- `HDR` → Header record identifier  
- `DTL` → Detail record identifier  
- `TRL` → Trailer record identifier

These identifiers are mapped under “**Record Definition → Record Parser**” settings.

## IBM Documentation: https://www.ibm.com/docs/en/webmethods-integration/wm-designer/11.1.0?topic=files-creating-flat-file-schemas
---



## 🧱 3. Steps to Create a Flat File Schema

1. In the same folder, right-click → **New → Flat File Schema**
2. Provide a name → Click **Finish**

### Configure Flat File Schema Properties

#### 🔸 Dictionary Reference
Link the dictionary you just created. This connects schema parsing logic to the structural definitions.

#### 🔸 Record Parser Type
Choose one:
- **Delimited** → Fields are separated by a delimiter (e.g., comma, pipe `|`, tab).
- **Fixed Length** → Each field occupies a fixed number of characters.

#### 🔸 Delimiter / Fixed Length Settings
If delimited, specify the delimiter (e.g., `,` or `|`).
If fixed-length, define the exact length per field in the dictionary.

#### 🔸 Record Definition Selection
Select the top-level record (e.g., “Header”) and map additional records as nested or repeating.

#### 🔸 Record Identification
Under each record definition, specify the **Record Identifier** (e.g., “HDR”, “DTL”).

#### 🔸 Repeating and Optional Records
You can configure whether a record can repeat or be optional.

---

## 🧩 4. Using the Schema in Flow Services

After creating the schema, you can use the following built-in services:

| Service | Purpose |
|----------|----------|
| `pub.flatFile:convertToValues` | Converts flat file data (string) into an IS document using the schema |
| `pub.flatFile:convertToString` | Converts an IS document into a flat file string |
| `pub.flatFile:getSchema` | Retrieves schema details dynamically |

---

## 💡 Example

Suppose you have this flat file content:

```
HDR|Invoice123|2025-10-20
DTL|Item1|100|5
DTL|Item2|200|3
TRL|END|2
```

### Step 1: Create Dictionary
Define:
- `Header` record with fields: `RecordType`, `InvoiceNumber`, `Date`
- `Detail` record with fields: `RecordType`, `ItemName`, `Price`, `Quantity`
- `Trailer` record with fields: `RecordType`, `EndFlag`, `Count`

### Step 2: Assign Identifiers
- Header → Identifier: `HDR`
- Detail → Identifier: `DTL`
- Trailer → Identifier: `TRL`

### Step 3: Create Schema
- Link to the dictionary
- Set delimiter = `|`
- Configure top-level records: Header, Detail (repeating), Trailer

---

## 🧠 Interview Tip

| Topic | Common Question | Expected Answer |
|--------|------------------|----------------|
| Flat File Dictionary | What is the difference between Record Definition and Composite Definition? | Record defines one logical line; Composite groups multiple related records. |
| Schema | How does the schema identify record types? | Through record identifiers defined in each record. |
| Services | Which services are used to convert between flat file and document? | `pub.flatFile:convertToValues` and `pub.flatFile:convertToString` |

---

## 🏁 Summary

- **Flat File Dictionary** defines structure and field layout.  
- **Flat File Schema** defines parsing rules, delimiters, and record hierarchy.  
- Both are essential for converting flat files ↔ IS documents in Integration Server.

---

> ✅ **Best Practice:** Always test your schema using a sample flat file in Designer before using it in production flow services.
