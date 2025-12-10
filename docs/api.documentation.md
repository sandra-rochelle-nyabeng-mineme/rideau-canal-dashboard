Voici ton fichier **`api-documentation.md`** complet, propre, structuré, et aligné avec **ton backend Node.js / Express + Cosmos DB** EXACTEMENT comme tu l’as implémenté.

Tu peux **copier-coller directement dans ton repo** 🎯

---

# 📄 API Documentation – Rideau Canal Dashboard

This document describes the REST API endpoints used by the **Rideau Canal Monitoring Dashboard**.
All endpoints are implemented in **Node.js + Express**, and data is retrieved from **Azure Cosmos DB (SQL API)**.

---

## 🌐 Base URL

For local development:

```
http://localhost:3000
```

For production (Azure App Service):

```
https://<your-app-name>.azurewebsites.net
```

All API endpoints are prefixed with:

```
/api
```

---

# 📘 Endpoints Overview

| Method | Endpoint                  | Description                                                    |
| ------ | ------------------------- | -------------------------------------------------------------- |
| GET    | `/api/health`             | Health check endpoint                                          |
| GET    | `/api/latest`             | Returns the **most recent** telemetry aggregation per location |
| GET    | `/api/history?minutes=XX` | Returns **historical data** for the selected time range        |

---

# 🩺 1. Health Check – `GET /api/health`

### **Description**

Used to verify that the API is running and responsive.

### **Response Example**

```json
{
  "status": "ok",
  "timestamp": "2025-12-03T06:22:11.331Z"
}
```

---

# 📊 2. Latest Aggregated Values – `GET /api/latest`

### **Description**

Returns the **most recent** aggregated metrics for each skating location:

* Dow’s Lake
* Fifth Avenue
* NAC

Data is grouped in **5-minute tumbling windows** by Stream Analytics and stored in Cosmos DB.

### Example Response

```json
{
  "success": true,
  "timestamp": "2025-12-03T05:34:28.509Z",
  "data": [
    {
      "location": "Fifth Avenue",
      "windowEnd": "2025-12-03T05:10:00Z",
      "avgIceThickness": 27.74,
      "minIceThickness": 20.36,
      "maxIceThickness": 37.21,
      "avgSurfaceTemperature": -3.51,
      "minSurfaceTemperature": -8.49,
      "maxSurfaceTemperature": 0.58,
      "maxSnowAccumulation": 12.73,
      "avgExternalTemperature": -5.55,
      "readingCount": 14,
      "safetyStatus": "Caution"
    }
  ]
}
```

---

# 🕒 3. Historical Data – `GET /api/history?minutes=60`

### **Description**

Returns all aggregated telemetry for the past **X minutes**.

### **Query Parameters**

| Parameter | Required | Description                            |
| --------- | -------- | -------------------------------------- |
| `minutes` | Yes      | Number of minutes of history to return |

### Example Request

```
GET /api/history?minutes=60
```

### Example Response

```json
{
  "success": true,
  "range": "60 minutes",
  "data": [
    {
      "location": "Dows Lake",
      "windowEnd": "2025-12-03T04:30:00Z",
      "avgIceThickness": 31.57,
      "avgSurfaceTemperature": -2.51,
      "maxSnowAccumulation": 13.69,
      "safetyStatus": "Safe"
    },
    { "... another document ..." }
  ]
}
```

---

# 🗂️ Cosmos DB Schema

Documents stored in Cosmos DB have the following structure:

```json
{
  "id": "uuid",
  "location": "Dows Lake",
  "windowEnd": "2025-12-03T05:10:00Z",
  "avgIceThickness": 31.57,
  "minIceThickness": 22.91,
  "maxIceThickness": 39.6,
  "avgSurfaceTemperature": -2.51,
  "minSurfaceTemperature": -5.84,
  "maxSurfaceTemperature": 1.47,
  "maxSnowAccumulation": 13.69,
  "avgExternalTemperature": -7.38,
  "readingCount": 14,
  "safetyStatus": "Safe"
}
```

### Partition Key:

```
/location
```

---

# 🛑 Error Handling

### Example Error Response

```json
{
  "success": false,
  "error": "Missing 'minutes' query parameter"
}
```

### Possible Errors

| Error                     | Meaning                                      |
| ------------------------- | -------------------------------------------- |
| 400 – Bad Request         | Missing or invalid query parameter           |
| 500 – Server Error        | Internal issue (Cosmos DB query, connection) |
| 503 – Service Unavailable | Cosmos DB unreachable                        |

---

# 🔐 Authentication

Currently, **no authentication** is required because this is a **school project** running in a controlled environment.

If needed, you can add:

* Azure App Service Authentication
* API Key headers
* Entra ID (Azure AD) JWT validation

---

# 🧪 Testing the API

## Using Browser

* `https://<yourapp>.azurewebsites.net/api/latest`
* `https://<yourapp>.azurewebsites.net/api/history?minutes=30`

## Using curl

```sh
curl https://<yourapp>.azurewebsites.net/api/latest
```

## Using Postman

Import the following endpoints manually.

---

# 🎯 Summary

This API provides:

* Live aggregated ice condition data
* Historical trend data
* Health check endpoint
* Standardized JSON responses
* Integration with Cosmos DB and the web dashboard

---

If you want, je peux aussi créer :

✔ `architecture.md`
✔ `stream-analytics.md`
✔ `cosmos-schema.md`
✔ `deployment-guide.md`

Just ask ❤️
