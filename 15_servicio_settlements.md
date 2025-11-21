# Servicio de Settlements (Liquidaciones)

## 1. Objetivo del Servicio
El Servicio de **Settlements** es responsable de calcular, consolidar y ejecutar las liquidaciones financieras entre:
- La plataforma ↔️ los merchants
- La plataforma ↔️ los procesadores externos
- La plataforma ↔️ los bancos
- Flujos internos entre diferentes cuentas operativas (operational → reserve → treasury)

Su función principal es **transformar la actividad transaccional (pay-ins, payouts, fees, FX)** en movimientos financieros estructurados, conciliados y ejecutables.

---
# 2. Alcance del Servicio
Incluye:
- Cálculo de liquidaciones por períodos (t+0, t+1, semanal, mensual)
- Cálculo de comisiones, fees y splits
- Conciliación con procesadores y bancos
- Generación de órdenes de payout para liquidar
- Generación de reportes
- Aplicación de FX
- Gestión de ajustes manuales y correcciones
- Integración con ledger
- Manejo de saldos retenidos (rolling reserve)

---
# 3. Conceptos Fundamentales

### 3.1 Settlement Window
Intervalo de tiempo sobre el cual se calcula una liquidación.
Ejemplos:
- Diaria: 00:00 → 23:59
- T+1: todas las transacciones del día anterior
- Semanal

### 3.2 Componentes de una liquidación
- Total pay-ins capturados
- Total refunds procesados
- Total chargebacks retenidos
- Payouts ya ejecutados
- Fees de procesamiento
- Tarifas del método de pago
- Impuestos aplicados
- Retenciones (reservas, hold)

---
# 4. Arquitectura del Servicio

## Componentes principales
- **Settlement Calculator**: motor que calcula montos netos a liquidar.
- **Data Aggregator**: obtiene información de payins, payouts, fees, ledger, FX.
- **FX Engine**: aplica conversiones de moneda.
- **Rule Engine**: aplica reglas de comisiones y retenciones.
- **Scheduler**: ejecuta cron jobs.
- **Settlement Orchestrator**.
- **Settlement DB**.
- **Generator de órdenes de payout**.
- **Report Generator**.

---
# 5. Flujo Completo de una Liquidación
```
1. Scheduler inicia liquidación diaria
2. Settlement Service obtiene transacciones del período
3. Se calculan importes:
    - ingresos (pay-ins)
    - egresos (refunds, payouts)
    - fees
    - impuestos
    - ajustes
4. Se aplica FX cuando corresponde
5. Se crea un settlement con estado CREATED
6. Orchestrator genera un payout asociado (si hay fondos a enviar)
7. Payout Service ejecuta transferencia
8. Settlement pasa a COMPLETED
9. Ledger registra asiento contable consolidado
10. Sistema genera reporte
```

---
# 6. Estados de un Settlement
```
CREATED → CALCULATED → READY_FOR_PAYOUT → PAYOUT_SENT → COMPLETED
                                  ↘ FAILED
```

---
# 7. Modelo de Datos Interno
```json
{
  "settlement_id": "uuid",
  "merchant_id": "m123",
  "period_start": "2024-02-01T00:00:00Z",
  "period_end": "2024-02-01T23:59:59Z",
  "currency": "USD",
  "gross_amount": 5500.00,
  "refunds": 120.00,
  "chargebacks": 0.00,
  "fees": 210.00,
  "fx_adjustments": 42.15,
  "net_amount": 5167.85,
  "status": "READY_FOR_PAYOUT",
  "created_at": "2024-02-02T01:00:00Z"
}
```

---
# 8. Validaciones Críticas
- Cierre contable del período
- Integridad de pay-ins y refunds
- Conciliación vs procesador externo
- Balance suficiente del merchant
- Verificación de no-duplicidad
- Procesos AML (si aplica por volumen)

---
# 9. Cálculo de Fees y Splits

### Ejemplo
```
Pay-ins del período:         10,000.00 USD
Refunds:                        300.00
Chargebacks:                     0.00
Fee plataforma (1.5%):         150.00
Fee procesador:                 40.00
Fee bancario:                   10.00
-------------------------------
NETO:                        9,500.00
```

---
# 10. Aplicación de FX
Cuando el merchant cobra en múltiples monedas pero quiere liquidación en una sola.

### Ejemplo
```
Transacciones: 500 USD + 4000 BRL
FX BRL → USD: 4.00
4000 BRL = 1000 USD
Total USD = 1500 USD
```

### Redondeo
- Redondeo bancario (“round half to even”)
- Configurable por merchant

---
# 11. Scheduler y Frecuencias
- Liquidación diaria
- T+1 automática
- Semanal
- Mensual
- Manual

Reintentos + logs + auditoría.

---
# 12. Generación de Orden de Payout
Si el net_amount es positivo, se crea:
```json
{
  "payout_id": "po_abc",
  "amount": 5167.85,
  "currency": "USD",
  "destination": {
    "type": "merchant_primary_bank"
  }
}
```

Se envía al Servicio de Payouts.

---
# 13. Publicación de Eventos
- `SETTLEMENT_CREATED`
- `SETTLEMENT_CALCULATED`
- `SETTLEMENT_READY`
- `SETTLEMENT_COMPLETED`

---
# 14. Conciliación externa
Comparación con archivos o webhooks de procesadores:
- Stripe balance report
- Adyen settlement detail report
- PayPal financial summary
- Bancos: archivos ACH, SEPA, SPEI

Acciones:
- Ajustes
- Reversos
- Correcciones

---
# 15. Métricas del Servicio
- Volumen liquidado por día
- Diferencias de conciliación
- Tiempo de cálculo por merchant
- Errores por FX
- Cruce de fees con procesadores

---
# 16. Checklist Técnico
- [ ] Scheduler configurado
- [ ] Motor de cálculo validado
- [ ] Integración con FX Engine
- [ ] Integración con Payins/Payouts
- [ ] Conciliación externa activa
- [ ] Auditoría
- [ ] Idempotencia por período
- [ ] Proceso antifraude/AML

---
> Este archivo puede expandirse con: diagramas completos de settlement, manejo multi-monedas avanzado, reconciliación bancaria automática, reportes customizados, y ejemplos por región (EU, Latam, APAC).
