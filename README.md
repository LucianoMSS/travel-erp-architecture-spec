# ✈️ Module 2: Global Distribution System (GDS) PNR Ingestion Pipeline

## 1. Overview & Data Flow Architecture

The GDS Interface engine automatically imports passenger reservation files (PNRs - Passenger Name Records) from GDS platforms (Amadeus, Sabre, Travelport) and transforms unformatted reservation data into structured financial ledger entries.

```mermaid
flowchart TD
    GDS[GDS Terminal / Amadeus / Sabre API] -->|PNR Data Stream| Interface[GDS Ingestion Daemon]
    
    subgraph Staging Schema
        Interface --> GDS_Gen[gds_general\nPNR Code, Creation Date]
        Interface --> GDS_Vuelos[gds_vuelos\nFlight Segments, Origin/Dest]
        Interface --> GDS_Hoteles[gds_hoteles\nHotel Stays & Confirmation IDs]
        Interface --> GDS_Boletos[gds_boletos\nIATA 13-Digit Ticket Numbers]
        Interface --> GDS_Remarks[gds_remarks\nCost Center & Corporate IDs]
    end

    subgraph ERP Invoicing Engine
        GDS_Gen --> Mapper[ERP Object Mapper]
        GDS_Boletos --> Mapper
        Mapper --> Invoice[datos_factura\nInvoicing Ledger]
        Mapper --> Ledger[cxc\nAccounts Receivable]
    end
```

---

## 2. Staging Pipeline Components

### 2.1 PNR Header (`gds_general`)
Pointers for every imported PNR session.
- `pnr_code`: 6-character alphanumeric Record Locator (e.g., `X7K2P9`).
- `creation_date`: Timestamp of reservation creation.
- `agent_sine`: Sine code of booking agent.

### 2.2 Flight Segments (`gds_vuelos`)
Stores itinerary leg details.
- `origin`: 3-letter IATA airport code (e.g., `MEX`).
- `destination`: 3-letter IATA airport code (e.g., `CUN`).
- `airline_code`: 2-character IATA airline code (e.g., `AM`).
- `flight_number`: Flight number string.

### 2.3 Ticket Numbers (`gds_boletos`)
Contains 13-digit IATA electronic ticket numbers.
- `ticket_number`: Full ticket sequence (e.g., `1392405829102`).
- `passenger_name`: Traveler name string.
- `bsp_code`: 3-digit IATA airline prefix.
