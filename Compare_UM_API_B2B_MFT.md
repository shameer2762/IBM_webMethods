# webMethods Integration Patterns – API vs UM vs B2B vs MFT

## 1. Introduction

In enterprise integration, the business requirement may remain the same, but the communication method can differ.

For example, a customer wants to place an order in SAP S/4HANA.

The order can reach SAP through different integration patterns:

1. API Integration
2. Universal Messaging (UM)
3. B2B Integration
4. Managed File Transfer (MFT)

All four can ultimately help transfer order information into SAP. However, each is designed to solve a different integration requirement.

**Important:** These technologies are not necessarily alternatives to one another. They can also be combined in a single integration architecture.

---

# 2. Quick Comparison – All Four Technologies

| Aspect | API Integration | Universal Messaging (UM) | B2B Integration | MFT |
|---|---|---|---|---|
| Full form | Application Programming Interface | Universal Messaging | Business-to-Business | Managed File Transfer |
| Main purpose | Application-to-application communication | Message exchange between applications | Business document exchange between organizations | Secure file transfer between systems or organizations |
| Communication | HTTP/HTTPS | Messaging protocols / JMS | AS2, SFTP, VAN, HTTPS, etc. | SFTP, FTPS, HTTPS, etc. |
| Data format | JSON, XML | Messages / Events | X12, EDIFACT, XML, etc. | CSV, XML, JSON, TXT, ZIP, etc. |
| Interaction | Synchronous or asynchronous | Usually asynchronous | Usually asynchronous document exchange | Usually asynchronous / batch |
| Sender | Website, mobile app, ERP, another application | Application / message producer | External trading partner | External system / file-producing application |
| Receiver | API provider | Message consumer / subscriber | Trading partner | File destination / receiving system |
| Typical webMethods components | API Gateway + IS | UM + IS | B2B + IS | Active Transfer / MFT + IS |
| Main advantage | Direct application communication | Decoupling and message buffering | Standardized partner communication | Secure and managed file exchange |
| Main limitation | API alone is not a durable messaging queue | Additional messaging infrastructure | Partner onboarding and EDI complexity | File-based processing is less immediate |

### Remember the main distinction

| Technology | Main question it answers |
|---|---|
| API | How can one application communicate with another application? |
| UM | How can applications exchange messages asynchronously and independently? |
| B2B | How can two different organizations exchange standardized business documents? |
| MFT | How can organizations or systems securely exchange files? |
| IS | How do we implement integration logic such as validation, mapping and routing? |

---

# 3. API Integration

## 3.1 What is API Integration?

API integration allows one application to communicate with another application through a defined interface.

For example:

- Customer opens an e-commerce website.
- Customer selects a product.
- Customer clicks Place Order.
- Website sends an HTTP REST API request.
- Integration Server processes the request.
- SAP S/4HANA creates the sales order.

The customer normally interacts with the website or mobile application, not directly with Integration Server.

## 3.2 Architecture

```text
Customer
   |
   v
E-commerce Website
   |
   | REST API (JSON)
   v
API Gateway
   |
   | Authentication / Authorization
   | Rate Limiting / API Policies
   v
Integration Server
   |
   | Validation
   | Data Transformation
   | Business Logic
   v
SAP S/4HANA
   |
   | Response
   v
Integration Server
   |
   v
Customer Website
```

## 3.3 What does each component do?

| Component | Responsibility |
|---|---|
| Website | Provides the customer interface |
| REST API | Defines how the website communicates with the backend |
| API Gateway | Secures and manages API access |
| Integration Server | Executes integration logic |
| SAP S/4HANA | Creates and manages the sales order |

Note: API Gateway is useful when centralized API security and governance are required. Not every API integration must necessarily pass through API Gateway.

## 3.4 Example

Customer sends this JSON request:

```json
{
  "orderId": "ORD1001",
  "customerId": "C1001",
  "material": "LAPTOP01",
  "quantity": 2
}
```

Integration Server:

1. Receives the API request.
2. Validates customer information.
3. Validates material information.
4. Converts customer and material numbers if required.
5. Prepares SAP-compatible data.
6. Calls the SAP API.
7. Processes the response.
8. Returns the result to the caller.

## 3.5 Advantages of API Integration

| Advantage | Explanation |
|---|---|
| Real-time communication | Caller can receive an immediate response |
| Standardized interface | Applications communicate through defined API contracts |
| Reusability | Multiple applications can consume the same API |
| Security | API Gateway supports authentication and authorization policies |
| Easy application integration | Websites, mobile applications and enterprise applications can consume APIs |
| Monitoring | API usage, errors and traffic can be monitored |

## 3.6 Disadvantages of API Integration

| Disadvantage | Explanation |
|---|---|
| Dependency | Synchronous calls may depend on backend availability |
| Timeout | Caller may experience timeout if the backend takes too long |
| Performance | Heavy traffic can increase backend load |
| Failure propagation | A downstream failure can affect the calling application |
| Retry handling | Retry and duplicate-request handling need proper design |
| No built-in queue | An API endpoint alone does not provide durable message buffering |

## 3.7 When should we use API Integration?

| Scenario | Why API is suitable |
|---|---|
| Customer places an order through an e-commerce website | Website needs to communicate with backend |
| Mobile application requests customer details | Application needs immediate data |
| Application retrieves product information from SAP | Request-response communication |
| External partner wants to consume company services | API provides a standardized interface |
| Customer needs immediate validation | API can return validation results |

---

# 4. Universal Messaging (UM)

## 4.1 What is UM?

Universal Messaging is messaging middleware.

It allows applications to exchange messages without requiring the producer and consumer to communicate at exactly the same time.

It supports messaging patterns such as:

- Queue-based messaging
- Publish/Subscribe
- Asynchronous communication
- Persistent messaging (when configured)
- Message acknowledgement
- Message expiry / TTL
- Clustering

UM acts as an intermediate messaging layer between applications.

## 4.2 Architecture – API + UM + SAP

Imagine a customer places an order on an e-commerce website.

Instead of directly waiting for SAP to process the order, Integration Server publishes the order into a UM queue.

A separate consumer processes the message later.

```text
Customer
   |
   v
E-commerce Website
   |
   | REST API
   v
API Gateway
   |
   v
Integration Server
   |
   | Publish Order Message
   v
Universal Messaging
   |
   | Order Queue
   |
   v
Integration Server Consumer
   |
   | Validate / Transform
   v
SAP S/4HANA
   |
   v
Sales Order Created
```

## 4.3 How does it work?

1. Customer places an order.
2. Website sends a REST API request.
3. Integration Server receives the request.
4. IS performs initial validation.
5. IS publishes the order message to UM.
6. UM stores the message according to the queue and persistence configuration.
7. IS returns an acknowledgement to the customer.
8. A separate IS consumer receives the message.
9. Consumer transforms the order into SAP-compatible format.
10. Consumer sends the order to SAP.
11. SAP creates the sales order.

The customer doesn't directly use UM.

The customer uses the website or API. UM operates behind the scenes.

## 4.4 Why do we need UM when APIs also support asynchronous communication?

This is an important interview question.

APIs can certainly support asynchronous communication.

For example:

```text
Customer Website
       |
       | REST API
       v
API Gateway
       |
       v
Integration Server
       |
       | Publish Message
       v
Universal Messaging
       |
       | Return HTTP 202
       v
Customer Website
```

The customer receives an acknowledgement without waiting for SAP.

A separate consumer processes the order.

The important distinction:

**API provides the application interface, whereas UM provides the messaging infrastructure.**

An asynchronous API needs some backend mechanism to store and process accepted work. This could be UM, another message broker, or a durable job store.

## 4.5 Advantages of UM

| Advantage | Explanation |
|---|---|
| Decoupling | Producer and consumer operate independently |
| Message buffering | Messages can wait until consumers are ready |
| Asynchronous processing | Sender does not need to wait for backend processing |
| Publish/Subscribe | Multiple subscribers can receive events |
| JMS support | Applications can use standard JMS interfaces |
| Persistence | Configured persistent messages can survive certain failures and restarts |
| Acknowledgement | Consumer can acknowledge message processing |
| TTL | Messages can have a defined validity period |
| Scalability | Consumers and messaging infrastructure can be scaled according to architecture |

## 4.6 Disadvantages of UM

| Disadvantage | Explanation |
|---|---|
| Infrastructure complexity | Requires realms, queues, connections, permissions and monitoring |
| Operational overhead | Messaging infrastructure needs administration |
| Queue buildup | Messages may accumulate when consumers cannot process them |
| Duplicate handling | Applications must handle retries and possible duplicate delivery |
| Delayed processing | Customer may not receive the final business result immediately |
| Additional cost | Infrastructure and maintenance resources are required |

Important: Persistence does not automatically guarantee exactly-once business processing. Idempotency, acknowledgement strategy and failure handling must also be designed.

## 4.7 When should we use UM?

| Scenario | Why UM is suitable |
|---|---|
| Customer submits orders, but SAP processing happens later | Asynchronous processing |
| SAP is temporarily unavailable | Messages can be buffered according to configuration |
| Thousands of orders arrive simultaneously | Buffering and controlled consumption |
| Multiple applications need the same order event | Publish/Subscribe |
| Producer and consumer should operate independently | Decoupling |
| JMS-based integration is required | JMS messaging support |

---

# 5. B2B Integration

## 5.1 What is B2B?

B2B means Business-to-Business integration.

It enables two separate organizations to exchange business documents electronically.

For example:

- Walmart sends a purchase order to a supplier.
- The supplier receives the order through its B2B platform.
- The supplier processes the order in SAP.
- The supplier sends an acknowledgement back to Walmart.

The customer usually uses its own ERP, EDI system, B2B provider or partner portal.

The customer does not need access to the supplier's SAP system.

## 5.2 Architecture

```text
Customer Company
       |
       v
Customer ERP
(SAP / Oracle / Other ERP)
       |
       | Generate EDI 850
       v
Customer EDI System
       |
       | AS2 / SFTP / VAN
       v
webMethods B2B
       |
       | Trading Partner Identification
       | EDI Validation
       | Document Processing
       v
Integration Server
       |
       | Mapping / Transformation
       v
SAP S/4HANA
       |
       | Sales Order Created
       v
Integration Server / B2B
       |
       | EDI 855 Acknowledgement
       v
Customer ERP
```

## 5.3 How does it work?

Imagine Walmart wants to purchase 1,000 laptops from your company.

1. Walmart creates a purchase order in its ERP.
2. Walmart's system generates an X12 850 document.
3. Walmart sends the document through an agreed protocol such as AS2.
4. Your B2B platform receives the document.
5. B2B identifies Walmart as the trading partner.
6. B2B validates and processes the EDI document.
7. Integration Server maps the EDI data into SAP-compatible format.
8. SAP S/4HANA creates the sales order.
9. Your company sends an X12 855 acknowledgement to Walmart.

## 5.4 Common EDI documents

| EDI Document | Business Meaning |
|---|---|
| X12 850 | Purchase Order |
| X12 855 | Purchase Order Acknowledgement |
| X12 860 | Purchase Order Change |
| X12 810 | Invoice |
| X12 856 | Advance Ship Notice |
| EDIFACT ORDERS | Purchase Order |
| EDIFACT ORDRSP | Purchase Order Response |
| EDIFACT INVOIC | Invoice |

## 5.5 Advantages of B2B

| Advantage | Explanation |
|---|---|
| Standardization | Supports standards such as X12 and EDIFACT |
| Trading partner management | Maintains partner-specific configurations |
| Document validation | Checks incoming business documents |
| Document tracking | Tracks partner document processing |
| Acknowledgements | Supports business and technical acknowledgements |
| Automation | Reduces manual exchange of purchase orders and invoices |
| Partner scalability | Helps manage multiple external trading partners |

## 5.6 Disadvantages of B2B

| Disadvantage | Explanation |
|---|---|
| Complex onboarding | Every trading partner requires configuration |
| EDI mapping | Different partners may have different implementation requirements |
| Testing | Partner-specific testing is required |
| Maintenance | Document versions and partner rules may change |
| Error handling | Technical and business validation errors need separate handling |
| Initial effort | Certificates, identifiers, protocols and document agreements require setup |

## 5.7 When should we use B2B?

| Scenario | Why B2B is suitable |
|---|---|
| Walmart sends purchase orders to a supplier | Standardized EDI exchange |
| Supplier sends invoices to retailers | Business document exchange |
| Manufacturer communicates with distributors | Trading partner integration |
| Retailer sends X12 850 | EDI processing |
| Supplier returns X12 855 | EDI acknowledgement |
| International partners exchange EDIFACT documents | Standardized cross-company communication |

**Important:** B2B is not restricted to large organizations. Smaller companies can also participate through EDI service providers and partner portals.

---

# 6. MFT – Managed File Transfer

## 6.1 What is MFT?

MFT stands for Managed File Transfer.

It is used to securely transfer files between organizations, systems or locations.

For example, a distributor may generate a CSV file containing 5,000 orders and upload it to the supplier's SFTP server.

The supplier's MFT platform receives and manages the file transfer.

Integration Server can then process the file contents.

## 6.2 Architecture

```text
Customer / Distributor
       |
       v
Customer ERP
       |
       | Generate CSV / XML
       v
Customer SFTP
       |
       | Secure File Transfer
       v
Active Transfer / MFT
       |
       | Transfer Monitoring
       | File Routing
       v
Integration Server
       |
       | Read File
       | Parse Records
       | Validate Data
       | Transform Data
       v
SAP S/4HANA
       |
       v
Sales Orders Created
```

## 6.3 How does it work?

1. Customer generates a CSV order file.
2. Customer uploads the file to the agreed SFTP endpoint.
3. Active Transfer receives or manages the file transfer.
4. File is routed to the appropriate destination.
5. Integration Server picks up the file.
6. IS parses and validates the records.
7. IS transforms the data into SAP-compatible format.
8. SAP receives the order data.
9. Processing status is recorded.

## 6.4 Advantages of MFT

| Advantage | Explanation |
|---|---|
| Secure transfer | Supports secure transfer protocols |
| Large files | Suitable for large files and batch processing |
| Automation | Supports scheduled file exchange |
| Tracking | Provides transfer status and audit information |
| Reliability | Provides managed transfer and recovery capabilities |
| Multiple formats | Supports CSV, XML, TXT, ZIP and other file types |
| Less custom development | Reduces the need for unmanaged FTP scripts |

## 6.5 Disadvantages of MFT

| Disadvantage | Explanation |
|---|---|
| Batch-oriented | Processing may not be immediate |
| File dependency | Requires agreed file formats and naming conventions |
| Duplicate files | Duplicate detection must be designed |
| Partial uploads | File completion must be handled correctly |
| File processing | MFT does not automatically understand business data |
| Large-file handling | May require splitting, compression or special processing |

## 6.6 When should we use MFT?

| Scenario | Why MFT is suitable |
|---|---|
| Distributor sends daily CSV files | Batch file processing |
| Partner sends XML files through SFTP | Secure file transfer |
| Large data files are exchanged | Managed transfer |
| Partner doesn't support APIs or EDI | File-based communication |
| Scheduled data exchange is required | Automated file transfer |
| Secure transfer tracking is required | MFT monitoring and audit |

---

# 7. Advantages and Disadvantages – Overall Comparison

| Technology | Advantages | Disadvantages |
|---|---|---|
| API | Real-time interaction, standardized interface, reusable services, API security | Synchronous dependency, timeouts, failure propagation, no built-in durable queue |
| UM | Decoupling, buffering, asynchronous processing, publish-subscribe, JMS | Infrastructure complexity, queue monitoring, duplicate handling, delayed response |
| B2B | EDI standardization, trading partner management, document validation, acknowledgements | Complex onboarding, mapping effort, partner-specific maintenance |
| MFT | Secure file transfer, large-file support, scheduling, tracking, automation | Batch delays, file dependencies, duplicate and partial-file handling |

---

# 8. Decision Table – Which Technology Should We Use?

| Business Requirement | Recommended Technology | Reason |
|---|---|---|
| Customer places an order through an e-commerce website and needs an immediate response | API | Request-response communication |
| Mobile app retrieves customer details from SAP | API | Direct application communication |
| Customer submits an order and processing happens later | API + UM | API receives the request; UM buffers the message |
| SAP is temporarily unavailable and orders should be processed later | UM | Asynchronous message buffering |
| Multiple systems need to receive OrderCreated events | UM | Publish/Subscribe |
| Walmart sends a purchase order in X12 850 format | B2B | EDI business document exchange |
| Supplier sends X12 855 acknowledgement | B2B | Partner document exchange |
| Distributor sends daily CSV files | MFT | Batch file transfer |
| Partner uploads XML files through SFTP | MFT | Secure file transfer |
| External application needs secure access to company APIs | API Gateway + IS | API security and governance |

---

# 9. Can These Technologies Work Together?

Yes. This is very important in enterprise architecture.

They solve different problems and can be combined.

## 9.1 B2B + UM + IS

Example: Walmart sends an EDI 850 purchase order.

```text
Walmart ERP
     |
     | EDI 850
     v
B2B Platform
     |
     | Validate / Parse
     v
Integration Server
     |
     | Publish Message
     v
Universal Messaging
     |
     | Consume Message
     v
Integration Server
     |
     v
SAP S/4HANA
```

Responsibilities:

- B2B: Receives and processes EDI documents.
- IS: Performs mapping and business logic.
- UM: Buffers and distributes messages.
- SAP: Creates the sales order.

## 9.2 API Gateway + IS + UM

Example: E-commerce website submits an order.

```text
Website
   |
   | REST API
   v
API Gateway
   |
   v
Integration Server
   |
   | Publish Order
   v
Universal Messaging
   |
   v
IS Consumer
   |
   v
SAP S/4HANA
```

Responsibilities:

- API Gateway: Secures the API.
- IS: Receives and validates the request.
- UM: Buffers the order message.
- IS Consumer: Processes the order.
- SAP: Creates the sales order.

## 9.3 MFT + IS + API

Example: Distributor sends a CSV file.

```text
Distributor
     |
     | CSV File
     v
SFTP / MFT
     |
     v
Integration Server
     |
     | Parse / Validate
     v
SAP API
     |
     v
SAP S/4HANA
```

Responsibilities:

- MFT: Transfers the file.
- IS: Parses and transforms file contents.
- SAP API: Accepts the order data.
- SAP: Creates the sales orders.

---

# 10. Practical Developer Perspective

When you receive an integration requirement, ask these questions before choosing the technology.

| Question | What it helps determine |
|---|---|
| Who is sending the data? | Customer application, internal system or trading partner |
| What is the source format? | JSON, XML, EDI or file |
| How will the data be transported? | HTTP, JMS, AS2, SFTP, etc. |
| Does the sender need an immediate business response? | Synchronous or asynchronous design |
| Do we need message buffering? | Whether UM or another messaging mechanism is appropriate |
| Do multiple applications need the same event? | Publish/Subscribe requirements |
| Are EDI standards involved? | B2B requirements |
| Are scheduled files involved? | MFT requirements |
| Which system owns the final business transaction? | SAP or another backend |

## Example: SAP S/4HANA Order Integration

| Customer Scenario | Entry Point | Integration Architecture |
|---|---|---|
| Individual customer uses an e-commerce website | REST API | API Gateway → IS → SAP |
| Customer places an order, but processing is deferred | REST API | API Gateway → IS → UM → IS Consumer → SAP |
| Walmart sends an EDI purchase order | EDI 850 | B2B → IS → SAP |
| Distributor sends CSV order files | SFTP | MFT → IS → SAP |

---

# 11. Interview Questions and Answers

### Q1. What is the difference between API integration and UM?

API integration exposes or consumes an application interface, commonly using REST or SOAP.

UM is messaging middleware used to publish and consume messages asynchronously.

API and UM can work together. For example, IS can receive an API request and publish the resulting message into UM for asynchronous processing.

### Q2. Why do we need UM if APIs support asynchronous communication?

APIs can support asynchronous communication, but an API alone does not provide durable messaging infrastructure.

UM provides messaging capabilities such as queues, topics, persistence, acknowledgements and publish-subscribe.

We use UM when these messaging capabilities are required.

### Q3. What is the difference between B2B and MFT?

B2B focuses on exchanging business documents between trading partners, including EDI standards, partner identification, validation and acknowledgements.

MFT focuses on secure and managed file transfer.

A B2B document can also be transported through SFTP, so both technologies can be used together.

### Q4. Does a B2B customer need access to our Integration Server or SAP?

Normally, no.

The trading partner sends a business document through an agreed communication channel.

Our B2B platform receives and processes the document, and the internal integration layer communicates with SAP.

### Q5. Does UM replace Integration Server?

No.

UM provides messaging infrastructure.

Integration Server executes business logic such as validation, transformation, routing and backend invocation.

They complement each other.

### Q6. Does MFT process the business data inside a CSV file?

MFT primarily manages file transfer.

Integration Server or another processing application normally reads the file, parses records, validates data and maps it into the target system.

### Q7. Can B2B and UM be used together?

Yes.

B2B can receive and validate an EDI document. The resulting message can be published to UM for asynchronous processing by downstream consumers.

### Q8. Is B2B used only by large organizations?

No.

Large organizations commonly use EDI because of their trading partner volumes, but smaller organizations can also participate through EDI providers, managed services and partner portals.

### Q9. Is API always synchronous?

No.

APIs can support asynchronous processing too.

For example, an API can accept a request, return HTTP 202 Accepted and allow processing to happen in the background.

### Q10. When should we avoid using UM?

When the integration is simple, the volume is manageable, immediate request-response is required, and there is no requirement for messaging, buffering or decoupling.

Adding UM without a business or technical requirement creates unnecessary infrastructure complexity.

---

# 12. Final Revision – Remember These Five Lines

| Technology | Remember it as |
|---|---|
| API | Application communicates through an interface |
| UM | Applications exchange messages through messaging infrastructure |
| B2B | Organizations exchange standardized business documents |
| MFT | Systems and organizations transfer files securely |
| IS | Executes integration logic: validate, transform, route and invoke |

## Final Architectural Rule

Do not select a product simply because it exists in the webMethods suite.

Select it based on:

1. Business communication requirement.
2. Source and target data formats.
3. Transport mechanism.
4. Synchronous or asynchronous processing needs.
5. Reliability and message-delivery requirements.
6. Trading partner requirements.
7. File-transfer requirements.
8. Monitoring, scalability and operational requirements.

**The most important concept:**

The business process may remain the same, but the integration architecture changes depending on how the systems need to communicate.

API, UM, B2B and MFT are different integration capabilities that can work independently or together to achieve the same business objective.
