

# 🌐 Rideau Canal Skateway – Real-time Monitoring Dashboard  
CST8916 – Remote Data and Real-time Applications  
Web Dashboard Repository
- Sandra Rochelle Nyabeng Mineme



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



### 🛠 Technologies Used

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
- (Optional but recommended) A running instance of the sensor simulator

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

ou

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

## 5.API Endpoints (Backend) 
### GET /api/latest

Returns the latest aggregated record for each location.

**Example response:** (Donner un vrai resultat ici)
[
  {
   "id": "Dows Lake-2025-12-02T15:35:00Z",
    "location": "Dows Lake",
    "windowStart": "2025-12-02T15:30:00Z",
    "windowEnd": "2025-12-02T15:35:00Z",
    "avgIceThickness": 31.2,
    "minIceThickness": 29.5,
    "maxIceThickness": 33.0,
    "avgSurfaceTemperature": -3.5,
    "minSurfaceTemperature": -5.0,
    "maxSurfaceTemperature": -1.0,
    "maxSnowAccumulation": 5.2,
    "avgExternalTemperature": -9.1,
    "readingCount": 30,
    "safetyStatus": "Safe"
  }
]

The frontend uses this endpoint to:

- Populate the 3 location cards
- Display color-coded safety badges

## GET /api/history?minutes=60

Returns all records newer than the specified number of minutes, across all locations.

**Example:**
```
GET /api/history?minutes=60
```

**Response:**

[
  {
   "id": "Dows Lake-2025-12-02T15:10:00Z",
    "location": "Dows Lake",
    "windowEnd": "2025-12-02T15:10:00Z",
    "avgIceThickness": 30.5,
    "avgSurfaceTemperature": -4.1,
    "safetyStatus": "Safe"
  }
]
The frontend uses this endpoint to:

- Build time-series arrays of values (per location)
- Plot them using Chart.js (e.g., line charts)

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