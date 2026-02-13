# 🏗️ Perishable Air Cargo Logistics — Complete Technical Architecture

**Routes:** 🇮🇳 Tamil Nadu / Kerala / Bangalore → 🇦🇪 Sharjah / Abu Dhabi  
**Cargo:** Flowers | Vegetables | Leaves | Food Items  
**Stack:** Node.js | Express | MongoDB | (Integrates with .NET POS/Branch systems)

---

## 📋 Table of Contents

1. [System Overview](#system-overview)
2. [Tech Stack](#tech-stack)
3. [Database Schema](#database-schema)
4. [API Architecture](#api-architecture)
5. [Application Structure](#application-structure)
6. [Roles & Permissions](#roles--permissions)
7. [Master Status Flow](#master-status-flow)
8. [Integration Points](#integration-points)
9. [Future Enhancement Hooks](#future-enhancement-hooks)

---

## 1. System Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    PERISHABLE AIR CARGO LOGISTICS ERP                        │
├─────────────────────────────────────────────────────────────────────────────┤
│  🇮🇳 INDIA                    ✈️ TRANSIT                    🇦🇪 UAE          │
│  ┌──────────┐               ┌──────────┐              ┌──────────┐         │
│  │ PO       │ ──► Packing ──►│ Flight   │──► Arrival ──►│ Warehouse│         │
│  │ Receive  │               │ Dispatch │    Clearance │ Delivery │         │
│  └──────────┘               └──────────┘              └──────────┘         │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Tech Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Backend** | Node.js + Express 4.x | REST API server |
| **Database** | MongoDB + Mongoose | Document store, flexible schema |
| **Auth** | JWT + bcrypt | Stateless auth for APIs |
| **Integration** | Axios + Webhooks | UAE PO receive, flight APIs |
| **Frontend** | React/Vanilla (client/) | Admin & operator dashboards |
| **Mobile** | React Native / Flutter (future) | Driver app |
| **Integration** | REST/Webhook | Connect to existing .NET POS/Branch |

---

## 3. Database Schema

### 3.1 Core Entities

#### **Organization & Locations**

```javascript
// Organization (Company)
{
  _id, name, code, country, timezone,
  branches: [{ _id, name, code, address, city, type: 'shop'|'warehouse'|'packing_center' }]
}

// Location
{
  _id, organizationId, name, code, type: 'shop'|'warehouse'|'packing_center'|'airport',
  address, city, country, airportCode?, coordinates
}
```

#### **Product Catalog**

```javascript
// Product
{
  _id, sku, name, category: 'flower'|'vegetable'|'leaves'|'food',
  unit: 'kg'|'box'|'bunch', perishabilityHours,
  storageTempMin, storageTempMax
}
```

---

### 3.2 Module 1 — PO Receive

```javascript
// PurchaseOrder
{
  _id, poNumber, status: 'created'|'received'|'ready_for_packing'|'packing'|'packed'|...,
  sourceBranchId,      // UAE shop
  destinationLocationId,
  createdAt, receivedAt,
  receivedQuantity, damagedQuantity, shortageQuantity,
  notes, createdBy
}

// POItem
{
  _id, purchaseOrderId, productId, orderedQty,
  receivedQty, damagedQty, shortQty, unit
}
```

---

### 3.3 Module 2 — Packing Management

```javascript
// PackingSession
{
  _id, sessionNumber, status: 'started'|'in_progress'|'completed',
  purchaseOrderIds: [ObjectId],
  startedAt, completedAt, packingCenterId
}

// Box
{
  _id, boxId, sessionId, type: 'carton'|'thermocol'|'plastic_crate',
  size: 'S'|'M'|'L', weightKg, destinationCode: 'SHJ'|'AUH',
  destinationLocationId, status: 'created'|'sealed'|'labeled'|'attached'
}

// BoxItem
{
  _id, boxId, productId, quantity, unit, weightKg
}

// PackingMaterial
{
  _id, type: 'ice_pack'|'cooling_sheet'|'thermocol_liner',
  quantityUsed, sessionId, boxId?
}
```

---

### 3.4 Module 3 — Box Labeling

```javascript
// BoxLabel (denormalized for quick scan)
{
  boxId, destinationCode, locationCode, productType, weightKg, qrCode,
  printedAt
}
```

---

### 3.5 Module 4 — Shipment Scheduling

```javascript
// Shipment
{
  _id, shipmentNumber, awbNumber,
  flightId, scheduledDate, scheduledTime,
  fromAirportId, toAirportId,
  boxIds: [ObjectId], totalWeightKg,
  status: 'scheduled'|'loaded'|'in_flight'|'arrived'|...
}

// Flight
{
  _id, flightNumber, airline, capacityKg,
  fromAirportId, toAirportId,
  scheduledDeparture, scheduledArrival
}
```

---

### 3.6 Module 5 — Flight Dispatch

```javascript
// ShipmentStatusLog
{
  shipmentId, status, timestamp, notes,
  delayReason?, offloadReason?, rebookedTo?
}
```

---

### 3.7 Module 6 — Arrival & Clearance

```javascript
// ClearanceRecord
{
  shipmentId, arrivedAt, clearedAt,
  customsStatus: 'pending'|'cleared'|'held',
  boxScans: [{ boxId, scannedAt, status }]
}
```

---

### 3.8 Module 7 — Warehouse Processing

```javascript
// WarehouseReceipt
{
  _id, shipmentId, warehouseId, receivedAt,
  verifiedBoxIds: [ObjectId], damagedBoxIds: [ObjectId],
  sortedBatches: [{ destinationLocationId, boxIds: [] }]
}

// RedistributionBatch
{
  _id, warehouseId, destinationLocationId,
  boxIds: [ObjectId], createdBy, createdAt
}
```

---

### 3.9 Module 8 — Delivery Management

```javascript
// DeliveryRun
{
  _id, runNumber, driverId, vehicleId,
  routePlan: [{ locationId, order, boxIds }],
  status: 'created'|'pickup'|'en_route'|'delivering'|'completed'
}

// DeliveryStop
{
  deliveryRunId, locationId, status: 'pending'|'completed'|'failed',
  arrivedAt, completedAt, notes
}
```

---

### 3.10 Module 9 — Transit Support

```javascript
// TransitLeg
{
  shipmentId, fromLocationId, toLocationId,
  status: 'pending'|'in_transit'|'arrived',
  departedAt, arrivedAt
}
```

---

### 3.11 Module 10 — Delivery Completion

```javascript
// DeliveryCompletion
{
  deliveryRunId, stopId, receiverName, signatureUrl?, photoUrls: [],
  podDocumentUrl, driverExpenses: [{ type, amount, currency }],
  status: 'delivered'|'completed'
}
```

---

### 3.12 User & Auth

```javascript
// User
{
  _id, email, passwordHash, name, role,
  branchId?, locationIds: [ObjectId],
  isDriver: Boolean
}
```

---

## 4. API Architecture

### Base URL

```
https://api.yourdomain.com/v1
```

### Auth

- **Login:** `POST /auth/login` → JWT
- **Refresh:** `POST /auth/refresh`
- All protected routes: `Authorization: Bearer <token>`

---

### Module 1 — PO

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/pos` | List POs (filter by status, branch) |
| GET | `/pos/:id` | Get PO + items |
| POST | `/pos` | Create PO (UAE branch) |
| POST | `/pos/:id/receive` | Receive PO (India) — captured received/damaged/short |
| PATCH | `/pos/:id` | Update PO |

---

### Module 2 — Packing

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/packing/sessions` | Create session (per PO or group) |
| GET | `/packing/sessions/:id` | Get session + boxes |
| POST | `/packing/sessions/:id/boxes` | Create box |
| PATCH | `/packing/boxes/:id` | Add items, weight, seal |
| POST | `/packing/sessions/:id/materials` | Record packing materials |
| POST | `/packing/sessions/:id/complete` | Mark session complete |

---

### Module 3 — Labeling

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/boxes/:id/label` | Generate label (QR + data) |
| GET | `/boxes/:id/label/pdf` | Get printable label PDF |

---

### Module 4 — Shipment

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/shipments` | Create shipment (flight, date, boxes) |
| GET | `/shipments` | List shipments |
| GET | `/shipments/:id` | Get shipment details |
| POST | `/shipments/split` | Split if over capacity |
| GET | `/flights` | List flights (by route, date) |

---

### Module 5 — Flight Dispatch

| Method | Endpoint | Description |
|--------|----------|-------------|
| PATCH | `/shipments/:id/status` | Update: loaded, in_flight, delayed, offload, rebook |

---

### Module 6 — Arrival & Clearance

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/shipments/:id/arrive` | Mark arrived |
| POST | `/shipments/:id/clear` | Mark customs cleared |
| POST | `/shipments/:id/scan` | Scan box at arrival |

---

### Module 7 — Warehouse

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/warehouse/receive` | Receive shipment at warehouse |
| POST | `/warehouse/verify` | Verify boxes, record damages |
| POST | `/warehouse/batches` | Create redistribution batch |

---

### Module 8 — Delivery

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/delivery/runs` | Create delivery run |
| PATCH | `/delivery/runs/:id` | Update status (pickup, en_route, delivered) |
| GET | `/delivery/runs` | List runs (driver, date) |
| POST | `/delivery/runs/:id/stops` | Record stop status |

---

### Module 9 & 10 — Transit & Completion

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/transit/legs` | Create transit leg |
| PATCH | `/transit/legs/:id` | Update leg status |
| POST | `/delivery/completions` | POD, signature, photos, expenses |

---

### Common

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/products` | Product catalog |
| GET | `/locations` | Branches, warehouses, airports |
| GET | `/tracking/:boxId` | Full tracking history |
| GET | `/tracking/po/:poNumber` | Track by PO |
| GET | `/tracking/awb/:awb` | Track by AWB |

---

## 5. Application Structure

```
perishable-air-cargo-logistics/
├── package.json
├── index.js                 # Entry
├── .env
├── config/
│   ├── db.js
│   ├── env.js
│   └── constants.js
├── models/
│   ├── Organization.js
│   ├── Location.js
│   ├── Product.js
│   ├── PurchaseOrder.js
│   ├── PackingSession.js
│   ├── Box.js
│   ├── Shipment.js
│   ├── Flight.js
│   ├── WarehouseReceipt.js
│   ├── DeliveryRun.js
│   └── User.js
├── routes/
│   ├── auth.js
│   ├── po.js
│   ├── packing.js
│   ├── shipment.js
│   ├── warehouse.js
│   ├── delivery.js
│   └── tracking.js
├── controllers/
│   ├── poController.js
│   ├── packingController.js
│   └── ...
├── middleware/
│   ├── auth.js
│   ├── roleCheck.js
│   └── validate.js
├── services/
│   ├── labelService.js
│   ├── trackingService.js
│   └── webhookService.js
├── docs/
│   └── ARCHITECTURE.md
└── client/                  # React/Vanilla frontend
    ├── src/
    │   ├── pages/
    │   │   ├── po/
    │   │   ├── packing/
    │   │   ├── shipments/
    │   │   ├── warehouse/
    │   │   └── delivery/
    │   └── components/
    └── package.json
```

---

## 6. Roles & Permissions

| Role | Scope | Access |
|------|-------|--------|
| **Super Admin** | All | Full access |
| **India Operations** | Packing centers | PO receive, Packing, Labeling, Shipment create |
| **UAE Operations** | UAE branches | PO create, Arrival, Warehouse, Delivery |
| **Warehouse Manager** | Warehouse(s) | Receive, Verify, Redistribution |
| **Driver** | Mobile | Pickup, En route, Delivered, POD |
| **Shop/Branch Staff** | Own branch | View POs, Track shipments |
| **Flight Dispatcher** | — | Flight status updates, Delays, Offload |

---

## 7. Master Status Flow

```
PO_CREATED
    ↓
PO_RECEIVED
    ↓
PACKING
    ↓
PACKED
    ↓
SHIPMENT_SCHEDULED
    ↓
LOADED → IN_FLIGHT
    ↓
ARRIVED
    ↓
CLEARED
    ↓
WAREHOUSE_RECEIVED (if warehouse destination)
    ↓
OUT_FOR_DELIVERY
    ↓
DELIVERED
    ↓
COMPLETED
```

---

## 8. Integration Points

### With Existing .NET POS / Branch System

1. **Webhook from POS → Node API**  
   - When UAE branch creates PO in POS → `POST https://logistics-api/pos/webhook`  
   - Payload: `{ poNumber, branchId, items[], ... }`

2. **API from Node → .NET**  
   - Fetch branch/location data: `GET https://pos-api/branches`  
   - Sync product catalog if shared

3. **Shared Auth (optional)**  
   - OAuth2 / OpenID Connect from .NET IdP  
   - Or API key for service-to-service

---

## 9. Future Enhancement Hooks

| Feature | Hook Point | Notes |
|---------|------------|-------|
| **AI Delay Prediction** | `Shipment.status` | Before `IN_FLIGHT`, call prediction service |
| **Spoilage Risk** | `Box`, `Shipment` | After `ARRIVED`, compute risk from perishability + delay |
| **Cold Chain Monitoring** | `Box`, `Shipment` | IoT temp sensors → store in `BoxTemperatureReadings` |
| **Profit Analysis** | `Shipment`, `DeliveryCompletion` | Cost per shipment vs revenue per PO |

---

## Implementation Order (Recommended)

1. **Phase 1:** Auth, Organizations, Locations, Products  
2. **Phase 2:** PO (Module 1), Packing (Module 2), Labeling (Module 3)  
3. **Phase 3:** Shipment (4), Flight Dispatch (5)  
4. **Phase 4:** Arrival & Clearance (6), Warehouse (7)  
5. **Phase 5:** Delivery (8, 9, 10)  
6. **Phase 6:** Tracking API, Driver App, Integrations  

---

*Document Version: 1.0 | Last Updated: Feb 2025*
