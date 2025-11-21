# Servicio de Reporting y Monitoring (Dashboards, Métricas, Logs y Alertas)

## 1. Objetivo
El **Reporting & Monitoring Service** permite a la plataforma de pagos:
- Visualizar el estado de transacciones y balances
- Monitorear métricas operativas y financieras
- Analizar logs y eventos
- Generar alertas ante incidentes o desviaciones

El servicio proporciona **información en tiempo real y reportes históricos** para operaciones, contabilidad y cumplimiento regulatorio.

---

## 2. Alcance
- Dashboards para merchants y operaciones internas
- Métricas de transacciones: pay-ins, payouts, refunds, fees, settlements
- Logs centralizados y correlación de eventos
- Alertas configurables (ej: tiempo de respuesta, errores de procesamiento, inconsistencias de ledger)
- Exportes a sistemas externos o BI tools

---

## 3. Componentes del servicio
- **Data Collector**: recolecta eventos de todos los microservicios (pay-ins, payouts, refunds, FX, fees, settlements)
- **Metrics Engine**: calcula métricas en tiempo real y agregadas
- **Dashboard API**: proporciona endpoints para visualización y filtrado
- **Logs Aggregator**: almacena logs centralizados y relaciona eventos por `trace_id`
- **Alerting Engine**: detecta anomalías y envía notificaciones a los canales configurados
- **Reporting Scheduler**: genera reportes periódicos y exports históricos

---

## 4. Métricas recomendadas
| Categoría | Métrica | Descripción |
|-----------|--------|------------|
| Transacciones | Total pay-ins, payouts, refunds | Cantidad y volumen diario, semanal, mensual |
| Financiera | Revenue neto, fees, FX gains/losses | Totales consolidados por merchant y global |
| Operacional | Latencia transacciones, tasa de fallas | Tiempo promedio de ejecución, errores por tipo |
| Settlement | % de settlements reconciliados automáticamente | Indicador de eficiencia del engine |
| Alerts | Cantidad de alertas críticas | Incidentes detectados por sistema |

---

## 5. Dashboards
- **Overview**: indicadores globales, transacciones, balances, FX, fees
- **Transactions**: desglose por tipo de operación y estado
- **Finance**: ingresos netos, fees, impuestos, FX, refunds
- **Settlements**: pendientes, completados, fallidos, ajustes
- **Alerts & Logs**: últimos eventos, alertas críticas, logs correlacionados por trace_id

---

## 6. Logs y eventos
- Centralización de logs de todos los servicios
- Correlación por `trace_id` y `business_event_id`
- Retención configurable según política de la plataforma
- Soporte para exportar logs a sistemas externos (SIEM, Splunk, ELK)

---

## 7. Alertas
- Configurables por umbral de métricas o patrones de logs
- Canales de notificación: email, Slack, SMS, webhook
- Tipos de alertas:
  - Error en pay-in/payout/refund
  - Desviaciones de FX o fees
  - Fallos en reconciliación
  - Latencia alta en servicios críticos

---

## 8. Modelos de datos
### 8.1 Evento de métricas
```json
{
  "metric_id": "uuid",
  "service": "payin",
  "type": "latency",
  "value": 250,
  "unit": "ms",
  "timestamp": "2024-02-21T12:00:00Z",
  "trace_id": "trace_001"
}
```

### 8.2 Evento de alerta
```json
{
  "alert_id": "uuid",
  "metric_id": "uuid",
  "severity": "CRITICAL",
  "description": "Pay-in failed for 5 transactions",
  "timestamp": "2024-02-21T12:05:00Z",
  "channel": "slack"
}
```

---

## 9. API Endpoints (ejemplo)
### 9.1 GET /metrics
- Devuelve métricas actuales y filtrables por servicio, tipo y periodo

### 9.2 GET /alerts
- Lista de alertas activas y pasadas, con filtro por severidad y servicio

### 9.3 GET /dashboard
- Endpoint resumido para visualización rápida de KPIs

### 9.4 POST /alerts/ack
- Acknowledge de alertas por usuario o sistema

---

## 10. Consideraciones
- Multi-tenancy: dashboards diferenciados por merchant o usuario interno
- Retención histórica configurable para auditoría
- Integración con sistemas de BI externos
- Soporte para métricas en tiempo real y batch reporting
- Escalabilidad horizontal para manejar alto volumen de eventos

---
> Este archivo puede ampliarse con diagramas de flujo de recolección de métricas, ejemplos de dashboards, alerting rules y scripts de simulación de eventos para testing.