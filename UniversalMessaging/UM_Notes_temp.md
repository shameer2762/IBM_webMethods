# Configuring Integration Server for JMS Messaging

## Overview

To configure Integration Server (IS) for JMS messaging:

1. Create one or more **JNDI Provider Aliases**.
2. Create one or more **JMS Connection Aliases**.
3. Optionally configure JMS-related server parameters.

---

## Working with JNDI Providers

### What is JNDI?

JNDI (Java Naming and Directory Interface) is a Java API that provides naming and directory functionality.

A JMS provider stores administered objects such as:
- Connection Factories
- Queues
- Topics

Integration Server uses a JNDI Provider Alias to locate these objects.

### Predefined JNDI Provider Alias

**DEFAULT_IS_JNDI_PROVIDER**

Default URL:

```text
nsp://localhost:9000
```

Used to connect to a local Universal Messaging realm.

---

## Creating a JNDI Provider Alias

Navigation:

```text
Messaging > JNDI Settings > Create JNDI Provider Alias
```

### Important Fields

| Field | Description |
|---------|-------------|
| JNDI Alias Name | Alias name |
| Initial Context Factory | JNDI provider class |
| Provider URL | JNDI server URL |
| Security Principal | Username |
| Security Credentials | Password |
| Other Properties | Provider-specific settings |
| Use SSL | Enable SSL |

### Universal Messaging URL Example

```text
nsp://127.0.0.1:9000
```

---

## SSL Configuration

SSL can be configured directly in the JNDI Provider Alias.

### Truststore Alias

Contains CA certificates.

### Keystore Alias

Contains client certificates and private keys.

Common Universal Messaging SSL properties:

```text
nirvana.ssl.keystore.path
nirvana.ssl.keystore.pass
nirvana.ssl.keystore.cert
nirvana.ssl.truststore.path
nirvana.ssl.truststore.pass
nirvana.ssl.protocol
```

---

## JMS Connection Alias

A JMS Connection Alias contains all information required to establish a connection between Integration Server and the JMS provider.

### Predefined Alias

- DEFAULT_IS_JMS_CONNECTION
- PE_NON_TRANSACTIONAL_ALIAS

### Navigation

```text
Messaging > JMS Settings > Create JMS Connection Alias
```

### Transaction Types

| Type | Description |
|--------|-------------|
| NO_TRANSACTION | Non-transacted session |
| LOCAL_TRANSACTION | Local transaction |
| XA_TRANSACTION | Distributed transaction |

---

## Client Side Queue (CSQ)

Used when the JMS provider is unavailable.

```text
Message -> CSQ -> JMS Provider
```

Settings:

```text
-1 = Unlimited
 0 = Disabled
```

---

## Producer Caching

Allows session and producer reuse to improve performance.

Modes:

- DISABLED
- ENABLED PER DESTINATION

---

## Automatic Retry

Configurable through:

- Max Retry Count
- Retry Interval

Used for transient JMS send failures.

---

## Connection Monitoring

Relevant server properties:

```text
watt.server.jms.connection.monitorPeriod
watt.server.jms.connection.retryPeriod
watt.server.jms.connection.pingPeriod
```

---

## Note

This file contains a cleaned and structured version of the JMS/JNDI content. The original content shared in chat exceeded normal document size limits and should be split into multiple sections for a complete documentation conversion.
