# Servicio de Settlements Engine (Reconciliación, Diferenciales y Ajustes)

## 1. Objetivo
El **Settlements Engine** gestiona la **conciliación automática** y la ejecución de liquidaciones para merchants, asegurando que todos los flujos de dinero (pay-ins, payouts, fees, FX) estén correctamente conciliados y ajustados.

Funcionalidades clave:
- Reconciliación entre transacciones internas y externas
- Cálculo de diferencias y ajustes
- Generación de órdenes de pago a merchants
- Integración con Ledger y FX Engine
- Auditoría completa y trazabilidad

---

## 2. Alcance
- Recepción de datos de pagos, refunds y fees
- Comparación contra procesadores y bancos (archivos, APIs, webhooks)
- Identificación de diferenciales: montos faltantes, sobregiros, errores de FX
- Aplicación de ajustes automáticos o manuales
- Actualización de balances y ledger
- Reportes de settlement diario, semanal o mensual

---

## 3. Flujo de reconciliación automática
```
1. Scheduler dispara el proceso de settlement
2. Settlements Engine recopila:
    - Pay-ins completados
    - Refunds ejecutados
    - Fees calculadas
    - Movimientos de payouts
3. Se compara con información externa (PSP, bancos)
4. Se identifican diferencias:
    - Discrepancias de montos
    - Pagos faltantes
    - Errores de FX
5. Se generan ajustes automáticos:
    - Asientos compensatorios en ledger
    - Corrección de balances en wallets
6. Se generan órdenes de pago a merchants
7. Publicación de eventos: SETTLEMENT_COMPLETED, ADJUSTMENTS_APPLIED
8. Generación de reportes y logs para auditoría
```

---

## 4. Tipos de ajustes
- Ajustes por diferencias de FX
- Ajustes por fees no aplicadas correctamente
- Reversos de pagos duplicados o fallidos
- Compensaciones internas entre wallets y cuentas operativas

---

## 5. Integración con otros servicios
- **Ledger Service**: registrar cada ajuste como asiento contable
- **FX Engine**: conversión de montos multi-moneda
- **Fees Engine**: recalcular fees si se aplican cambios
- **Wallet Service**: actualizar saldos de merchants y reserves
- **Event Bus**: publicar eventos de reconciliación y ajustes

---

## 6. Modelos de datos internos
### 6.1 Settlement Record
```json
{
  "settlement_id": "uuid",
  "merchant_id": "m123",
  "period_start": "2024-02-01T00:00:00Z",
  "period_end": "2024-02-01T23:59:59Z",
  "currency": "USD",
  "gross_amount": 5000.00,
  "fees_total": 210.00,
  "refunds_total": 120.00,
  "fx_adjustments": 42.15,
  "net_amount": 4627.85,
  "adjustments": [
    {"type": "fx_correction", "amount": 5.00, "reason": "Rate mismatch"},
    {"type": "fee_adjustment", "amount": -2.00, "reason": "Processor fee misapplied"}
  ],
  "status": "COMPLETED",
  "created_at": "2024-02-02T01:00:00Z"
}
```

### 6.2 Adjustment Record
```json
{
  "adjustment_id": "uuid",
  "settlement_id": "uuid",
  "type": "fx_correction",
  "amount": 5.00,
  "currency": "USD",
  "applied_at": "2024-02-02T02:00:00Z",
  "reason": "Rate mismatch",
  "ledger_journal_id": "journal_uuid"
}
```

---

## 7. Estados de settlement
```
CREATED → RECONCILING → READY_FOR_PAYOUT → PAYOUT_SENT → COMPLETED
                                    ↘ FAILED
```
- `RECONCILING`: proceso automático de comparación y ajustes
- `FAILED`: inconsistencias no reconciliables automáticamente

---

## 8. API Endpoints (ejemplo)
### 8.1 POST /settlements/execute
```json
{
  "merchant_id": "m123",
  "period_start": "2024-02-01T00:00:00Z",
  "period_end": "2024-02-01T23:59:59Z"
}
```
- Inicia el proceso de settlement y reconciliación

### 8.2 GET /settlements/{id}
- Devuelve estado, ajustes aplicados, net_amount y eventos asociados

---

## 9. Reconciliación multi-moneda y FX
- Identificar diferencias entre moneda de origen y moneda de liquidación
- Aplicar corrección FX y registrar en ledger
- Reportar gain/loss por diferencias de FX
- Control de redondeo y consistencia determinística

---

## 10. Automatización y alertas
- Scheduler programado (diario, T+1, semanal)
- Alertas por diferenciales mayores a umbral configurado
- Logs de reconciliación y auditoría disponibles para control interno
- Capacidad de aplicar ajustes manuales supervisados si la reconciliación automática falla

---

## 11. Auditoría y trazabilidad
- Cada ajuste tiene referencia a event_id, ledger_journal_id y business_event_id
- Histórico completo de cambios y reversiones
- Reproducibilidad de settlement y ajustes históricos

---

## 12. Métricas del servicio
- % de settlements reconciliados automáticamente
- Tiempo promedio de reconciliación por merchant
- Diferencias totales ajustadas
- Fallos por FX y fees
- Logs y eventos publicados por periodo

---
> Este archivo puede ampliarse con diagramas de secuencia ASCII o Mermaid, ejemplos de reconciliación multi-moneda y batch settlements, y scripts de testing de ajustes automáticos.