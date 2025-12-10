# Rideau Canal Skateway – Real-time Monitoring Dashboard  
CST8916 – Remote Data and Real-time Applications  
Web Dashboard Repository
- Sandra Rochelle Nyabeng Mineme

---

## 1. Overview
### Dashboard features

This repository contains the **web dashboard application** for the Rideau Canal Skateway Real-time Monitoring System.

The dashboard allows users to:

- View **real-time ice conditions** for:
  - Dow’s Lake
  - Fifth Avenue
  - NAC
- Monitor **safety status** per location:
  - Safe
  - Caution
  - Unsafe
- Visualize **historical trends** (last hour) using charts
- See **automatic updates every 30 seconds**

The application is built as:

- A **Node.js / Express** backend that:
  - Connects to **Azure Cosmos DB**
  - Exposes REST API endpoints for the latest data and history
- A **frontend** using:
  - HTML / CSS / JavaScript
  - Chart.js for data visualization
- Hosted on **Azure App Service**



###  Technologies Used

### Backend
- **Node.js 18+**
- **Express.js**
- **@azure/cosmos** (Azure Cosmos DB SDK)
- **dotenv** (environment variables)
- **cors** (Cross-Origin Resource Sharing, if needed)

### Frontend
- **HTML5**
- **CSS3**
- **Vanilla JavaScript**
- **Chart.js** (line charts / bar charts)

### Hosting
- **Azure App Service (Linux, Node runtime)**



## 2.Prerequisites

You need the following before running the dashboard:

- Node.js 18+ and npm installed locally
- A running **Azure Cosmos DB for NoSQL** instance:
  - Database: `RideauCanalDB`
  - Container: `SensorAggregations`
  - Partition key: `/location`
- A working Stream Analytics job writing aggregated data into Cosmos DB
- A running instance of the sensor simulator

<br>

## 3.Installation
Clone the repository:

```
git clone https://github.com/sandra-rochelle-nyabeng-mineme/rideau-canal-dashboard.git
cd rideau-canal-dashboard
```

**Install dependencies:**

```
npm install
```
<br>

## 4.Configuration 
Create a .env file following .env.example:
```
PORT=3000

COSMOS_ENDPOINT=<your_cosmos_endpoint>
COSMOS_KEY=<your_cosmos_primary_key>
COSMOS_DB_NAME=RideauCanalDB
COSMOS_CONTAINER_NAME=SensorAggregations
```

**Running Locally**

Start the backend server and dashboard:
```
npm start
```

or

```
node server.js
```

**Then visit:**

http://localhost:3000


The dashboard will:

- Fetch /api/latest every 30 seconds
- Fetch /api/history?minutes=60
- Update cards and graphs dynamically

You should see:

- Three cards (Dow’s Lake, Fifth Avenue, NAC)
- Safety status for each location
- Last-hour chart(s) once enough data exists
- Auto-refresh happening every 30 second

<br>

## 5.API Endpoints (Backend)

The Rideau Canal Dashboard exposes several REST API endpoints used by the frontend to retrieve real-time and historical ice-monitoring data processed by Azure Stream Analytics and stored in Cosmos DB.

### 1. Health Check
**GET /api/health**

Verifies that the backend is running and that the Cosmos DB configuration is loaded.

Example Response:

```{
  "status": "healthy",
  "timestamp": "2025-12-03T06:22:11.331Z",
  "cosmosdb": {
    "endpoint": "configured",
    "database": "RideauCanalDB",
    "container": "SensorAggregations"
  }
}
```
### 2. Latest Aggregated Values per Location
**GET /api/latest**

Returns the most recent aggregated record for each skating location:

- Dows Lake
- Fifth Avenue
- NAC

The backend fetches all documents for each location, sorts them by windowEnd, and returns the most recent.

Example Response:
```
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
### 3. Historical Data for a Specific Location
**GET /api/history/:location?limit=XX**

Returns the latest N records for a given location, where limit corresponds to the number of 5-minute aggregation windows.

limit=12 → last hour
limit=24 → last two hours

Path Parameter
- Name	Description
- location	One of: "Dows Lake", "Fifth Avenue", "NAC"
- Query Parameter
- Name	Required	Description
- limit	No	Number of records (default: 12)

Example Request:
`GET /api/history/Dows%20Lake?limit=12`

Example Response: 
```{
  "success": true,
  "location": "Dows Lake",
  "data": [
    {
      "windowEnd": "2025-12-03T04:30:00Z",
      "avgIceThickness": 31.57,
      "avgSurfaceTemperature": -2.51,
      "maxSnowAccumulation": 13.69,
      "safetyStatus": "Safe"
    },
    { "... more documents ..." }
  ]
}
```
### 4. System Status Across All Locations
**GET /api/status**

Returns:

-  individual latest status per location
- computed overall status according to rules:
  - if any location is Unsafe → overall Unsafe
  - else if any location is Caution → overall Caution
  - else → overall Safe

Example Response:
```
{
  "success": true,
  "overallStatus": "Caution",
  "locations": [
    {
      "location": "Dows Lake",
      "safetyStatus": "Safe",
      "windowEnd": "2025-12-03T05:10:00Z"
    },
    {
      "location": "Fifth Avenue",
      "safetyStatus": "Caution",
      "windowEnd": "2025-12-03T05:10:00Z"
    },
    {
      "location": "NAC",
      "safetyStatus": "Safe",
      "windowEnd": "2025-12-03T05:10:00Z"
    }
  ]
}
```

### 5. Full Data Dump (Debug Only)
**GET /api/all**

Returns every document in the Cosmos DB container, sorted by windowEnd descending.

Useful for debugging but not used by the dashboard.

Example Response:

```
{
  "success": true,
  "count": 178,
  "data": [
    {
      "location": "NAC",
      "windowEnd": "2025-12-03T05:10:00Z",
      "avgIceThickness": 30.03,
      "safetyStatus": "Safe",
      "id": "d4664662-1110-4e44-8869-c286ed43627e"
    }
  ]
}
```

### Summary

This backend API enables the dashboard to:

- Display real-time readings for all locations
- Render historical charts
- Compute overall safety status
- Monitor backend health

All endpoints are lightweight, return JSON, and are optimized for Cosmos DB fast reads using the /location partition key.
<br>

## 6.Deployment to Azure App Service
### Step 1 — Create App Service
1. Create an App Service Plan (Linux, Node)
2. Create a Web App:
- Runtime: Node 18
- Publish type: Code
- OS: Linux

### Step 2 — Configure Application Settings

In Azure Portal → Settings → Environment variables → App settings:

**Add:**
```
COSMOS_ENDPOINT
COSMOS_KEY
COSMOS_DB_NAME
COSMOS_CONTAINER_NAME
```
Add the same values as in .env.

### Step 3 — Deploy Code

Options:

- GitHub Actions (Recommended and what i choose):
  - Link the GitHub repo to the Web App in Deployment Center
  - Select branch main
- ZIP Deploy
  - Zip the project (without node_modules)
  - Use Deploy Center 
- VS Code Deployment : Use “Azure App Service” extension to deploy directly

### Step 4 — Restart and Test
Use “Azure App Service” extension to deploy directly
Go to:
```
https://rideau-canal-dashboard-web-fdfvg0a8anbcagbw.canadacentral-01.azurewebsites.net/
or 
https://<your-app-service-name>.azurewebsites.net
```

Verify:

- Cards display data
- Charts render
- Auto-refresh is working while Stream Analytics and simulator are running
<br>

## 7.Dashboard Features

- Real-time data refresh every 30 seconds
- 3 location summary cards
- Real-time summaries 
 - Latest ice thickness
 - Surface temperature
 - Snow accumulation
 - External temperature
 - Safety status per location
- Safety status badges:

  - 🟢 Safe : Ice ≥ 30 cm and Surface Temp ≤ -2 °C
  - 🟡 Caution : Ice ≥ 25 cm and Surface Temp ≤ 0 °C
  - 🔴 Unsafe : All other conditions
- Historical charts (Chart.js):
 - Plots average ice thickness and/or surface temperature over time
 - Per location or combined
- Backend API integrated with Cosmos DB
<br>

## 8.Troubleshooting
### 500 API Errors
- Check App Service Logs.
- Check if Cosmos DB keys and variables are correct
- Verify container name: SensorAggregations
- Ensure Stream Analytics has already written data

### CORS Errors

Enable CORS in Express:
```
app.use(cors());
```

Configure allowed origins in Azure or in the CORS options.

### Charts not updating, No data

- Not enough historical data (wait 5+ minutes)
- Simulator not running
- Stream Analytics job not running

### Dashboard shows "No Data"

- Cosmos DB may not contain data yet
- Ensure partition key = /location
- Ensure Stream Analytics output uses correct key

### Frontend not served on Azure

Ensure in server.js you serve static files from public/:
```
app.use(express.static("public"));
```

Make sure index.html, styles.css, and app.js are inside public/