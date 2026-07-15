# TWSC.i3xConnector

---

# Disclaimer
By downloading this software, the user acknowledges that it is unsupported, not reviewed for security purposes, and that the user assumes all risk for running it.

Users accept all risk whatsoever regarding the security of the code they download.

This Software is not officially supported by PTC

PTC will not accept technical support cases logged related to to this Software

ThingWorx R&D will not be responsible for any code on the PTC GitHub repository.

PTC has not performed its full breadth of testing and quality assurance on this software.

This Software has not been released for general distribution or sale.

This Software may not be compatible with any existing or future commercial release versions of PTC software.

PTC is under no obligation to and may never commercially release this software or other software containing functionality contained in this Software.

PTC is not responsible for any maintenance for this software.

---

# Overview

ThingWorx integration template for connecting to i3x-compliant REST APIs.

This project contains:

- `TWSC.i3xConnector_TT` — Base Thing Template used for i3x integrations
- `TWSC.i3xConnector.i3xdev` — Example Thing configured against a development i3x endpoint

The goal of this project is to provide a reusable ThingWorx abstraction layer over standard i3x REST APIs, allowing ThingWorx applications to interact with external systems using strongly-typed services rather than manually constructing HTTP requests.

The connector encapsulates:

- Endpoint configuration
- Authentication handling
- URL generation
- Request execution
- Error handling
- Response normalization
- Service wrappers for standard i3x APIs

Application developers create a Thing from the template, configure the endpoint, and then consume the provided services.

---

# Architecture

```text
ThingWorx Application
        |
        v
TWSC.i3xConnector_TT
        |
        v
i3x REST API
        |
        +--- /info
        +--- /namespaces
        +--- /objecttypes
        +--- /relationshiptypes
        +--- /objects
        +--- /objects/value
        +--- /objects/history
        +--- /subscriptions
```

---

# Included Entities

## Thing Template

```text
TWSC.i3xConnector_TT
```

Base template containing:

### Configuration Tables

```text
EndpointConfiguration
```

Stores endpoint and authentication configuration.

### Core Services

```text
BuildUrl
ExecuteI3XRequest
TestEndpointConnection
```

### i3x Services

Examples include:

```text
getInfo
getNamespaces

getObjectTypes
queryObjectTypesById

getRelationshipTypes
queryRelationshipTypesById

getObjects
listObjectsById

queryLastKnownValues
queryHistoricalValues

updateCurrentValues
updateHistoricalValues

createSubscription
deleteSubscriptions
registerMonitoredItems
unregisterMonitoredItems
```

### Example Utility Services

```text
GetElementIdFromThingName
GetOEEElementIdForThing
UpdateOEEPropertiesForThing
```

These services demonstrate how application-specific business logic can be built using the generic i3x services exposed by the connector.

---

# How the Template Was Created

The template was designed using a reusable service-oriented architecture:

- Configuration is stored in Configuration Tables.
- Authentication is centralized.
- URL generation is centralized.
- Request execution is centralized.
- Individual i3x endpoint services act as thin wrappers around common execution logic.

The template internally uses:

```text
BuildUrl
```

to construct endpoint URLs and:

```text
ExecuteI3XRequest
```

to perform HTTP requests through ThingWorx `ContentLoaderFunctions`.

This design keeps endpoint-specific code minimal and simplifies maintenance.

---

# Authentication Support

The connector supports the following authentication types.

---

## None

Used for unsecured or test environments.

```text
authType = none
```

No Authorization header is sent.

---

## Bearer Token

Used when the target endpoint requires an OAuth bearer token or static bearer token.

```text
authType = bearerToken
```

Configuration:

```text
bearerToken = eyJhbGciOi...
```

Generated header:

```http
Authorization: Bearer eyJhbGciOi...
```

---

## Basic Authentication

```text
authType = basic
```

Configuration:

```text
username = apiuser
password = secret
```

Generated header:

```http
Authorization: Basic base64(username:password)
```

---

# Creating a New Connector Thing

Create a new Thing using:

```text
TWSC.i3xConnector_TT
```

Example:

```text
TWSC.i3xConnector.MyEndpoint
```

---

# Configuring EndpointConfiguration

Populate the `EndpointConfiguration` table.

Example:

| Field | Value |
|---------|---------|
| endpointName | myendpoint |
| description | My i3x Endpoint |
| baseUrl | https://api.example.com/v1 |
| authType | bearerToken |
| bearerToken | eyJhbGc... |
| timeoutSeconds | 30 |
| enabled | true |

---

# Included Development Example

The included example Thing:

```text
TWSC.i3xConnector.i3xdev
```

is configured as:

| Field | Value |
|---------|---------|
| endpointName | i3xdev |
| description | Test i3x development endpoint |
| baseUrl | https://api.i3x.dev/v1 |
| authType | none |
| timeoutSeconds | 30 |
| enabled | true |

---

# Validating Connectivity

Execute:

```text
TestEndpointConnection
```

Expected result:

```json
{
  "success": true
}
```

If SSL validation errors occur:

```text
PKIX path building failed
```

the server certificate chain must be imported into the Java truststore used by the ThingWorx platform.

---

# Example Service Calls

## Retrieve Server Information

```javascript
Things["TWSC.i3xConnector.i3xdev"].getInfo();
```

---

## Query Namespaces

```javascript
Things["TWSC.i3xConnector.i3xdev"].getNamespaces();
```

---

## Retrieve Objects

```javascript
Things["TWSC.i3xConnector.i3xdev"].getObjects({
    root: true,
    includeMetadata: true
});
```

---

## Retrieve Specific Objects

```javascript
Things["TWSC.i3xConnector.i3xdev"].listObjectsById({
    body: {
        elementIds: ["pump-101"],
        includeMetadata: true
    }
});
```

Example request body:

```json
{
  "elementIds": [
    "pump-101"
  ],
  "includeMetadata": true
}
```

---

## Query Current Values

```javascript
Things["TWSC.i3xConnector.i3xdev"].queryLastKnownValues({
    body: {
        elementIds: ["pump-101"],
        includeMetadata: false
    }
});
```

Example request body:

```json
{
  "elementIds": [
    "pump-101"
  ],
  "includeMetadata": false
}
```

---

## Query Historical Values

```javascript
Things["TWSC.i3xConnector.i3xdev"].queryHistoricalValues({
    body: {
        elementIds: ["pump-101"],
        startTime: "2026-01-01T00:00:00Z",
        endTime: "2026-01-02T00:00:00Z"
    }
});
```

---

# Example Integration Pattern

A common integration pattern is:

```text
Thing Name
    |
    v
GetElementIdFromThingName
    |
    v
listObjectsById
    |
    v
relationships.hasOEE
    |
    v
oee-pump-101
    |
    v
queryLastKnownValues
    |
    v
UpdateOEEPropertiesForThing
```

`GetOEEElementIdForThing` retrieves metadata relationships from the object and identifies the related OEE element.

`UpdateOEEPropertiesForThing` then retrieves the latest OEE record and updates ThingWorx properties such as:

```text
AvailabilityOEE
Performance
Quality
```

---

# Example Response Structures

## Object Metadata Response

```json
{
  "results": [
    {
      "elementId": "pump-101",
      "result": {
        "metadata": {
          "relationships": {
            "hasOEE": [
              "oee-pump-101"
            ]
          }
        }
      }
    }
  ]
}
```

---

## Current Value Response

```json
{
  "results": [
    {
      "elementId": "oee-pump-101",
      "result": {
        "value": {
          "availability": 0.95,
          "performance": 0.91,
          "quality": 0.98,
          "oee": 0.85
        }
      }
    }
  ]
}
```

---

# Logging and Troubleshooting

Most services include detailed debug logging.

Enable DEBUG logging and review:

```text
Monitoring
  → Script Log
```

Common log entries include:

```text
Thing Name

Resolved Element ID

Request Body

Response Payload

Metadata Relationships

Selected OEE Element ID

Availability Value

Performance Value

Quality Value

Property Updates
```

These logs are invaluable when troubleshooting endpoint connectivity or response parsing issues.

---

# Extending the Connector

Additional i3x endpoints can be added by:

1. Creating a new service.
2. Building the request payload.
3. Calling:

```text
ExecuteI3XRequest
```

4. Returning the normalized response.

Recommended practices:

- Keep endpoint-specific logic in individual services.
- Reuse authentication and request execution services.
- Avoid direct `ContentLoaderFunctions` calls outside the template.

---

# Version Control

The entities in this repository are maintained as ThingWorx Source Control exports.

Recommended process:

1. Modify entities in a development environment.
2. Export entities using Source Control format.
3. Commit exported XML to source control.
4. Promote through environments using exports rather than manual Composer changes.

---

# License

## PTC Proprietary Freeware License

I accept the PTC End User License Agreement (https://www.ptc.com/en/documents/legal-agreements/on-premise-license-agreements) and agree that any software downloaded/utilized will be in compliance with that Agreement.  However, despite anything to the contrary in the License Agreement, I agree as follows:

I acknowledge that I am not entitled to support assistance with respect to the software, and PTC will have no obligation to maintain the software or provide bug fixes or security patches or new releases.

The software is provided “As Is” and with no warranty, indemnitees or guarantees whatsoever, and PTC will have no liability whatsoever with respect to the software, including with respect to any intellectual property infringement claims or security incidents or data loss.
