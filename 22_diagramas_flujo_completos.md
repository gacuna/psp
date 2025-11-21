# Diagramas de Flujo Completos de la Plataforma

Este archivo reúne diagramas de secuencia para cada módulo crítico de la plataforma de pagos, en formato ASCII para referencia rápida.

---

## 1. Pay-in → Ledger → Fees → FX → Settlements
```
Usuario        Plataforma        Procesador         Event Bus        FX Engine        Fees Engine       Ledger Service      Settlements Engine      DB Ledger
  |               |                 |                   |                 |                 |                  |                      |                 |
  |--- Pay-in --->|                 |                   |                 |                 |                  |                      |                 |
  |               |--- API Call --->|                   |                 |                 |                  |                      |                 |
  |               |                 |-- Confirma ------>|                 |                 |                  |                      |                 |
  |               |<-- callback ----|                   |                 |                 |                  |                      |                 |
  |               |--- Publish Event PAYIN_COMPLETED -->|                 |                 |                  |                      |                 |
  |               |                 |                   |--- Event ------>|                 |                  |                      |                 |
  |               |                 |                   |                 |--- Calc FX ----->|                  |                      |                 |
  |               |                 |                   |                 |                 |--- Calc Fees --->|                      |                 |
  |               |                 |                   |                 |                 |                  |--- Create JE/JL --->|                 |
  |               |                 |                   |                 |                 |                  |                      |-- Settlement -->|
  |               |                 |                   |                 |                 |                  |                      |                 |
```

---

## 2. Payout → Ledger → Wallet → Settlements
```
Plataforma     Payout Service      Event Bus        Ledger Service      Wallet Service      DB Ledger
  |               |                   |                 |                  |                 |
  |--- Init Payout ->|                 |                 |                  |                 |
  |               |--- Validate & Calc -->|             |                  |                 |
  |               |--- Publish PAYOUT_INIT -->|        |                  |                 |
  |               |                   |--- Event --->|                  |                 |
  |               |                   |             |--- Create JE/JL ->|                 |
  |               |                   |             |                  |--- Update Balance|                 |
  |               |<-- Ack -----------|             |                  |                 |
```

---

## 3. Refund → Ledger → FX → Fees → Wallet
```
Usuario       Plataforma       Event Bus      FX Engine      Fees Engine       Ledger Service      Wallet Service
  |               |               |               |               |                  |                  |
  |--- Solicita Refund --->|      |               |               |                  |                  |
  |                       |--- Publish REFUND_INIT -->|          |               |                  |                  |
  |                       |               |--- Calc FX -->|     |               |                  |                  |
  |                       |               |               |--- Calc Fees -->|     |                  |                  |
  |                       |               |               |               |--- Create JE/JL --->|                  |
  |                       |               |               |               |                  |--- Update Balance --->|
  |<-- Refund Confirmado --|               |               |               |                  |                  |
```

---

## 4. Settlements & Reconciliation
```
Settlement Engine   Ledger Service       FX Engine        Fees Engine        Wallet Service      DB Ledger
        |                 |                 |                 |                  |                 |
        |--- Gather Transactions --->|     |                 |                  |                 |
        |                             |--- Query Ledger -->|                  |                 |
        |                             |                   |--- Query FX -->   |                 |
        |                             |                   |                 |--- Query Fees -->|
        |--- Identify Differences --->|                   |                 |                  |                 |
        |--- Apply Adjustments ------>|--- Create JE/JL -->|                 |                  |                 |
        |--- Update Wallets -------->|                   |                 |                  |                 |
        |--- Publish SETTLEMENT_COMPLETED -->|           |                 |                  |                 |
```

---

## 5. Reporting & Monitoring
```
All Services      Event Bus       Metrics Engine       Dashboard API       Alerting Engine      Reporting Scheduler
     |               |                 |                  |                     |                  |
     |--- Publish Events -->|         |                  |                     |                  |
     |                       |--- Consume Event -->| |                     |                  |
     |                       |                 |--- Aggregate Metrics ->|                     |                  |
     |                       |                 |                      |--- Generate Dashboards -->|                  |
     |                       |                 |                      |--- Trigger Alerts ------->|                  |
     |                       |                 |                      |                         |--- Generate Reports -->|
```

---
> Este archivo puede ampliarse con diagramas Mermaid y desgloses detallados para cada sub-proceso del engine de pagos.
