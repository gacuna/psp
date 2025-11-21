# Servicio de FX Engine (Cálculo de Tipo de Cambio, Márgenes y Consistencia)

## 1. Objetivo
El **FX Engine** gestiona la conversión de monedas en la plataforma, garantizando:
- Integridad en cálculos de conversión
- Aplicación de márgenes y fees FX
- Consistencia en saldos y reporting contable
- Registro de transacciones FX para auditoría y reconciliación

El FX Engine es crítico para plataformas multi-moneda, especialmente para **pay-ins, payouts, refunds y settlements**.

---

## 2. Alcance del Servicio
- Obtención de tipos de cambio (fuentes externas confiables)
- Normalización y almacenamiento de FX rates
- Aplicación de conversiones para:
  - Liquidaciones
  - Pagos y reembolsos
  - Reportes contables
- Registro de márgenes FX aplicados
- Validación de consistencia y alertas en discrepancias
- Soporte para **multi-moneda** y **cross-currency transactions**

---

## 3. Fuentes de Tipo de Cambio
- APIs bancarias
- Agregadores de FX (ej: Open Exchange Rates, Xignite, OANDA)
- Bancos locales o internacionales (SEPA, ACH, SWIFT)

### 3.1 Estrategias de fuente
- Preferencia de un **proveedor primario**, fallback a secundario
- Almacenamiento histórico para auditoría y reconciliación
- Timestamp de obtención de cada rate

---

## 4. Estructura de datos FX
```json
{
  "currency_pair": "USD/BRL",
  "rate": 5.1123,
  "source": "OANDA",
  "fetched_at": "2024-02-21T10:00:00Z",
  "margin_percent": 0.5,
  "effective_rate": 5.1379
}
```
- `rate`: tasa base del proveedor
- `margin_percent`: porcentaje aplicado por la plataforma
- `effective_rate`: rate final usado para conversiones

---

## 5. Flujo de conversión
```
1. Solicitud de conversión (currency_from, currency_to, amount)
2. FX Engine consulta última tasa válida
3. Se aplica margen y fees FX si corresponde
4. Se registra operación FX (metadata, rate, margin)
5. Se devuelve amount convertido
6. Ledger recibe amount convertido y registra asiento
```

---

## 6. Cálculo de márgenes y fees FX
- Margen se aplica sobre la tasa de mercado: `effective_rate = rate * (1 + margin_percent / 100)`
- Fee fijo opcional por transacción
- Ejemplo:
```
USD → BRL 100 USD
Rate mercado: 5.1123
Margen 0.5% → 5.1123 * 1.005 ≈ 5.1379
Monto BRL = 100 * 5.1379 ≈ 513.79 BRL
```
- Registro en metadata para auditoría

---

## 7. Integridad y consistencia
- Todos los cálculos FX deben ser **determinísticos**
- Redondeo configurable por moneda (bankers rounding o round-half-up)
- Cada operación FX ligada a **trace_id** y **business_event_id**
- Auditoría: guardar rate, timestamp y fuente

---

## 8. Actualización de rates y expiración
- Rates expiran según TTL (ej: 5 min para intraday, 24h para historico)
- Logs de cambios y diferencia vs rates anteriores
- Alertas si variación superior a umbral configurado

---

## 9. Integración con otros servicios
- **Pay-ins**: calcular amount neto en moneda de ledger
- **Payouts**: calcular monto a transferir a moneda destino
- **Refunds**: revertir conversiones si necesario
- **Settlements**: consolidar montos multi-moneda para liquidaciones
- **Ledger**: registrar FX gain/loss si la operación implica conversión

---

## 10. API Endpoints (ejemplo)
### 10.1 GET /fx-rates
- Devuelve tasa actual y márgenes
```json
{
  "currency_from": "USD",
  "currency_to": "BRL",
  "rate": 5.1123,
  "effective_rate": 5.1379,
  "source": "OANDA",
  "fetched_at": "2024-02-21T10:00:00Z"
}
```

### 10.2 POST /convert
```json
{
  "amount": 100.0,
  "currency_from": "USD",
  "currency_to": "BRL",
  "business_event_id": "be_123"
}
```
- Respuesta:
```json
{
  "amount_converted": 513.79,
  "effective_rate": 5.1379,
  "fee_applied": 0.0
}
```

---

## 11. Monitoreo y métricas
- Latencia de actualización de rates
- Diferencia entre rate de mercado y rate efectivo aplicado
- Tasa de conversiones FX fallidas
- Logs de auditoría y reconciliación diaria

---

## 12. Consideraciones
- Multi-source: fallback de proveedores
- Multi-currency: soporte de monedas principales + exóticas
- Trazabilidad: cada conversión tiene referencia a evento y ledger
- Consistencia: redondeo determinista y control de errores de fuente
- Historico: almacenar rates para reconstrucción contable y reconciliación

---
> Este archivo puede ampliarse con: diagramas de flujo, ejemplo de reconciliación FX, batch FX para liquidaciones multi-moneda y scripts de testing de márgenes.