# MVP de Plataforma de Pagos – Arquitectura, Roadmap y Módulos Prioritarios

## 1. Objetivo del MVP
El objetivo del MVP es procesar **pay-ins** de forma segura, auditable y consistente. El sistema debe recibir un pago, procesarlo mediante un PSP sandbox o mock, registrar los movimientos financieros en un ledger inmutable, y actualizar el balance del merchant.

El MVP se enfoca en:
- Recibir y procesar **pay-ins** (cobros).
- Registrar movimientos financieros en un **ledger**.
- Exponer el **balance** del merchant (wallet derivado).
- Manejar **idempotencia**, **auditoría** y **seguridad mínima**.
- Integrarse con un PSP sandbox (Stripe o un mock propio).

No incluye: payouts, refunds, settlements, reconciliación avanzada, multi-moneda, múltiples PSPs.

---

## 2. Arquitectura del MVP (mínima)
### Servicios a implementar
1. **API Gateway**
2. **Servicio de Pay-ins** (orquestación)
3. **Ledger Service** (libro mayor)
4. **Wallet Service** (cálculo de balances)

### Infraestructura mínima
- PostgreSQL (transaccional + ledger)
- Redis (idempotencia, cache)
- Kafka o RabbitMQ (opcional pero recomendado para eventos internos)
- Logging centralizado (ELK o similar)
- Contenedores Docker para desarrollo

### Flujo esencial
1. Merchant crea un pay-in.
2. Servicio Pay-ins valida, registra, y envía al PSP.
3. PSP responde (sync o async vía webhook).
4. Evento `PAYIN_COMPLETED` dispara creación de asientos contables en el Ledger.
5. Wallet Service deriva el balance del merchant desde ledger.
6. Merchant puede consultar su balance en cualquier momento.

---

## 3. Módulos a desarrollar primero (orden recomendado)
1. **Idempotency Layer**
2. **Servicio Pay-ins** (con integración con PSP sandbox)
3. **Ledger básico** (journal entries y lines)
4. **Wallet Service** (materialización y consulta de balances)
5. **Event bus interno** (o colas internas)
6. **Logging, trazabilidad y auditoría**
7. **Testing E2E y sandbox environment**

---

## 4. Roadmap por fases

### 🚀 Phase 0 – Infra & Setup (1–2 semanas)
- Configuración de PostgreSQL, Redis y Kafka/RabbitMQ.
- API Gateway básico.
- Docker compose para entorno local.
- Logging + trace-id.

**Entregable:** infraestructura lista + endpoint base

---

### 🚀 Phase 1 – Pay-ins MVP (3–4 semanas)
- `POST /payins`
- `GET /payins/{id}`
- Integración PSP sandbox
- Webhook handler
- Idempotencia completa
- Ledger inicial
- Wallet derivado

**Entregable:** primer pago completo con balance actualizado.

---

### 🚀 Phase 2 – Robustez del core (3–5 semanas)
- Ledger hardening
- Wallet snapshots
- Retry + DLQ
- Primeras reglas de fees (estáticas)
- Monitoreo básico
- Seguridad endurecida

**Entregable:** núcleo financiero estable

---

### 🚀 Phase 3 – Payouts y Refunds (4–6 semanas)
> *Fuera del MVP pero dentro del roadmap inmediato*

- Payouts: reserva de fondos, integración bancaria, reversión en fallos
- Refunds: reversión parcial o total
- Ledger entries

**Entregable:** flujo completo cobro → retiro → devolución

---

### 🚀 Phase 4 – Settlements + Reconciliación (4–8 semanas)
> *Avanzado – no necesario para versión inicial comercial*

- Generación de liquidaciones
- Ingesta de archivos PSP
- Matching automático
- Ajustes contables

**Entregable:** liquidación financiera autónoma

---

## 5. Criterios de aceptación del MVP
### ✔ Pay-in
- Sistema recibe y procesa un pago exitosamente.
- Ledger crea un asiento balanceado (debit == credit).
- Wallet refleja aumento del balance neto.
- Idempotencia evita pagos duplicados.

### ✔ Auditoría y seguridad
- Logs con trace-id.
- Webhooks validados por firma.
- Ledger inmutable.

### ✔ Confiabilidad
- Tolerancia a reintentos.
- Eventos registrados de forma consistente.

---

## 6. Qué NO incluye el MVP
Para mantener foco y reducir scope:
- ❌ Multi-moneda
- ❌ Múltiples PSPs
- ❌ Payouts / refunds
- ❌ Settlement automático
- ❌ Reconciliación avanzada
- ❌ Dashboard gráfico
- ❌ Motor FX
- ❌ Motor de fees dinámico

---

## 7. Siguientes pasos sugeridos
1. Generar **diagrama de arquitectura (Mermaid)** para documentación.
2. Crear **OpenAPI 3.0** con endpoints del MVP.
3. Desglosar el MVP en **tickets de desarrollo**.
4. Modelar el **ledger exacto**: entradas por tipo de pay-in.

