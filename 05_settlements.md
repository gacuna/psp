# Settlements y Conciliación

## 1. Definición y propósito
Los **Settlements** son procesos mediante los cuales la plataforma consolida, verifica y liquida los fondos entre cuentas internas y externas, asegurando que todos los registros coincidan con las transacciones realizadas.

**Objetivos:**
- Garantizar que los balances internos y externos coincidan.
- Ajustar diferencias por fees, FX e impuestos.
- Generar reportes contables y fiscales precisos.
- Minimizar errores y riesgos de conciliación.

## 2. Actores involucrados
- **Plataforma:** realiza la conciliación y ajustes internos.
- **Bancos / Procesadores de pagos:** proporcionan reportes de transacciones y balances.
- **Clientes / Proveedores:** indirectamente afectados por discrepancias o ajustes.

## 3. Flujos de operación paso a paso
1. Recolección de todas las transacciones del día (pay-ins, payouts, refunds, chargebacks).
2. Comparación de transacciones internas con reportes de procesadores y bancos.
3. Identificación de diferencias:
   - Pagos pendientes
   - Pagos duplicados
   - Pagos fallidos
4. Aplicación de ajustes:
   - Reconciliación de fees e impuestos
   - Conversión FX si corresponde
   - Corrección de balances internos
5. Generación de reporte contable y fiscal.
6. Notificación a áreas relevantes (finance, soporte, proveedores) en caso de discrepancias críticas.

## 4. Ejemplo numérico
Supongamos que la plataforma recibe el siguiente resumen diario:
- Pay-ins totales: 5000 USD
- Payouts totales: 3000 USD
- Refunds: 200 USD
- Fees internos cobrados: 100 USD
- FX aplicado: 50 USD

### 4.1 Cálculo de balances internos
1. Fondos netos en wallet interna: 5000 - 3000 - 200 + 100 (fees cobradas) ± 50 FX = **1950 USD**
2. Comparación con reporte del procesador/banco: 1950 USD → conciliación exitosa.

### 4.2 Caso de diferencia
- Si el banco reporta 1920 USD, entonces diferencia de **30 USD**.
- Posibles causas: retrasos en pagos, ajustes de FX, fees no reportadas.
- Plataforma identifica origen y realiza corrección o seguimiento.

## 5. Casos especiales
1. **Transacciones pendientes:**
   - Pagos autorizados pero no cobrados.
   - Se mantienen pendientes hasta confirmación.

2. **Transacciones duplicadas:**
   - Detectadas durante la conciliación.
   - Se ajusta el balance interno y se notifica a soporte.

3. **Errores de FX o fees:**
   - Ajuste manual o automático según política de la plataforma.

## 6. Registro y reporting
- Cada settlement genera un **registro completo**:
  - Total pay-ins, payouts, refunds
  - Fees e impuestos aplicados
  - FX aplicado
  - Diferencias identificadas y ajustes realizados
  - Estado de conciliación (completo, pendiente, error)

- Usos del registro:
  - Conciliación diaria, semanal y mensual
  - Auditorías internas y externas
  - Reporting contable y fiscal
