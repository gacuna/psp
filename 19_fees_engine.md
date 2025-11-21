# Servicio de Fees Engine (Reglas, Motores, Cálculos y Versionado)

## 1. Objetivo
El **Fees Engine** es responsable de calcular y aplicar todos los tipos de tarifas dentro de la plataforma de pagos:
- Fees de la plataforma (por transacción, por suscripción, por uso)
- Fees de procesadores externos
- Impuestos y retenciones asociados a fees
- Split de fees entre actores (merchants, partners, affiliates)

El motor debe garantizar **consistencia, trazabilidad y versionado**, permitiendo actualizar reglas sin afectar transacciones históricas.

---

## 2. Alcance
- Definición y versionado de reglas de fees
- Cálculo automático de fees al momento de:
  - Pay-ins
  - Payouts
  - Refunds
  - Settlements
- Aplicación de tasas fijas, porcentuales o combinadas
- Reglas condicionadas por: moneda, volumen, tipo de merchant, región, método de pago
- Registro histórico de versiones de reglas para auditoría
- Publicación de eventos de fees calculados para ledger y reporting

---

## 3. Componentes del Fees Engine
- **Rules Repository**: almacena reglas activas y versiones anteriores
- **Calculation Engine**: aplica reglas sobre eventos transaccionales
- **Version Controller**: asegura que cambios en reglas no afecten transacciones anteriores
- **Event Handler**: recibe eventos de negocio y genera cálculo de fees
- **Metadata Store**: guarda resultados detallados de cada cálculo para auditoría

---

## 4. Tipos de Fees
| Tipo | Descripción | Aplicación |
|------|------------|-----------|
| Platform Fee | Comisión por uso de la plataforma | Pay-ins / Subscriptions |
| Processor Fee | Comisión cobrada por PSP | Pay-ins / Payouts |
| Transaction Tax | IVA, GST o impuestos locales | Platform Fees |
| Split Fee | Porcentaje a partners o affiliates | Pay-ins / Payouts |
| FX Fee | Margen sobre conversión de moneda | Multi-currency transactions |

---

## 5. Flujo de cálculo de fees
```
1. Evento de negocio recibido (pay-in, payout, refund)
2. Identificar merchant, método de pago, moneda, volumen, región
3. Consultar Rules Repository para reglas activas (según version)
4. Calcular fees:
   - Platform Fee
   - Processor Fee
   - FX Fee (si aplica)
   - Taxes
5. Registrar fees en Metadata Store
6. Publicar evento FEES_CALCULATED con detalle
7. Actualizar Ledger (journal entries) con cada fee
```

---

## 6. Versionado de reglas
- Cada regla tiene:
  - `rule_id`
  - `version`
  - `effective_from`
  - `effective_to` (nullable)
- Cuando se publica una nueva versión, las transacciones previas se siguen calculando con la versión vigente al momento de la operación
- Se permite simulación de reglas para testing

---

## 7. Ejemplo de regla JSON
```json
{
  "rule_id": "platform_fee",
  "version": 3,
  "condition": {
    "merchant_type": "standard",
    "payment_method": ["card","wallet"],
    "currency": "USD"
  },
  "calculation": {
    "type": "percentage",
    "value": 1.5
  },
  "taxable": true,
  "effective_from": "2024-01-01T00:00:00Z"
}
```

---

## 8. Cálculo de ejemplo
- Pay-in 100 USD
- Platform Fee: 1.5% → 1.50 USD
- IVA 10% sobre fee → 0.15 USD
- Processor Fee: 2.50 USD
- Split 20% a partner → 0.30 USD
- Neto recibido merchant → 100 - 1.50 - 2.50 - 0.30 = 95.70 USD

---

## 9. Integración con otros servicios
- **Ledger Service**: registrar cada fee como asiento contable
- **FX Engine**: calcular fees en moneda destino si hay conversión
- **Event Bus**: notificar FEES_CALCULATED para downstream services
- **Refunds**: recalcular fees si la transacción es parcialmente devuelta
- **Settlements**: consolidar fees para liquidación de merchants

---

## 10. API Endpoints (ejemplo)
### 10.1 POST /calculate-fees
```json
{
  "amount": 100.00,
  "currency": "USD",
  "merchant_id": "m123",
  "payment_method": "card",
  "business_event_id": "be_001"
}
```
- Respuesta:
```json
{
  "fees": [
    {"type": "platform", "amount": 1.50, "tax": 0.15},
    {"type": "processor", "amount": 2.50},
    {"type": "split", "amount": 0.30}
  ],
  "net_amount": 95.70
}
```

### 10.2 GET /rules
- Devuelve reglas activas y versiones históricas

---

## 11. Auditoría y trazabilidad
- Cada cálculo guarda:
  - `business_event_id`
  - `rule_version`
  - `timestamp`
  - `fees_breakdown`
- Permite reproducir cálculos históricos y conciliar discrepancias

---

## 12. Consideraciones
- Soporte para reglas complejas y condicionales
- Posibilidad de rollback de reglas en caso de error crítico
- Integración con simuladores y entornos de prueba
- Redondeos consistentes (bankers rounding o configurable)
- Multi-moneda y FX-aware

---
> Este documento puede expandirse con diagramas de secuencia, ejemplos de cambios de versión de reglas, batch calculation para grandes volúmenes y scripts de testing de reglas.