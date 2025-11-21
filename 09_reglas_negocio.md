# Reglas de Negocio y Casos Especiales

## 1. Definición y propósito
Esta sección documenta las **reglas de negocio** que rigen la operativa de la plataforma de pagos, así como los **casos especiales** que requieren atención particular para mantener consistencia, cumplimiento y eficiencia en los flujos de dinero.

**Objetivos:**
- Definir límites, políticas y procedimientos internos.
- Gestionar eventos excepcionales de manera consistente.
- Minimizar riesgos operativos y financieros.
- Garantizar transparencia hacia clientes, proveedores y autoridades.

## 2. Límites y validaciones
1. **Límites de transacciones por usuario:**
   - Máximo diario, semanal o mensual.
   - Aplicable a pay-ins, payouts y refunds.
2. **Límites por método de pago:**
   - Diferentes límites para tarjeta, transferencia bancaria o wallet.
3. **Validaciones de saldo:**
   - Antes de permitir payouts o refunds, verificar fondos disponibles.
4. **Validaciones KYC/AML:**
   - Bloqueo de transacciones hasta completar verificación de identidad.

## 3. Gestión de pagos fallidos y retries
- **Pagos fallidos:**
  - Registro del error y notificación al usuario.
  - Reintento automático según política y método de pago.
- **Retries manuales:**
  - Para casos que requieren intervención de soporte.
- **Cancelaciones:**
  - Transacciones no procesadas o revertidas antes de liquidación.

## 4. Políticas de retención de fondos
- Retención temporal de fondos en casos de:
  - Disputas activas
  - Transacciones sospechosas
  - Payouts programados que exceden límites de liquidez
- Notificación al proveedor y registro contable.

## 5. Prevención de fraude y monitoreo de riesgo
- Monitoreo en tiempo real de patrones sospechosos:
  - Transacciones inusuales en monto, frecuencia o ubicación.
- Alertas automáticas para revisión manual.
- Bloqueo de cuentas o transacciones hasta validación.
- Integración con sistemas antifraude externos si aplica.

## 6. Casos especiales
1. **Transacciones parciales:**
   - Pagos o payouts incompletos.
   - Ajustes proporcionales de fees e impuestos.
2. **Transacciones reversadas:**
   - Fondos devueltos antes de liquidación.
   - Ajustes de fees y reporting.
3. **Errores de FX:**
   - Ajustes de conversión o fees aplicados incorrectamente.
4. **Pagos internacionales complejos:**
   - Combinación de múltiples monedas, fees y retenciones.
   - Registro detallado para conciliación y fiscalidad.

## 7. Registro y reporting
- Cada evento especial debe generar un **registro completo**:
  - Tipo de evento y motivo
  - Monto involucrado y moneda
  - Ajustes aplicados (fees, impuestos, FX)
  - Estado final de la transacción
  - Acciones correctivas realizadas

- Usos del registro:
  - Auditoría interna y externa
  - Conciliación contable
  - Monitoreo de riesgos y prevención de fraude
  - Reporting regulatorio si aplica