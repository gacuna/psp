# 17. Ledger Contable de la Plataforma

> Documento: diseño completo del **Ledger (libro mayor contable)** para la plataforma de pagos. Incluye modelo de doble entrada, eventos contables mapeados, esquemas de tablas, ejemplos de asientos, snapshots, conciliación, performance, auditoría y buenas prácticas para integridad financiera.

---

## 1. Objetivo
El ledger es la **fuente de la verdad** para todos los movimientos económicos de la plataforma. Debe permitir:

- Registrar cada evento financiero con integridad (inmutable o append-only).
- Soportar modelo de **doble entrada** para facilitar reporting contable y auditorías.
- Servir como origen para conciliación, settlements y reporting fiscal.
- Permitir reconstrucción histórica y reproducibilidad de saldos.

Requisitos clave:
- Idempotencia y tolerancia a duplicados.
- Alto rendimiento en escrituras y consultas por saldo.
- Auditability (WORM-like logs) y retención según normativa.
- Soporte multi-moneda y tratamiento de FX.
- Integración con Event Bus y microservicios (Pay-ins, Payouts, Refunds, Fees, Settlements).

---

## 2. Principios de diseño
1. **Doble entrada**: cada movimiento genera al menos dos líneas (debe = haber).
2. **Append-only**: no sobrescribir asientos; corregir mediante asientos compensatorios.
3. **Eventos como fuente**: los microservicios publican eventos que el ledger consume y materializa en asientos.
4. **Atomicidad por transacción de negocio**: crear asiento(s) relacionados a un `business_event_id` con atomic commit.
5. **Separación ledger vs balances**: ledger almacena asientos; balances se derivan por agregación (materialized views / snapshots).
6. **Traza y correlación**: cada asiento guarda `trace_id`, `payment_id`, `processor_ref`, `settlement_id`.

---

## 3. Modelo de cuentas (Chart of Accounts) — ejemplo base

| Código | Nombre de cuenta | Tipo |
|--------|------------------|------|
| 1000 | Cash - Bank (operational) | Asset |
| 1010 | Cash - Bank (reserve) | Asset |
| 1200 | Accounts Receivable (pending pay-ins) | Asset |
| 2000 | Payables (pending payouts) | Liability |
| 3000 | Revenue - Processing Fees | Revenue |
| 3010 | Revenue - FX Margin | Revenue |
| 4000 | Expenses - Processor Fees | Expense |
| 5000 | Chargebacks & Reserves | Expense / Contra-Revenue |
| 6000 | Taxes Payable (IVA, GST) | Liability |
| 9000 | Equity / Retained Earnings | Equity |

> Este Chart es un ejemplo; será adaptado según reporting contable y reglas fiscales por jurisdicción.

---

## 4. Esquema de tablas (SQL simplificado)

### 4.1 `journal_entries`
```sql
CREATE TABLE journal_entries (
  journal_id UUID PRIMARY KEY,
  business_event_id UUID, -- correlate with domain event
  created_at TIMESTAMP WITH TIME ZONE,
  posted_at TIMESTAMP WITH TIME ZONE,
  description TEXT,
  currency CHAR(3),
  metadata JSONB -- payment_id, processor_ref, trace_id, settlement_id etc
);
```

### 4.2 `journal_lines`
```sql
CREATE TABLE journal_lines (
  line_id UUID PRIMARY KEY,
  journal_id UUID REFERENCES journal_entries(journal_id),
  account_code INT,
  amount NUMERIC(20,6), -- positive values. Sign determined by account debit/credit convention
  side CHAR(1) CHECK (side IN ('D','C')),
  created_at TIMESTAMP WITH TIME ZONE
);
CREATE INDEX ON journal_lines (account_code);
CREATE INDEX ON journal_lines (journal_id);
```

### 4.3 `balances_snapshot`
Materialized views or table for fast balance reads.
```sql
CREATE TABLE balances_snapshot (
  account_code INT,
  currency CHAR(3),
  balance NUMERIC(20,6),
  snapshot_time TIMESTAMP WITH TIME ZONE,
  PRIMARY KEY(account_code, currency, snapshot_time)
);
```

---

## 5. Mapeo de eventos a asientos (ejemplos)
Cada evento de negocio produce un `journal_entry` con líneas de débito y crédito.

### 5.1 Pay-in completado (cliente paga 100 USD; procesador fee 2.50 USD; platform fee 1.00 USD; IVA 0.10 USD sobre platform fee)
- Evento: `PAYIN_COMPLETED` (payment_id p_1)

**Asientos (simplificado):**

Journal description: "Pay-in p_1 - gross 100 USD"

| Line | Account | Side | Amount USD |
|------|--------|------|------------|
| 1 | 1200 Accounts Receivable (pending pay-ins) | D | 100.00 |
| 2 | 1000 Cash - Bank (operational) | C | 100.00 |

Luego, fees y allocations (same journal or subsequent journal linked by business_event_id):

| Line | Account | Side | Amount USD |
|------|--------|------|------------|
| 3 | 4000 Expenses - Processor Fees | D | 2.50 |
| 4 | 1000 Cash - Bank (operational) | C | 2.50 |
| 5 | 3000 Revenue - Platform Fees | C | 1.00 |
| 6 | 6000 Taxes Payable | C | 0.10 |
| 7 | 1000 Cash - Bank (operational) | D | 1.10 |

**Nota:** dependiendo de política, platform fee puede ser registrado como revenue cuando se reconoce (immediately or deferred).

### 5.2 Payout ejecutado (se envía 50 USD a proveedor)
- Evento: `PAYOUT_COMPLETED` (payout_id po_1)

| Line | Account | Side | Amount USD |
|------|--------|------|------------|
| 1 | 2000 Payables (pending payouts) | D | 50.00 |
| 2 | 1000 Cash - Bank (operational) | C | 50.00 |

Si se aplica retención fiscal:
| 3 | 6000 Taxes Payable | C | 2.50 |
| 4 | 2000 Payables | D | 2.50 |

### 5.3 Refund completo (100 USD)
- Evento: `REFUND_COMPLETED` (refund r_1)

Asientos:
| Line | Account | Side | Amount USD |
|------|--------|------|------------|
| 1 | 2000 Payables OR 1000 Cash (reverse) | D | 100.00 |
| 2 | 1200 Accounts Receivable (or Revenue reversal) | C | 100.00 |

Ajustar fees si policy mandates refunds of platform fees.

### 5.4 FX adjustment
- Registro del gain/loss por FX se realiza en cuenta `FX Gains/Losses` (e.g., 7000)
- Aplica a la conversión entre moneda de ledger y moneda de liquidación.

---

## 6. Multi-moneda y FX handling
- Cada `journal_entry` tiene un `currency`.
- Mantener tanto montos en `original_amount` + `original_currency` y `functional_amount` en `book_currency` (por ejemplo USD) con el `fx_rate` aplicado.
- Guardar `fx_rate`, `fx_timestamp` y `fx_provider` en `metadata`.
- Reconocer diferencias por FX en cuenta específica (gains/losses).
- Para settlements que agregan múltiples monedas, efectuar conversión en settlement engine y crear asiento con detalles de conversión.

---

## 7. Snapshots y materialized balances
- No calcular balances on-the-fly desde `journal_lines` en cada request; en lugar, generar **snapshots** por intervalo (cada 5 min, 1h, EOD).
- Estrategia:
  - Incremental updates: al publicar un journal, también actualizar `balances_snapshot` para la cuenta correspondiente (transactional write if ledger and snapshot share transaction scope).
  - Materialized view para queries históricas.
- Mantener historial de snapshots para auditoría y reconstrucción.

---

## 8. Consistencia, Idempotencia y Reconciliation
- **Idempotencia:** journal creation must check `business_event_id` uniqueness. If duplicate event arrives, ignore or reconcile.
- **Reconciliation pipelines:**
  - Periodic matching between ledger and external reports (processor settlement files, bank statements).
  - Reconciliation engine marks entries as `reconciled` with references (external_file_id, line_ref).
  - Differences produce `adjustment` journals (with reason code) — never mutate original journals.

---

## 9. Snapshots de prueba y reconciliación ejemplo (numérico)
Supongamos periodo diurno:
- Pay-ins bruto: 5,000.00
- Processor fees: 150.00
- Platform fees: 50.00 (IVA 5.00)
- Payouts: 3,000.00

Ledger should show:
- Cash inflow: +5,000
- Cash outflow: -3,150 (payouts + processor fees + payouts bank costs)
- Net movement: +850 (before platform fees tax)

Reconciliation: match `bank_statement` records vs `journal_entries` filtered by `payment_id`/`processor_ref` and by time window.

---

## 10. Auditoría y retención
- Logs append-only y exportables en formato inmutable (WORM if required by jurisdiction).
- Retención mínima por jurisdicción (e.g., 7 años en muchas jurisdicciones fiscales) — plan de archivado y cold storage.
- Registrar actor/actor_id en metadata (who created/reconciled the journal).

---

## 11. Performance y escalabilidad
- Shardear `journal_lines` por `account_code` or `currency` para lecturas paralelas.
- Archivar líneas antiguas a cold storage y mantener pre-aggregated snapshots.
- Uso de columnar storage / data warehouse para reporting.
- Optimizar índices para queries frecuentes: `account_code`, `payment_id` in metadata, `created_at`.

---

## 12. Seguridad
- Encriptación en reposo y en tránsito (TLS + DB encryption at rest).
- Acceso restringido por roles: solo contadores y servicios autorizados pueden crear journals en determinados scopes.
- HSM/KMS para claves de firma de asientos si se requiere firma digital de journals.

---

## 13. Integración con sistemas contables externos
- Exportadores para: QuickBooks, Xero, SAP, Oracle NetSuite.
- Mapeos configurables: account_code → external COA.
- Export formats: CSV, QIF, OFX, XML, or direct API pushes.

---

## 14. Error handling y correcciones
- Nunca borrar un journal; para correcciones crear journal compensatorio (reverse journal) con `reversal_of_journal_id` in metadata.
- Registrar reason codes and approval workflow for manual adjustments.
- Support for bulk adjustments (e.g., fee rate change retroactive) via controlled batch journals with approval and audit trail.

---

## 15. Tests, invariants y monitoring
- Invariants to check periodically:
  - Sum(Debits) == Sum(Credits) per journal_id
  - Balances derived from snapshots equal aggregation of lines up to snapshot_time
  - No orphaned journal_lines (every line has journal entry)
- Monitor metrics: journals/sec, snapshot refresh time, reconciliation mismatch rate.

---

## 16. Ejemplos completos (línea por línea)
### 16.1 Pay-in example (100 USD)
Journal entry id: j_payin_001
- Line 1: D 1200 Accounts Receivable 100.00 USD
- Line 2: C 1000 Cash - Bank 100.00 USD
- Line 3: D 4000 Processor Fee Expense 2.50 USD
- Line 4: C 1000 Cash - Bank 2.50 USD
- Line 5: D 1000 Cash - Bank 1.10 USD
- Line 6: C 3000 Revenue - Platform Fees 1.00 USD
- Line 7: C 6000 Taxes Payable 0.10 USD

(Posted atomically; ledger stores metadata {payment_id, processor_ref, trace_id})

### 16.2 Settlement & Payout example
Settlement net_amount 5167.85 USD → generate payout po_001
Journal entry j_settlement_001
- Line: D 2000 Payables 5167.85
- Line: C 1000 Cash - Bank 5167.85

---

## 17. Roadmap de entrega e integración
- Fase 1: Implement ledger core (journal_entries + journal_lines) + simple snapshots
- Fase 2: Integración con Pay-ins / Payouts events + reconciliation pipeline
- Fase 3: Multi-currency, FX_rates recording, gains/losses
- Fase 4: Audit WORM, archival, exports a ERP
- Fase 5: Automated tests, invariants, monitoring & alerts

---

## 18. Conclusión
Un ledger robusto es crítico para la operación de una plataforma de pagos. El diseño propuesto prioriza integridad, auditabilidad, rendimiento y la separación entre el registro histórico inmutable (journal) y vistas/materializaciones de saldo para uso operativo. Implementando las medidas descritas, la plataforma podrá soportar crecimiento, auditorías y requisitos regulatorios.

Si querés, puedo:
- Generar el SQL completo para las tablas propuestas y migraciones.
- Producir scripts de verificación de invariantes (unit tests/integration tests).
- Crear ejemplos de consultas para reporting contable y dashboards.
- Añadir diagramas de secuencia para eventos → asientos.


---

## 19. Diagrama de Secuencia (ASCII) — Evento Pay‑in → Ledger

```
Usuario        Plataforma        Procesador         Event Bus        Ledger Service      DB Ledger
  |               |                 |                   |                 |                 |
  |--- Pay-in --->|                 |                   |                 |                 |
  |               |--- API Call --->|                   |                 |                 |
  |               |                 |-- Confirma ------>|                 |                 |
  |               |<-- callback ----|                   |                 |                 |
  |               |--- Publish Event PAYIN_COMPLETED -->|                 |                 |
  |               |                 |                   |--- Event ------>|                 |
  |               |                 |                   |                 |-- Create JE -->|
  |               |                 |                   |                 |   (journal_entries)
  |               |                 |                   |                 |-- Create JL -->|
  |               |                 |                   |                 |   (journal_lines)
  |               |                 |                   |                 |--- ACK -------->|
  |               |                 |                   |<-- processed ---|                 |
  |               |<-- success -----|                   |                 |                 |
  |               |                 |                   |                 |                 |
```

### Explicación breve del flujo
1. **Usuario inicia un Pay-in** en la plataforma.
2. La plataforma envía el pago al **Procesador**.
3. El procesador confirma el pago y envía un **callback**.
4. La plataforma publica el evento `PAYIN_COMPLETED` en el **Event Bus**.
5. El **Ledger Service** escucha el evento y crea un `journal_entry` + `journal_lines`.
6. Se persiste en la base de datos del ledger.
7. Se confirma el procesamiento al Event Bus.
8. La plataforma informa el éxito al usuario.

---

