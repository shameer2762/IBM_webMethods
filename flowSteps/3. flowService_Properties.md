# 🧩 Flow Service Properties in webMethods Designer

When a **Flow Service** is created in **webMethods Designer**, it comes with a set of configurable properties that control **how the service behaves**, **who can access it**, **how it handles errors**, **logging**, and **performance behavior**.

This document provides a clear and detailed explanation of each property, including all possible values and their meanings.

---

## ⚙️ 1. General

The **General** section defines the **basic access permissions**, **reusability**, and **metadata** of the service.

### 🔹 Permissions (List, Read, Write, Execute)
Permissions determine **who can perform specific actions** on the service using **Access Control Lists (ACLs)**.

| Property | Description | Common ACL Values |
|-----------|--------------|-------------------|
| **List ACL** | Defines who can **see** the service in Designer or IS Admin. | `Default`, `Developers`, `Administrators` |
| **Read ACL** | Defines who can **view** the service code. | `Default`, `Developers` |
| **Write ACL** | Defines who can **modify** or **edit** the service. | `Developers`, `Administrators` |
| **Execute ACL** | Defines who can **invoke** (run) the service. | `Default`, `Anonymous`, `Developers`, `Administrators` |

**Example:**  
- Developers may have `Write` access,  
- End users or other systems may only have `Execute` access.

---

### 🔹 Reuse (Private / Public)
Specifies **whether the service can be reused by other packages or external systems**.

| Option | Description |
|---------|--------------|
| **Private** | The service can only be called from within its own package or folder. |
| **Public** | The service can be called from any other package or invoked externally. |

**Example:**  
Internal helper services → *Private*  
Main business services (used by other apps) → *Public*

---

### 🔹 Source URI
This is a **free-text field** used for documentation.  
It can store a link or a path to where the service originated from — for example, a **Git repository link**, **design document**, or **external reference**.

---

## 🚀 2. Runtime

The **Runtime** section defines **how the flow service executes** on the Integration Server — including caching, loading behavior, debugging, and HTTP access.

### 🔹 Stateless (True / False)
Controls whether the service **remembers session data** between invocations.

| Value | Description |
|--------|--------------|
| **True** | The service does not maintain session data. Each execution is independent. *(Recommended for performance)* |
| **False** | The service maintains session context between invocations. *(Used rarely for session-based processing)* |

---

### 🔹 Cache Results (True / False)
Specifies whether the output of the service should be **stored in memory** for reuse.

| Value | Description |
|--------|--------------|
| **True** | Enables caching. Subsequent calls with the same inputs will return the same output without re-executing logic. |
| **False** | The service will execute its logic every time. |

**Use Case:**  
When output doesn’t change frequently (e.g., static configuration data), caching improves performance.

---

### 🔹 Cache Expire
Defines the **duration (in seconds)** after which the cached result becomes invalid.  
After this period, the next call will re-execute the service and refresh the cache.

**Example:**  
- Value: `600` → Cache valid for 10 minutes.

---

### 🔹 Reset Cache
A **manual action button** to clear the existing cache.  
Used when you update the service or want to clear old cached data immediately.

---

### 🔹 Prefetch (True / False)
Controls whether the service should be **preloaded into memory** when the Integration Server starts.

| Value | Description |
|--------|--------------|
| **True** | The service is preloaded during server startup for faster execution. |
| **False** | The service loads only when it is first executed. |

**Best Practice:**  
Set `True` for frequently used or critical services.

---

### 🔹 Prefetch Activation
Defines **when** the service should be preloaded during server startup.  
- Default value = `1`  
- Lower values load earlier, higher values load later.

> Example: Prefetch Activation = 2 → Service loads after priority 1 services.

---

### 🔹 Execution Locale
Defines the **locale (language and region settings)** that the service should use during execution.  
If no locale is defined, it uses the default server locale.

| Option | Description |
|---------|--------------|
| **[$null] No Locale Policy** | Uses the default locale configured in Integration Server. |
| **Custom Locale (e.g., en_US, fr_FR)** | Forces the service to use a specific locale (for date/time formatting, etc.). |

---

### 🔹 HTTP URL Alias
A **custom URL path** that allows accessing the service with a friendly or shorter URL.  
It is useful when exposing services as HTTP or REST endpoints.

**Example:**  
Instead of calling:  
`http://server:5555/invoke/project:getDetails`  
You can define alias: `/api/getDetails`  

Then call:  
`http://server:5555/api/getDetails`

---

### 🔹 Pipeline Debug
Controls **how input and output pipeline data is handled** during debugging or simulation.

| Option | Description |
|---------|--------------|
| **None** | No pipeline data is stored. *(Default)* |
| **Save** | Saves the current input/output pipeline for debugging later. |
| **Restore (Override)** | Loads saved pipeline and replaces existing values. |
| **Restore (Merge)** | Loads saved pipeline and merges with current data. |

**Use Case:**  
Helps developers test or debug services without re-running real systems.

---

### 🔹 Default XML Format
Specifies how XML data should be **processed and represented** in the pipeline.

| Option | Description |
|---------|--------------|
| **bytes** | Treats XML as raw byte data (no parsing). |
| **enhanced** | Parses XML with additional details and metadata. |
| **node** | Represents XML as a node object (DOM tree). |
| **stream** | Processes XML as a stream — useful for large XML files. |

**Tip:**  
Use **stream** for large data to avoid memory issues.

---

### 🔹 Allowed HTTP Methods
Defines which **HTTP request types** can invoke this service.  
Used when exposing the service as a **RESTful API** or HTTP endpoint.

**Possible values:**
- `GET` – Retrieve data.
- `POST` – Submit or create data.
- `PUT` – Update existing data.
- `DELETE` – Remove data.
- `PATCH` – Partially update data.
- `OPTIONS` / `HEAD` – For metadata or testing endpoints.

---

## ⚠️ 3. Transient Error Handling

Defines how the service should **retry automatically** when temporary (transient) errors occur, such as:
- Network interruptions
- Connection timeouts
- Temporary unavailability of external systems

| Property | Description | Example |
|-----------|--------------|----------|
| **Max Retry Attempt** | Number of times to retry the service after a transient error. | `3` = retry 3 times before failing. |
| **Retry Interval** | Time gap (in milliseconds or seconds) between retries. | `5000` = retry every 5 seconds. |

> This helps ensure **reliability** in integrations by retrying failed calls automatically.

---

## 🧾 4. Audit

Defines how and when service execution details are **logged for auditing** in My webMethods Server (MWS).  
Auditing helps track service activity, performance, and errors.

| Property | Description | Options |
|-----------|--------------|----------|
| **Enable Auditing** | Controls when the service should be audited. | - **Never** – No audit logs created.<br>- **When top-level service only** – Audits only when run directly.<br>- **Always** – Logs every execution (recommended for critical flows). |
| **Log On** | Defines when logs are written. | - **Error Only** – Log only failures.<br>- **Error and Success** – Log both success and failures.<br>- **Error, Success, and Start** – Log full execution details. |
| **Include Pipeline** | Specifies whether to include the service input/output data in audit logs. | - **Never** – Do not include pipeline.<br>- **On Errors Only** – Include only for failed runs.<br>- **Always** – Include pipeline every time. |

**Tip:**  
Enable auditing only where needed — full pipeline logging can increase storage usage.

---

## 🌍 5. Universal Name

The **Universal Name** uniquely identifies a service across different Integration Server environments (e.g., DEV, TEST, PROD).

| Property | Description |
|-----------|--------------|
| **Namespace Name** | Defines a global namespace (like a logical grouping or organization domain). |
| **Local Name** | Defines the specific name of the service within that namespace. |

**Example:**  
- Namespace: `com.company.sales`  
- Local Name: `orderService`  
- Universal Name: `com.company.sales:orderService`

This helps when synchronizing or deploying packages across environments.

---

## 🧱 6. Output Template

Defines how the **service output is formatted** when displayed or returned to a client (e.g., browser or API consumer).

| Property | Description |
|-----------|--------------|
| **Name** | The name of the output template (e.g., `resultTemplate`). |
| **Type** | Format type — typically `html`. |
| **Template Source** | The actual HTML or file path containing the output design. |

**Example:**  
You can design an HTML template to display order details nicely formatted on a web page.

---

## 🧍‍♂️ 7. Concurrent Requests Limit

Used to **limit the number of simultaneous executions** of the service.  
Helps prevent overloading the server when multiple clients call the service at once.

| Property | Description | Example |
|-----------|--------------|----------|
| **Enabled for Top-Level Service** | If set to `True`, the limit applies only to top-level invocations. | `True` |
| **Max Concurrent Requests** | Maximum number of service executions allowed simultaneously. | `10` (means 10 users can run at a time) |

**Example:**  
If `Max Concurrent Requests = 5`, the 6th request will wait until one of the earlier executions finishes.

---

## ✅ Summary Table

| Section | Purpose | Key Highlights |
|----------|----------|----------------|
| **General** | Basic configuration and permissions | ACLs, reusability, metadata |
| **Runtime** | Execution behavior and performance | Caching, debugging, HTTP access, XML handling |
| **Transient Error Handling** | Fault tolerance | Retries on network or system errors |
| **Audit** | Logging and monitoring | Tracks service executions in MWS |
| **Universal Name** | Identification | Unique global reference for deployments |
| **Output Template** | Presentation | Formats service output for display |
| **Concurrent Requests Limit** | Load control | Restricts simultaneous service executions |

---

🧠 **Tip:**  
While most defaults work fine, properties like **Stateless**, **Cache Results**, **Audit**, and **Concurrent Requests Limit** can be tuned carefully for **performance optimization** and **better governance** in production.

---
