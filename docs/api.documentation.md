# API Documentation – Rideau Canal Monitoring Dashboard

This document describes the REST API used by the **Rideau Canal Real‑Time Monitoring Dashboard**.  
The backend is implemented in **Node.js + Express**, and all telemetry data is retrieved from **Azure Cosmos DB (SQL API)**.

---

## 1. Base URLs

### Local development

```text
http://localhost:3000
```

### Production (Azure App Service)

```text
https://<your-app-name>.azurewebsites.net
```

Unless otherwise noted, all API endpoints are available under this base URL.

---

## 2. Endpoints Overview

| Method | Endpoint                | Description                                                                                 |
|--------|-------------------------|---------------------------------------------------------------------------------------------|
| GET    | `/health`               | Returns API health information and checks Cosmos DB configuration                           |
| GET    | `/api/latest`           | Returns the most recent aggregated reading for each location                                |
| GET    | `/api/history/:location`| Returns historical aggregated readings for a specific location (with optional `limit`)      |
| GET    | `/api/status`           | Returns the latest safety status per location and the overall canal status                  |
| GET    | `/api/all`              | Returns **all** documents from the Cosmos DB container (debug / troubleshooting only)       |

---

## 3. Health Check – `GET /health`

### Description

Simple endpoint used to verify that the backend is running and that Cosmos DB configuration values are present.

### Example request

```http
GET /health HTTP/1.1
Host: localhost:3000
```

### Example response

```json
{
  "status": "healthy",
  "timestamp": "2025-12-03T08:27:39.791Z",
  "cosmosdb": {
    "endpoint": "configured",
    "database": "RideauCanalDB",
    "container": "IceReadings"
  }
}
```

If the Cosmos DB configuration is missing, the `endpoint` field may be `"missing"` or the process may throw errors in the logs.

---

## 4. Latest Readings – `GET /api/latest`

### Description

Returns the **most recent** aggregated reading for each configured location.  
Locations are currently hard‑coded in the backend as:

- `Dows Lake`
- `Fifth Avenue`
- `NAC`

For each location, the server:

1. Queries Cosmos DB for all documents with that `location`.
2. Sorts results by `windowEnd` in descending order.
3. Returns the first (latest) document.

### Example request

```http
GET /api/latest HTTP/1.1
Host: localhost:3000
```

### Example response

```json
{
  "success": true,
  "timestamp": "2025-12-03T05:34:28.509Z",
  "data": [
    {
      "id": "Dows Lake-2025-12-03T05:30:00Z",
      "location": "Dows Lake",
      "windowEnd": "2025-12-03T05:30:00Z",
      "avgIceThickness": 30.8,
      "avgSurfaceTemperature": -4.3,
      "maxSnowAccumulation": 11.4,
      "avgExternalTemperature": -6.1,
      "readingCount": 15,
      "safetyStatus": "Safe"
    },
    {
      "id": "Fifth Avenue-2025-12-03T05:30:00Z",
      "location": "Fifth Avenue",
      "windowEnd": "2025-12-03T05:30:00Z",
      "avgIceThickness": 27.2,
      "avgSurfaceTemperature": -2.7,
      "maxSnowAccumulation": 14.0,
      "avgExternalTemperature": -4.9,
      "readingCount": 14,
      "safetyStatus": "Caution"
    }
  ]
}
```

### Error response example

```json
{
  "success": false,
  "error": "Failed to fetch latest data"
}
```

---

## 5. Historical Data – `GET /api/history/:location?limit=XX`

### Description

Returns historical aggregated readings for a **single** location.  
Results are returned in chronological order (oldest first) to make charting easier.

Internally the API:

1. Queries Cosmos DB for all documents where `c.location = @location`.
2. Sorts by `windowEnd` descending.
3. Takes the first `limit` documents (default 12).
4. Reverses the list so responses are returned from oldest to newest.

### URL parameters

| Type | Name       | Required | Description                                        |
|------|------------|----------|----------------------------------------------------|
| Path | `location` | Yes      | Location name (e.g., `Dows Lake`, `Fifth Avenue`) |

### Query parameters

| Name   | Required | Default | Description                                             |
|--------|----------|---------|---------------------------------------------------------|
| `limit`| No       | `12`    | Number of records to return (5‑minute windows per row) |

> Note: Because Stream Analytics uses 5‑minute tumbling windows, `limit = 12` roughly corresponds to one hour of data.

### Example request

```http
GET /api/history/Dows%20Lake?limit=12 HTTP/1.1
Host: localhost:3000
```

### Example response

```json
{
  "success": true,
  "location": "Dows Lake",
  "data": [
    {
      "location": "Dows Lake",
      "windowEnd": "2025-12-03T04:30:00Z",
      "avgIceThickness": 31.5,
      "avgSurfaceTemperature": -5.2,
      "maxSnowAccumulation": 10.9,
      "avgExternalTemperature": -7.0,
      "readingCount": 13,
      "safetyStatus": "Safe"
    },
    {
      "location": "Dows Lake",
      "windowEnd": "2025-12-03T04:35:00Z",
      "avgIceThickness": 31.2,
      "avgSurfaceTemperature": -5.0,
      "maxSnowAccumulation": 11.2,
      "avgExternalTemperature": -6.8,
      "readingCount": 14,
      "safetyStatus": "Safe"
    }
  ]
}
```

### Error response example

```json
{
  "success": false,
  "error": "Failed to fetch historical data"
}
```

---

## 6. System Status – `GET /api/status`

### Description

Provides a high‑level summary of the canal’s safety status.

For each location, the API:

1. Queries Cosmos DB for documents matching the location.
2. Sorts by `windowEnd` descending.
3. Takes the most recent document and extracts its `safetyStatus`.

Then it computes the `overallStatus`:

- If **all** locations are `Safe` → overall = `Safe`
- If **any** location is `Unsafe` → overall = `Unsafe`
- Otherwise → overall = `Caution`

### Example request

```http
GET /api/status HTTP/1.1
Host: localhost:3000
```

### Example response

```json
{
  "success": true,
  "overallStatus": "Caution",
  "locations": [
    {
      "location": "Dows Lake",
      "safetyStatus": "Safe",
      "windowEnd": "2025-12-03T05:30:00Z"
    },
    {
      "location": "Fifth Avenue",
      "safetyStatus": "Caution",
      "windowEnd": "2025-12-03T05:30:00Z"
    },
    {
      "location": "NAC",
      "safetyStatus": "Safe",
      "windowEnd": "2025-12-03T05:30:00Z"
    }
  ]
}
```

### Error response example

```json
{
  "success": false,
  "error": "Failed to fetch system status"
}
```

---

## 7. All Data (Debug) – `GET /api/all`

### Description

Debugging endpoint used to inspect the raw contents of the Cosmos DB container.  
Returns **all documents**, sorted by `windowEnd` in descending order.

This endpoint should not be exposed in production to anonymous users, but is useful for development and validation.

### Example request

```http
GET /api/all HTTP/1.1
Host: localhost:3000
```

### Example response

```json
{
  "success": true,
  "count": 203,
  "data": [
    {
      "id": "Dows Lake-2025-12-03T05:30:00Z",
      "location": "Dows Lake",
      "windowEnd": "2025-12-03T05:30:00Z",
      "avgIceThickness": 30.8,
      "avgSurfaceTemperature": -4.3,
      "maxSnowAccumulation": 11.4,
      "avgExternalTemperature": -6.1,
      "readingCount": 15,
      "safetyStatus": "Safe"
    }
  ]
}
```

---

## 8. Cosmos DB Data Model

### Container

- **Database:** `RideauCanalDB`
- **Container:** `IceReadings`
- **Partition key:** `/location`

### Example document

```json
{
  "id": "Dows Lake-2025-12-03T05:10:00Z",
  "location": "Dows Lake",
  "windowEnd": "2025-12-03T05:10:00Z",
  "avgIceThickness": 31.57,
  "avgSurfaceTemperature": -2.51,
  "maxSnowAccumulation": 13.69,
  "avgExternalTemperature": -7.38,
  "readingCount": 14,
  "safetyStatus": "Safe"
}
```

Fields like `avgIceThickness`, `avgSurfaceTemperature`, and `maxSnowAccumulation` are calculated in Azure Stream Analytics using 5‑minute tumbling windows.

---

## 9. Error Handling

### General pattern

On success, endpoints return:

```json
{
  "success": true,
  "...": "..."
}
```

On failure, endpoints return:

```json
{
  "success": false,
  "error": "Human-readable error message"
}
```

### Typical error cases

| Status code | Scenario                                    |
|------------:|---------------------------------------------|
| 400         | Invalid parameters (if validation is added) |
| 500         | Cosmos DB query failures or unexpected errors |
| 503         | Cosmos DB temporarily unavailable           |

Errors are also logged to the server console for debugging.

---

## 10. Testing the API

### Browser

For quick manual tests in a browser:

- `http://localhost:3000/health`
- `http://localhost:3000/api/latest`
- `http://localhost:3000/api/status`
- `http://localhost:3000/api/history/Dows%20Lake?limit=12`

### curl

```bash
curl http://localhost:3000/api/latest
curl "http://localhost:3000/api/history/Dows%20Lake?limit=6"
curl http://localhost:3000/api/status
```

### Postman / Thunder Client

- Create a collection with:
  - `GET /health`
  - `GET /api/latest`
  - `GET /api/history/Dows%20Lake?limit=12`
  - `GET /api/status`
  - `GET /api/all`
- Configure the correct base URL for local or Azure.

---

## 11. Summary

The Rideau Canal Monitoring Dashboard API exposes a small, focused set of endpoints that:

- Provide **latest** ice and safety conditions per location  
- Return **historical** aggregated data for charting trends  
- Compute an **overall safety status** for the canal  
- Offer **health** and **debug** endpoints for operations and troubleshooting  

These APIs are consumed by the web dashboard frontend to display near real‑time conditions for Dows Lake, Fifth Avenue, and the NAC sections of the Rideau Canal.
