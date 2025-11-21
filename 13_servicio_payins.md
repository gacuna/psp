# Servicio de Pay-ins

## 1. Objetivo del Servicio
El Servicio de **Pay-ins** es el módulo encargado de gestionar la entrada de dinero desde múltiples métodos de pago (tarjetas, bancos, e-wallets, cash-in, etc.) hacia la plataforma. Es el punto inicial de cualquier flujo financiero y fuente de múltiples eventos del ecosistema.

Su función principal es **recibir, validar, procesar y notificar** pagos iniciados por merchants o plataformas clientes.

---

# 2. Alcance del Servicio
El Servicio de Pay-ins cubre:
- Inicialización de pagos.
- Validación de merchants, montos, monedas y permisos.
- Comunicación con procesadores externos.
- Recepción y validación de webhooks.
- Publicación de eventos críticos al Event Bus.
- Creación y actualización del estado del pay-in.
- Manejo de errores, timeouts, duplicados e idempotencia.
- Registro contable en el ledger (indirecto vía eventos).

---

# 3. Arquitectura Interna del Servicio

## 3.1 Componentes principales
- **Controller / API Layer**: recibe requests del merchant.
- **Payment Orchestrator**: ejecuta el flujo completo del pago.
- **Processor Connector**: módulo responsable de integrarse con cada proveedor externo.
- **Webhook Handler**: recepción de notificaciones.
- **Event Publisher**: envía eventos al bus Kafka/RabbitMQ.
- **State Machine**: manejo de estados del pago.
- **DB Transaccional**: registros principales.

## 3.2 Estados típicos de un Pay-in
```
CREATED → PENDING_PROCESSOR → AUTHORIZED → CAPTURED → COMPLETED
                                    ⤷ FAILED
                                    ⤷ CANCELLED
```
Algunas integraciones requieren autorización + captura. Otras confirman en un solo paso.

---

# 4. Flujo completo de Pay-in (detalle técnico)

## 4.1 Paso a paso
1. Merchant envía `POST /payins` con `amount`, `currency`, `payment_method`, etc.
2. API Gateway valida token JWT → reenvía al servicio de pay-ins.
3. Servicio valida:
   - Merchant habilitado.
   - Límites de monto.
   - Moneda permitida.
   - Reglas antifraude.
4. Se genera un `payment_id` idempotente.
5. Se almacena el estado en **CREATED**.
6. El Orchestrator invoca al Processor Connector.
7. Conector envía datos al procesador externo.
8. El procesador responde:
   - **sync**: `AUTHORIZED` o `FAILED`.
   - **async**: `PENDING` y luego envía webhook.
9. Al recibir confirmación, se actualiza estado.
10. Se publica evento `PAYIN_COMPLETED`.
11. Ledger registra movimiento.
12. Fees Engine calcula comisiones.
13. FX Engine aplica conversión si corresponde.
14. Wallet del merchant se acredita.

---

# 5. Modelo de Datos Interno

```json
{
  "payment_id": "uuid",
  "merchant_id": "uuid",
  "amount": 100.00,
  "currency": "USD",
  "status": "PENDING_PROCESSOR",
  "payment_method": {
    "type": "card",
    "network": "visa",
    "last4": "4242"
  },
  "processor": "stripe",
  "processor_reference": "ch_19238asd",
  "created_at": "2024-01-01T10:00:00Z",
  "updated_at": "2024-01-01T10:00:05Z"
}
```

---

# 6. Endpoints del Servicio

## 6.1 POST /payins
Crea un pay-in.

### Request
```json
{
  "merchant_id": "m123",
  "amount": 50.0,
  "currency": "USD",
  "payment_method": {
    "type": "card_token",
    "token": "tok_abc123"
  },
  "metadata": {
    "order_id": "A1001"
  }
}
```

### Response
```json
{
  "payment_id": "p_12345",
  "status": "PENDING_PROCESSOR",
  "redirect_url": null
}
```

---

## 6.2 GET /payins/{id}
Devuelve estado actual.

---

## 6.3 Webhook /processors/{processor}/notifications
Recibe notificaciones externas.

### Validaciones
- Firma criptográfica.
- Timestamp.
- Idempotencia.

---

# 7. Idempotencia
El servicio debe garantizar que:
- 2 requests idénticos no generen 2 pagos.
- 2 webhooks idénticos no actualicen dos veces el estado.

Métodos:
- Uso de `idempotency-key` enviado por el merchant.
- Locks optimistas sobre la fila del pago.

---

# 8. Manejo de Errores

| Error | Causa | Acción |
|------|-------|--------|
| `PROCESSOR_TIMEOUT` | El proveedor no responde | Retry + DLQ |
| `INVALID_AMOUNT` | Monto menor a mínimo permitido | 400 |
| `DUPLICATE_PAYMENT` | Conflicto de idempotencia | 409 |
| `WEBHOOK_INVALID_SIGNATURE` | Falsificación | 403 + bloqueo |

---

# 9. Publicación de Eventos
Eventos publicados al completar flujo:
- `PAYIN_CREATED`
- `PAYIN_PENDING`
- `PAYIN_FAILED`
- `PAYIN_COMPLETED`
- `PAYIN_REFUNDED` (si refund)

Todos se envían al Event Bus.

---

# 10. Integración con procesadores externos
El Processor Connector debe implementar:
- Mapeo de requests.
- Firma y validación.
- Adaptadores para cada método de pago.
- Transformación de respuestas.

---

# 11. Ejemplo completo de flujo con Stripe (texto)
```
Merchant → API Gateway → Pay-ins
Pay-ins → Stripe (payment_intent.create)
Stripe → Pay-ins (webhook payment_intent.succeeded)
Pay-ins → Event Bus → Ledger
Ledger → Wallet Service → Merchant balance updated
```

---

# 12. Métricas y Monitoreo
- Latencia por procesador.
- Tasa de fallos.
- Ratio de conversion (init → success).
- Tiempo promedio de webhook.
- Cantidad de reintentos.

---

# 13. Checklist técnico
- [ ] Validación de token JWT
- [ ] Validación antifraude
- [ ] Idempotencia completa
- [ ] Retries con backoff
- [ ] Validación firma de webhooks
- [ ] Publicación de eventos con confirmación
- [ ] Integridad de estados
- [ ] Auditoría completa

---

> Este archivo será expandido si necesitas agregar: diagramas de secuencia, contratos OpenAPI, manejos avanzados de estado, integración con más procesadores o modelos anti-fraude.
