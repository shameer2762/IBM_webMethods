# 🧩 webMethods Flow Steps

Flow steps are the building blocks of a Flow service in **webMethods Designer**.  
Each step defines a specific operation, control logic, or service invocation executed sequentially by the Integration Server.

---

## 🔹 1. SEQUENCE

**Purpose:**  
Executes the contained steps one after another in the specified order.

**Key Features:**
- Groups related steps logically.
- Controls exception handling using **Exit on** property (`SUCCESS`, `FAILURE`, `DONE`).
- Often used with **TRY-CATCH-FINALLY** for structured error handling.

**Common Properties:**

| Property | Description |
|-----------|-------------|
| Comments | Optional note for documentation. |
| Exit on | Defines when to stop executing contained steps. |
| Scope | Limits visible pipeline variables to a document. |
| Timeout | Maximum time allowed for this step. |

---

## 🔹 2. BRANCH

**Purpose:**  
Executes a child step based on a condition or matching value.

**Usage Types:**
- **Switch-based:** Executes matching label when a variable’s value equals label.
- **Expression-based:** Evaluates each label as an expression when `Evaluate labels = true`.

**Common Properties:**

| Property | Description |
|-----------|-------------|
| Switch | Variable name used for comparison. |
| Evaluate labels | Enables expression evaluation. |
| Label | Defines condition or label to match. |
| Default ($default) | Executes when no other condition matches. |

---

## 🔹 3. LOOP

**Purpose:**  
Iterates over elements of an array or document list and executes child steps for each item.

**Common Properties:**

| Property | Description |
|-----------|-------------|
| Input array | Array or document list to loop through. |
| Output array | Field where collected results are stored. |
| Label | Optional identifier for use with EXIT step. |

---

## 🔹 4. MAP

**Purpose:**  
Used for **data transformation** — copying, dropping, renaming, or transforming fields between pipeline variables.

**Key Capabilities:**
- Assign static values.
- Use built-in or custom transformers.
- Map complex documents or nested structures.

**Common Actions:**
- `Map Set Value`
- `Map Copy`
- `Map Drop`
- `Map Transformer`

---

## 🔹 5. INVOKE

**Purpose:**  
Calls another Flow, Java, or Adapter service.

**Common Properties:**

| Property | Description |
|-----------|-------------|
| Service | Fully qualified name of the service to invoke. |
| Validate input/output | Ensures pipeline matches service signature. |
| Timeout | Maximum execution time for invoked service. |

> 💡 **Tip:** Use **MAP** before and after INVOKE to prepare and clean pipeline data.

---

## 🔹 6. REPEAT

**Purpose:**  
Repeats a set of steps based on retry count or condition. Commonly used for transient error handling.

**Common Properties:**

| Property | Description |
|-----------|-------------|
| Repeat on success | Continue repeating even on success (rare). |
| Repeat on failure | Retry when child steps fail. |
| Count | Maximum number of retries. |
| Interval | Wait time between retries. |

---

## 🔹 7. EXIT

**Purpose:**  
Stops the execution of a loop, sequence, or the entire Flow service.

**Common Properties:**

| Property | Description |
|-----------|-------------|
| Exit from | `$flow`, `$loop`, `$parent`, or specific label. |
| Signal | `SUCCESS` or `FAILURE`. |
| Failure message | Optional message for logging. |

---

## 🔹 8. TRY / CATCH / FINALLY

**Purpose:**  
Implements structured error handling within Flow services.

### TRY
Encapsulates the main logic to execute.  
If any step fails, control passes to **CATCH**.

### CATCH
Defines recovery or compensation logic for exceptions.  
Can catch specific failure types or all.

### FINALLY
Executes cleanup actions regardless of success or failure (e.g., closing connections).

**Common Properties (for TRY/CATCH/FINALLY):**

| Property | Description |
|-----------|-------------|
| Comments | Optional description. |
| Scope | Restrict variable access to a document. |
| Timeout | Step timeout duration. |

---

## 🔹 9. SEQUENCE with TRY-CATCH Example

```mermaid
flowchart TD
    A[TRY Block] -->|Failure| B[CATCH Block]
    A -->|Success| C[FINALLY Block]
    B --> C
