# Production Forecasting

## General Information and Purpose
- **Service Name/Title:** `production_forecasting`
- **Description and purpose:** Accurately forecast energy production from residential prosumers to assess flexibility capabilities and support market participation decisions.
- **Owner/Contact Information:** ICCS

---

## Functional Requirements
1. Collect and process real-time data from photovoltaic (PV) systems and smart meters.  
2. Access historical production data provided by energy suppliers.  
3. Perform data pre-processing, including cleaning and normalization.  
4. Generate accurate short-term and long-term production forecasts.  
5. Support the integration of production forecasts into optimization algorithms for market bidding.  
6. Provide system operators with forecast data to identify grid issues.  

---

## Non-Functional Requirements
- Forecasts should be available in real-time or near real-time (within a few seconds).  
- Accuracy of production forecasts should be maintained at a high level.  
- System uptime: minimum 99.9%.  
- Access restricted to authorized users.  
- Periodic backups with automated recovery.  

---

## Service Interfaces

### Endpoint 1 — Train Production Forecasting Model
- **URL:** `/forecast/production/train`  
- **Method:** `POST`  
- **Description:** Triggers training or retraining of the model using historical data and relevant features (solar/wind generation, weather patterns, IoT sensor data).  

**Headers**  
- `Content-Type: application/json`  
- `Authorization: Bearer <token>`  

**Request Example**
```json
{
  "start_time": "2023-01-01T00:00:00Z",
  "end_time": "2024-01-01T00:00:00Z",
  "granularity": "hourly",
  "customer_id": "CUST456"
}
```

**Response Example**
```json
{
  "message": "Model training initiated successfully",
  "model_id": "M789",
  "status": "queued",
  "timestamp": "2025-06-16T10:15:00Z"
}
```

**Error Handling**
```json
400 Bad Request: { "error": "Invalid input parameters", "details": "Missing start_time or invalid granularity" }
401 Unauthorized: { "error": "Unauthorized", "message": "Invalid token" }
500 Internal Server Error: { "error": "Training failed", "message": "Model could not be trained due to internal error" }
```

---

### Endpoint 2 — Get Production Forecast
- **URL:** `/forecast/production`  
- **Method:** `GET`  
- **Description:** Provides a forecast of energy production for a specified time range.  

**Headers**  
- `Content-Type: application/json`  
- `Authorization: Bearer <token>`  

**Request Example**
```json
{
  "start_time": "2024-10-01T00:00:00Z",
  "end_time": "2024-10-01T12:00:00Z",
  "granularity": "hourly",
  "model_id": "M456",
  "customer_id": "CUST123"
}
```

**Response Example**
```json
{
  "status": "200 OK",
  "model_id": "M456",
  "data": [
    {"timestamp": "2024-10-01T00:00:00Z", "production": 300.5},
    {"timestamp": "2024-10-01T01:00:00Z", "production": 290.7}
  ]
}
```

**Error Handling**
```json
400 Bad Request: { "error": "Invalid data format" }
401 Unauthorized: { "error": "Unauthorized", "message": "Invalid token" }
```

---

## UI Mock-ups
The user interface (embedded in the Aggregator Platform) should allow users to:  
- Input start and end time for forecasts.  
- Select granularity.  
- View forecasts as interactive charts/tables.  
- Download forecasts as CSV.  

UI elements:  
- Date-Time Picker  
- Granularity Dropdown  
- Display Panel  
- Error Messages  

---

## Data Model

### Entities and Relationships
- **Users** manage **Houses**.  
- **Houses** contain **Devices** (PV, smart meters).  
- **Users** can join **Energy Communities** managed by **Aggregators**.  
- **PV Parks** linked to communities, private PVs linked to houses.  

### Database Schema
- **Real-Time Energy Measurements:** Detailed high-frequency PV data.  
- **Aggregated Energy Usage:** Summarized hourly/daily/quarterly production.  
- **Metadata Schema:** Users, houses, devices, communities, PVs.  

---

## Integration and Dependencies

### Integration
- **LFM API Interface:** Forecast outputs support bid formation and pricing.  
- **Internal Data Sources:** Historical PV data, IoT telemetry, weather.  
- **Edge/Cloud Integration:** Smart meters, PV monitors, cloud storage.  
- **App Store/Data Space Integration (Planned):** Shared models and compliance.  

### Dependencies
- **External:** Weather APIs, IoT connectivity.  
- **System:** Forecasting modules, system DB, UIs.  
- **Third-Party:** None currently.  

### Roles
- **Aggregator:** Uses forecasts to bid flexibility.  
- **Market Operator (HENEX):** Relies on forecast-informed bids.  
- **Supplier:** Monitors generation trends.  
- **Data Provider:** Supplies historical and real-time data.  
- **ESCO:** Recommends efficiency measures.  

### Domains
- Forecasting Zone  
- Metering Grid Area  
- Scheduling Area  

### Key Elements
- Metering Points / Smart Meters  
- Forecasting Models  
- Weather Data  
- Metered Data Roles  

---

## Security and Privacy
- **Data Sensitivity:** Customer-level forecasts, model metadata, weather data.  
- **Encryption:** TLS 1.2+ in transit, AES-256 at rest, encrypted backups.  
- **RBAC:** Operator, Administrator, Analyst roles.  
- **Audit Logging:** Forecast requests, training runs, data access.  
- **Anomaly Detection:** Detects unusual requests, injections, unauthorized access.  

---
- Dataset: Dingle community, Ireland (StoreNet project).  
- 20 residential buildings, 10 with PVs (2–2.2 kWp).  
- High-resolution weather data (1-min, aggregated to 15-min).  

### Functionality
- Data ingestion (PV, weather, time features).  
- Feature engineering (lags, rolling stats, time/weather).  
- Normalization (MinMaxScaler, MinIO persistence).  
- Model: Stacked Bi-LSTM for recursive short-term forecasting.  
- Model persistence/versioning: MinIO with models, scalers, metrics.  

### Results
- Accuracy: R² = 0.89, MAE = 0.0593 kWh.  
- Validated recursive Bi-LSTM approach for 15-min forecasts.  
- System is demo-ready and suitable for integration.    
