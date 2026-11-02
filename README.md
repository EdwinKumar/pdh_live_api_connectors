# 🌊 2050 Blue Pacific — Live Public API Connectors Module (`pdh_live_api_connectors.py`)

## 📌 Executive Overview

The `pdh_live_api_connectors.py` library is a production-grade Python package designed to execute automated data extraction from live public REST, SDMX, and OData web services across regional and global data custodians.

Developed under the **2050 Strategy for the Blue Pacific Continent** framework, this module connects the regional monitoring system directly to centralized data repositories—specifically the **Pacific Data Hub (SPC)**, **World Bank Data Library**, **WHO Global Health Observatory**, and the **Asian Development Bank (ADB) Data Library**.

---

## 🛠️ Supported Connectors & API Class Reference

### 1. `PacificDataHubConnector` (Secretariat of the Pacific Community - SPC)
* **Target Domain:** `api.pacificdata.org` (CKAN) & `stats.pacificdata.org` (SDMX)
* **Methods:**
  * `search_datasets(query: str = "2050 Strategy", rows: int = 10)`: Searches the CKAN catalog for regional datasets and returns metadata packages.
  * `query_datastore_resource(resource_id: str, limit: int = 100)`: Queries tabular data resources directly via the CKAN DataStore API.
  * `get_sdmx_data(flow_id: str = "DF_PACIFIC_INDICATORS", key: str = "all")`: Fetches structured SDMX statistical time-series feeds.

### 2. `WorldBankDataConnector` (World Bank Open Data)
* **Target Domain:** `api.worldbank.org/v2`
* **Methods:**
  * `get_country_indicator(country_code: str = "FJI", indicator: str = "NY.GDP.PCAP.KD.ZG", date_range: str = "2015:2025")`: Fetches country-level economic growth, governance (WGI), and poverty indicators for all 18 Forum Member countries in JSON format.

### 3. `WHOGlobalHealthConnector` (WHO Global Health Observatory)
* **Target Domain:** `ghoapi.azureedge.net/api`
* **Methods:**
  * `get_indicator_data(indicator_code: str = "NCD_MORT_3070", filter_str: Optional[str] = None)`: Queries health metrics (such as premature NCD mortality, tobacco use rates, and UHC coverage index) via OData interfaces.

### 4. `ADBDataLibraryConnector` (Asian Development Bank)
* **Target Domain:** `data.adb.org`
* **Methods:**
  * `search_adb_datasets(query: str = "Pacific", rows: int = 10)`: Queries ADB's public CKAN dataset registry for regional economic forecasts, trade balances, and development effectiveness indicators.

---

## 💻 Quickstart & Python Code Examples

### Basic Usage in Python

```python
from pdh_live_api_connectors import (
    PacificDataHubConnector,
    WorldBankDataConnector,
    WHOGlobalHealthConnector,
    ADBDataLibraryConnector
)

# 1. Query Pacific Data Hub for Climate Finance Datasets
pdh = PacificDataHubConnector()
pdh_results = pdh.search_datasets(query="climate finance", rows=5)

if pdh_results["status"] == "success":
    datasets = pdh_results["payload"]["result"]["results"]
    print(f"Found {len(datasets)} datasets on Pacific Data Hub.")

# 2. Fetch GDP Growth for Fiji from World Bank API
wb = WorldBankDataConnector()
wb_results = wb.get_country_indicator(country_code="FJI", indicator="NY.GDP.PCAP.KD.ZG")

if wb_results["status"] == "success":
    data_points = wb_results["payload"][1]
    print(f"Retrieved {len(data_points)} historical GDP records for Fiji.")

# 3. Query WHO NCD Mortality Data
who = WHOGlobalHealthConnector()
who_results = who.get_indicator_data(indicator_code="NCD_MORT_3070")
```

---

## ⚙️ Resilience, Security & Offline Airgap Handling

1. **Custom User-Agent & Headers:** All HTTP requests transmit identifying headers (`User-Agent: 2050-Strategy-Pipeline/1.0 (UNFPA/PIFS)`) to ensure compliance with API rate-limiting rules.
2. **Network Timeout Guardrails:** Requests enforce strict `timeout=15` limits to prevent blocking server threads during temporary network degradation.
3. **Graceful Airgap / Offline Fallback:** In environments without internet routing (such as air-gapped test sandboxes), the connectors gracefully intercept connection errors (`requests.exceptions.ConnectionError`) and return structured error diagnostics (`status: "network_airgap_notice"`) without crashing the downstream application.

---

## 🔄 Data Harmonisation & Mapping Protocol

Data returned by these connectors is automatically mapped to the **80 Core Outcome Indicators** data model:

```
┌────────────────────────────────┐       ┌────────────────────────────────┐
│   Raw API Payload (JSON/SDMX)  │       │  Fact Core Results Warehouse   │
├────────────────────────────────┤       ├────────────────────────────────┤
│ • Raw Country Name ("Fiji")    │  ───► │ • ISO-3 Code: "FJI"            │
│ • Raw Date ("2024-Q4")         │  ETL  │ • Reporting Period: "2024"     │
│ • Value ("12.5")               │       │ • Actual Value: 12.5           │
│ • Indicator Code ("NY.GDP...") │       │ • Indicator ID: "IND57"        │
└────────────────────────────────┘       └────────────────────────────────┘
```

---

## 🏛️ Governance & Institutional Attribution

* **Author:** Adviser – Data Analytics & Insights
* **Institutional Partners:** United Nations Population Fund (UNFPA) & Pacific Islands Forum Secretariat (PIFS 2050 Unit)
* **Compliance Standards:** Pacific Statistics Methods Board (PSMB) Data Sovereignty Framework & Pacific Data Hub Interoperability Guidelines.
