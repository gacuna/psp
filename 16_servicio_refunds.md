# Servicio de Refunds (Devoluciones)

## 1. Objetivo del Servicio
El Servicio de **Refunds** administra todo el ciclo de vida de las devoluciones de dinero asociadas a pay-ins. Permite a un merchant devolver total o parcialmente un pago, garantizando integridad contable, seguridad, trazabilidad y sincronización con procesadores externos.

Su función es **validar, procesar, registrar y notificar** reembolsos, además de integrarse con pay-ins, payouts, ledger, antifraude y conciliación.

---
# 2. Alcance del Servicio
Incluye:
- Creación de solicitudes de refund
- Refunds totales y parciales
- Validación de límites (merchant, método de pago, procesador)
- Validación antifraude / AML (según país)
- Cálculo de fees reembolsables o no reembolsables
- Integración con procesadores externos
- Manejo de estados asincrónicos
- Actualización de ledger
- Publicación de eventos
- Conciliación y auditoría

No incluye:
- Chargebacks (se manejan en otro módulo)

---
# 3. Caso de Uso Principal
```
Cliente compra → Merchant procesa pay-in
↓
Cliente pide devolución → Merchant crea refund
↓
Plataforma valida y ejecuta refund
↓
Procesador confirma
↓
Ledger registra
↓
Merchant recibe actualización
```

---
# 4. Arquitectura Interna del Servicio
Componentes clave:
- **Refund API / Controller**
- **Refund Orchestrator**
- **Pay-in Validator** (consulta servicio de pay-ins)
- **Balance Validator** (consulta Wallet Service)
- **Processor Connector** (Stripe/Adyen/PayPal/etc.)
- **Webhook Handler**
- **Refund State Machine**
- **Refund Ledger Mapper**
- **Refund DB** (transaccional)

---
# 5. Estados de un Refund
```
CREATED → VALIDATED → SENT_TO_PROCESSOR → PENDING → COMPLETED
                                            ↘ FAILED
```

Estados adicionales (si soportados por el procesador):
- `QUEUED` (programado o en batch)
- `CANCELLED` (merchant canceló antes del execution)

---
# 6. Flujo Completo del Refund
```
1. Merchant envía POST /refunds
2. API Gateway valida el JWT
3. Refund Service valida:
      - pay-in existe
      - pay-in está capturado/completado
      - monto no excede lo disponible
      - límites del merchant
      - reglas AML
      - si ya existe un refund duplicado
4. Se crea refund con estado CREATED
5. Orchestrator prepara request al procesador
6. Processor Connector ejecuta refund
7. El procesador responde sync o async (webhook)
8. Se actualiza estado a COMPLETED o FAILED
9. Ledger registra movimiento
10. Event Bus publica REFUND_COMPLETED
11. Merchant recibe notificación
```

---
# 7. Modelo de Datos Interno
```json
{
  "refund_id": "uuid",
  "payment_id": "p_12345",
  "merchant_id": "m123",
  "amount": 20.00,
  "currency": "USD",
  "reason": "customer_requested",
  "status": "PENDING",
  "processor": "stripe",
  "processor_reference": "re_99123",
  "created_at": "2024-02-10T10:00:00Z"
}
```

---
# 8. Validaciones Críticas
- El pay-in debe existir
- El pay-in debe estar en estado CAPTURED/COMPLETED
- Monto del refund ≤ monto restante disponible
- Límites del merchant por período
- Reglas antifraude (según región)
- Refunds múltiples requieren control de concurrencia
- `idempotency-key` obligatorio

---
# 9. Lógica de Refunds Parciales
Si un pay-in de 100 USD ya tiene 2 refunds previos:
```
Refund 1: 30 USD
Refund 2: 20 USD
Disponible: 50 USD
```
Solicitar un refund de 60 USD debe devolver error:
```
AMOUNT_EXCEEDS_AVAILABLE_REFUND
```

---
# 10. Fees en Refunds
Dependiendo del país y proveedor:

| Fee | Reembolsable | Notas |
|-----|--------------|--------|
| Fee plataforma | A veces | Depende contrato merchant |
| Fee procesador | Casi nunca | Stripe no lo devuelve |
| Fee bancario | No | ACH/SEPA cobran costos |

Ejemplo:
```
Pay-in: 100 USD
Fee procesador: 2 USD
Fee plataforma: 1 USD
Refund total: 100 USD
Fees retenidos: 3 USD
```

---
# 11. Integración con Procesadores / PSPs
Cada procesador tiene modelos diferentes:

### Stripe
- Refund instantáneo
- Webhooks: `charge.refunded`

### Adyen
- Refund en batch
- Estado `received → processing → refunded`

### PayPal
- Lógica async compleja

### Bancos (transferencias)
- Archivo de devolución
- Reconciliación manual parcial

El **Processor Connector** abstrae todo esto.

---
# 12. Endpoints del Servicio

## 12.1 POST /refunds
### Request
```json
{
  "payment_id": "p_12345",
  "amount": 10.00,
  "reason": "product_return"
}
```

### Response
```json
{
  "refund_id": "r_789",
  "status": "CREATED"
}
```

---
## 12.2 GET /refunds/{id}
Devuelve estado actual del refund.

---
## 12.3 Webhook /processors/{processor}/refunds
Recibe confirmaciones externas.

---
# 13. Idempotencia
Basada en:
- `idempotency-key`
- combinación (merchant + payment_id + amount)
- locking optimista sobre el refund

Evita duplicar devoluciones.

---
# 14. Manejo de Errores
| Error | Causa | Acción |
|-------|--------|--------|
| `PAYMENT_NOT_FOUND` | Pay-in inexistente | 404 |
| `INVALID_REFUND_AMOUNT` | Monto inválido | 400 |
| `REFUND_LIMIT_EXCEEDED` | Límite diario/semanal excedido | 403 |
| `DUPLICATE_REFUND` | Idempotencia | 409 |
| `PROCESSOR_TIMEOUT` | PSP no responde | Retry + DLQ |
| `WEBHOOK_INVALID_SIGNATURE` | Falsificación | 403 |

---
# 15. Eventos Publicados
- `REFUND_CREATED`
- `REFUND_PENDING`
- `REFUND_COMPLETED`
- `REFUND_FAILED`

Usados por ledger, notificaciones y conciliación.

---
# 16. Conciliación (Reconciliation)
Conciliación diaria:
- Refunds solicitados vs refund confirmados por procesador
- Diferencias → `adjustments`

Ejemplos de desvíos:
- Procesador rechazó refund por límite
- Refund duplicado detectado afuera
- Procesador aplicó FX distinto

---
# 17. Métricas
- % de refund/volumen
- Tiempo de refund
- Tasa de fallas
- Diferencias por conciliación
- Refunds parciales promedio

---
# 18. Checklist Técnico
- [ ] Validación de estado del pay-in
- [ ] Control de refunds parciales
- [ ] Límite máximo por pay-in
- [ ] Límite temporal por merchant
- [ ] Idempotencia estricta
- [ ] Integración con múltiples PSP
- [ ] Manejo de webhooks con firma
- [ ] Ledger actualizado
- [ ] Auditoría completa

---
> Este archivo puede ampliarse con: flujos multi-moneda, refunds complejos por split payments, diagramas de orquestación, ejemplos específicos por región y contratos OpenAPI completos.
