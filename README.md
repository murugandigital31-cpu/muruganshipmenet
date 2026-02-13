# Perishable Air Cargo Logistics System

🇮🇳 India → 🇦🇪 UAE | Flowers • Vegetables • Leaves • Food Items

A specialized ERP for perishable air cargo logistics: **PO → Packing → Flight → Warehouse → Delivery**.

## Features
- **PO Receive** — India packing center receives POs from UAE branches
- **Packing Management** — Sessions, boxes (carton/thermocol/plastic), materials tracking
- **Box Labeling** — QR labels with destination, weight, product type
- **Shipment Scheduling** — Flight assignment, AWB, split support
- **Flight Dispatch** — Status flow: Ready → Loaded → In Flight
- **Arrival & Clearance** — Customs, box scanning at SHJ/AUH
- **Warehouse Processing** — Receive, verify, redistribute to shops
- **Delivery Management** — Runs, driver app, POD
- **Transit Support** — Multi-stage (e.g. Sharjah → Abu Dhabi → Shops)
- Real-time tracking & temperature monitoring (future)

## Architecture

📄 **[Complete Technical Architecture](docs/ARCHITECTURE.md)** — Database schema, APIs, modules, roles, integration points.

## Tech Stack
- **Backend:** Node.js, Express, Mongoose
- **Database:** MongoDB
- **Integration:** REST/Webhook (connects to existing .NET POS/Branch systems)

## Getting Started
```bash
npm install
# Configure .env (MONGODB_URI, JWT_SECRET, etc.)
npm run server   # nodemon
npm run client   # frontend (when available)
```