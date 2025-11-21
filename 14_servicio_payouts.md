# Servicio de Payouts (Pagos Salientes)

## 1. Objetivo del Servicio
El Servicio de **Payouts** gestiona la salida de dinero desde la plataforma hacia los merchants, usuarios finales u otros destinos financieros. Es el módulo responsable de ejecutar **retiros, transferencias bancarias, cash-out, envíos a billeteras externas, pagos masivos y liquidaciones programadas**.

Su función principal es **orquestar, validar, programar y ejecutar** desembolsos con trazabilidad, seguridad e integridad contable.

---
# 2. Alcance del Servicio
Incluye:
- Creación de solicitudes de payout.
- Validación de balance disponible.
- Validación de KYC/KYB del merchant.
- Cálculo de comisiones y fees.
- Gestión de métodos de retiro (banco, wallet, tarjeta, etc.).
- Integración con bancos y procesadores.
- Payouts instantáneos, diferidos o programados.
- Control de riesgos (AML, velocity, límites).
- Actualización de ledger.
- Emisión de eventos.
- Manejo de reintentos y estados.

---
# 3. Tipos de Payouts
- **Instantáneo:** se ejecuta en tiempo real (ej: push-to-card, RTP, PIX, SPEI).
- **Batch/Diferido:** se agrupa por ventana horaria y se procesa en lotes.
- **Programado:** el merchant define fecha y hora.
- **Automático:** basado en reglas (ej: liquidación semanal).
- **Masivo:** múltiples destinatarios en una sola orden.

---
# 4. Arquitectura Interna
## Componentes principales
- **API Layer / Controller**
- **Payout Orchestrator**
- **Balance Validator** (consulta Wallet Service)
- **Processor/Banks Connector**
- **Scheduler** (para batch y programados)
- **AML & Risk Validator**
- **Webhook Handler**
- **State Machine**
- **DB Transaccional**

---
# 5. Flujo completo de Payout
```
1. Merchant hace POST /payouts
2. API Gateway valida JWT
3. Payout Service verifica:
      - balance disponible
      - límites diarios
      - KYC/KYB completo
      - riesgo/AML
4. Se crea payout con estado CREATED
5. Orchestrator envía solicitud al proveedor/banco
6. Proveedor responde sync o async
7. Si async → se espera webhook
8. Estado cambia a:
      - PENDING
      - COMPLETED
      - FAILED
9. Se publica evento PAYOUT_COMPLETED
10. Ledger descuenta el balance
11. Merchant recibe notificación
```

---
# 6. Estados del Payout
```
CREATED → VALIDATED → SENT_TO_PROCESSOR → PENDING → COMPLETED
                                         ↘ FAILED
```

---
# 7. Modelo de Datos Interno
```json
{
  "payout_id": "uuid",
  "merchant_id": "uuid",
  "amount": 120.50,
  "currency": "USD",
  "destination": {
    "type": "bank_account",
    "bank": "Chase",
    "account_number": "****1234"
  },
  "status": "PENDING",
  "processor": "bank_ach",
  "reference": "ach_20240211_123",
  "scheduled_at": null,
  "created_at": "2024-02-11T12:00:00Z"
}
```

---
# 8. Validaciones Críticas
- **Balance**: debe ser mayor o igual al monto.
- **Equipo AML**:
  - listas negras
  - países restringidos
  - montos sospechosos
- **Velocity rules**:
  - máximo por hora/día/mes
  - máximo por beneficiario
- **Datos bancarios válidos**
- **Moneda soportada**
- **Límites de retiro del merchant**

---
# 9. Webhooks del proveedor
Validaciones:
- Firma criptográfica
- Timestamp
- Idempotencia
- Verificar que el payout existe y no está cerrado

---
# 10. Endpoints

## 10.1 POST /payouts
### Request
```json
{
  "merchant_id": "m123",
  "amount": 120.50,
  "currency": "USD",
  "destination": {
    "type": "bank_account",
    "account_number": "****1234",
    "routing": "021000021"
  }
}
```

### Response
```json
{
  "payout_id": "po_9991",
  "status": "CREATED"
}
```

---
## 10.2 GET /payouts/{id}
Devuelve estado actual del payout.

---
## 10.3 POST /payouts/batch
Crea un lote de pagos.

---
## 10.4 Webhooks /processors/{processor}/payouts
Notificación de bancos y proveedores.

---
# 11. Reglas de Idempotencia
- Uso obligatorio de `idempotency-key`.
- Un payout con la misma combinación:
  - merchant
  - monto
  - destino
  - idempotency-key
  **no debe ejecutarse dos veces**.

---
# 12. Manejo de Errores
| Error | Causa | Acción |
|-------|-------|--------|
| `INSUFFICIENT_BALANCE` | Wallet insuficiente | 400 |
| `AML_BLOCKED` | Riesgo alto | 403 |
| `BANK_REJECTED` | Destino inválido | 422 |
| `PROCESSOR_TIMEOUT` | Banco no responde | Retry + DLQ |
| `DUPLICATE_PAYOUT` | Idempotencia | 409 |

---
# 13. Publicación de Eventos
- `PAYOUT_CREATED`
- `PAYOUT_VALIDATED`
- `PAYOUT_COMPLETED`
- `PAYOUT_FAILED`
- `PAYOUT_SCHEDULED`

Usados por ledger, notificaciones, liquidador, etc.

---
# 14. Scheduler para Payouts Programados
Características:
- Cron interno
- Jobs configurables por merchant
- Retries
- Auditoría

Ejemplos:
- Liquidación diaria 17:00
- Semanal lunes 09:00
- Mensual

---
# 15. Integración con Bancos/PSPs
Módulo Connector debe incluir:
- Validación de destino
- Mapping de formatos (ACH, SEPA, PIX, SPEI…)
- Reintentos con backoff
- Reconciliación
- Conversión de estados
- Liquidación de fees

---
# 16. Seguridad
- Encriptación de datos bancarios (AES-256 + KMS)
- Tokenización de cuentas
- Accesos mínimos
- Auditoría completa

---
# 17. Métricas a Monitorear
- Tasa de fallas
- Tiempo de procesamiento
- Tiempo de confirmación del banco
- Tasa de reintentos
- Límites por merchant
- Volumen por país y método

---
# 18. Checklist Técnico
- [ ] Validación de balance
- [ ] Validación AML/KYC
- [ ] Programación de payouts
- [ ] Idempotencia estricta
- [ ] Seguridad bancaria
- [ ] Manejo de múltiples procesadores
- [ ] Publicación de eventos
- [ ] Reconcilación bancaria
- [ ] Auditoría obligatoria

---
> Este archivo está listo para ampliarse con: diagramas, contratos OpenAPI, ejemplos por país (ACH, SEPA, PIX, SPEI), manejo de batchs, liquidación automática, o flujos multi-moneda.