# Northwind Products - SAP Fiori List Report Application

A SAP Fiori Elements List Report Object Page application that displays product data from the Northwind OData V2 service. Built using SAP Business Application Studio (BAS) and configured for deployment on SAP Business Technology Platform (BTP) Cloud Foundry, with XSUAA authentication and an Application Router for secure OData proxying.

**Repository (D1):** https://github.com/yasmen-ahmed/northwindapp

---

## Architecture Overview

In this project, SAP Business Application Studio (BAS) is used to develop and test the Fiori application.
The Destination Service in SAP BTP stores the connection to the Northwind OData service, allowing the app to access external data securely without hardcoding URLs.
After development, the application is deployed to Cloud Foundry, which hosts and runs the app online. Cloud Foundry uses the configured destination to route requests from the Fiori app to the Northwind backend service.


---

## Setup Instructions

### Prerequisites

- SAP BTP Trial or Enterprise account with Cloud Foundry environment enabled
- SAP Business Application Studio subscription
- Node.js LTS (v18+) and npm
- Cloud Foundry CLI (`cf`) installed

### Step-by-Step Setup

1. **Configure the Northwind Destination in BTP Cockpit:**
   - Navigate to your BTP subaccount → Connectivity → Destinations
   - Create a new destination:
     - **Name:** `Northwind`
     - **Type:** HTTP
     - **URL:** `https://services.odata.org`
     - **Proxy Type:** Internet
     - **Authentication:** NoAuthentication
   - Add additional property: `WebIDEUsage` = `odata_gen`
   - Add additional property: `WebIDEEnabled` = `true`

2. **Open SAP Business Application Studio:**
   - Create a Dev Space with "SAP Fiori" extensions enabled
   - Clone or import this repository into the workspace

3. **Install Dependencies:**
   ```bash
   npm install
   ```

4. **Run Locally with Live Data:**
   ```bash
   npm start
   ```
   This starts the Fiori tools development server, proxying OData requests to `services.odata.org`.

5. **Run with Mock Data (offline):**
   ```bash
   npm run start-mock
   ```

6. **Build for Cloud Foundry Deployment:**
   ```bash
   npm run build:mta
   ```

7. **Deploy to Cloud Foundry:**
   ```bash
   cf login
   npm run deploy
   ```
# mbt build && cf deploy mta_archives/northwindapp_1.0.0.mtar
---

## OData Entity Used

### Entity: `Products` (from Northwind OData V2 Service)

### Why `Products`?

`Products` was selected as the main entity set because it is the core catalog entity in the Northwind sample service: each row is a sellable item with a clear ID, name, price, and stock level. That makes it a good fit for a List Report that must show real records, support filtering and sorting, and allow drill-down to an Object Page for a single product.

### List Report columns configured

The task required at least **three meaningful columns**. Four were configured in `webapp/annotations/annotation.xml` via the `UI.LineItem` annotation on `NorthwindModel.Product`:

- **Product ID** (`ProductID`) — unique key for each product
- **Product Name** (`ProductName`) — readable product label
- **Unit Price** (`UnitPrice`) — sales price (decimal)
- **Units In Stock** (`UnitsInStock`) — current inventory level

The same fields are also used as **selection fields** on the filter bar (`UI.SelectionFields`), so users can search and filter the list in BAS preview and after deployment.

### Verification in BAS

After generation, the app was started in BAS (`npm start`) and opened in the **application preview**. The List Report loaded live data from the Northwind OData service and displayed product records in the table. A screenshot of this preview is included as deliverable **D3** — see [`docs/task4.jpeg`](./docs/task4.jpeg).

---

## Challenges Faced

### 1. Destination Configuration and CORS Issues

**Problem:** During initial setup, the Fiori preview in BAS could not reach the Northwind OData service. Requests to `/V2/Northwind/Northwind.svc/` returned 404 errors because the destination was not correctly mapped in the `ui5.yaml` backend proxy configuration.

**Resolution:** Ensured the `ui5.yaml` file's `fiori-tools-proxy` backend configuration matched the exact path prefix (`/V2`) and pointed to the correct destination URL (`https://services.odata.org`). Additionally, verified that the BTP destination had `WebIDEEnabled=true` and `WebIDEUsage=odata_gen` properties set for BAS integration.

### 2. Annotation-Driven UI Without Custom Views

**Problem:** As a Fiori Elements V2 app, the entire UI is driven by annotations rather than custom XML views. Initial attempts to customize the table columns resulted in no visible changes because the annotations were not correctly targeting the `NorthwindModel.Product` namespace.

**Resolution:** Carefully matched the annotation target namespace (`NorthwindModel.Product`) with what the OData metadata defines, and used the correct vocabulary terms (`UI.LineItem`, `UI.SelectionFields`, `UI.HeaderInfo`, `UI.Facets`) to configure the List Report and Object Page.

---

## Bonus Tasks Completed

- **B1 — Deploy to Cloud Foundry (completed):** Built and deployed the MTA archive to Cloud Foundry. The app runs on the Application Router URL. Screenshot includes the browser with the URL visible and `cf apps` output.

- **B2 — Postman OData queries (completed):** Exported a Postman collection and captured OData GET query results against the Northwind service.

- **B3 — XSUAA login on deployed app (completed):** The deployed application prompts for BTP login (XSUAA) before showing the UI. Screenshot taken on the live URL.

- **B4 — Object Page in BAS preview (completed):** The Object Page shows product detail in **General Information** and **Stock Information** sections when a row is selected in the List Report.

- **B5 — Postman POST (writable OData) (completed):** POST request and response documented in Postman against a writable OData service.

- **B6 — SAP Build Work Zone Launchpad (not completed):** The subscription for SAP Build Work Zone failed in the BTP subaccount, so the launchpad could not be configured or tested.

---

## Deployed Application URL

https://5ffadf12trial-dev-northwindapp-approuter.cfapps.us10-001.hana.ondemand.com

---

## Screenshots

All deliverable files are in the [`/docs`](./docs/) folder.

### Mandatory deliverables

- **D2** — Configured Northwind destination in BTP Cockpit: [`docs/task3-configured.png`](./docs/task3-configured.png)
- **D3** — Application running in BAS preview with real data: [`docs/task4.jpeg`](./docs/task4.jpeg)

### Setup and configuration (supporting)

- [`docs/task1.jpeg`](./docs/task1.jpeg) — BAS / project setup
- [`docs/task2-Service.jpeg`](./docs/task2-Service.jpeg) — Service configuration
- [`docs/task2-role.jpeg`](./docs/task2-role.jpeg) — Role configuration
- [`docs/task3-Connection.png`](./docs/task3-Connection.png) — OData service connection in BAS

### Bonus deliverables

- **B1** — Deployed app in browser + `cf apps` output: [`docs/b1.png`](./docs/b1.png)
- **B2** — Postman collection export: [`docs/solex task.postman_collection.json`](./docs/solex%20task.postman_collection.json)
- **B2** — OData query results (top / select / filter / orderby):
  - [`docs/b2-top.png`](./docs/b2-top.png)
  - [`docs/b2-select.png`](./docs/b2-select.png)
  - [`docs/b2-filter.png`](./docs/b2-filter.png)
  - [`docs/b2-orderby.png`](./docs/b2-orderby.png)
- **B3** — Login prompt on deployed application URL: [`docs/b3.png`](./docs/b3.png)
- **B4** — Object Page showing record detail in BAS preview: [`docs/b4.jpeg`](./docs/b4.jpeg)
- **B5** — Postman POST request and response: [`docs/b5-create.png`](./docs/b5-create.png)

---

## Application Details

- **Generation Date and Time:** Thu May 14 2026 10:47:52 GMT+0000 (Coordinated Universal Time)
- **App Generator:** SAP Fiori Application Generator (v1.24.0)
- **Generation Platform:** SAP Business Application Studio
- **Template Used:** List Report Page V2
- **Service Type:** OData URL
- **Service URL:** https://services.odata.org/V2/Northwind/Northwind.svc/
- **Module Name:** northwindapp
- **Application Title:** Northwind Products
- **Namespace:** com.intern.northwindapp
- **UI5 Theme:** sap_horizon
- **UI5 Version:** 1.148.0
- **Enable TypeScript:** False
- **Main Entity:** Products

---

## Running the Application

```bash
# Start with live OData data
npm start

# Start with mock data
npm run start-mock

# Build MTA archive
npm run build:mta

# Deploy to Cloud Foundry
npm run deploy
```
