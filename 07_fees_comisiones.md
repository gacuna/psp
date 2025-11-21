# Cálculo de Fees y Comisiones

## 1. Definición y propósito
Los **Fees y Comisiones** son cargos aplicados sobre transacciones por la plataforma y los procesadores de pago. Su correcta gestión es crítica para la rentabilidad de la plataforma y la transparencia hacia clientes y proveedores.

**Objetivos:**
- Cobrar por servicios prestados de manera clara.
- Registrar correctamente fees y comisiones para conciliación y reporting.
- Permitir distintas configuraciones según tipo de transacción, país o moneda.

## 2. Tipos de Fees y Comisiones
1. **Comisión de procesador**
   - Porcentaje sobre el monto de la transacción.
   - Fijo por tipo de pago (tarjeta, transferencia, wallet).
2. **Comisión de la plataforma**
   - Fijo o porcentaje.
   - Puede ser por pay-in, payout o ambos.
3. **Fee por FX / Conversión de moneda**
   - Porcentaje aplicado sobre el monto convertido.
   - Se puede combinar con margen de FX.
4. **Impuestos sobre comisiones**
   - IVA u otros impuestos aplicables según legislación.

## 3. Flujo de cálculo paso a paso
1. Determinar monto bruto de la transacción.
2. Aplicar fees del procesador según método de pago.
3. Aplicar comisión de la plataforma según política.
4. Aplicar FX si corresponde.
5. Calcular impuestos sobre fees de la plataforma.
6. Determinar monto neto a acreditar o transferir.

## 4. Ejemplo numérico
- Pago de cliente: **100 USD**
- Fee procesador: 2.5% = 2.50 USD
- Comisión plataforma: 1 USD fijo + IVA 10% = 1 + 0.10 = 1.10 USD
- FX: no aplica (misma moneda)

**Monto neto al proveedor:** 100 − 2.50 − 1.10 = **96.40 USD**

## 5. Casos especiales
1. **Pago parcial:** calcular fees proporcionalmente.
2. **Pago internacional:** aplicar FX y fee FX antes de calcular monto neto.
3. **Transacción revertida o refund:** ajustar fees según política (absorber o devolver parcialmente).

## 6. Registro y reporting
- Cada transacción debe registrar:
  - Monto bruto
  - Fees de procesador
  - Comisión de plataforma
  - Impuestos aplicables
  - FX y margen (si aplica)
  - Monto neto acreditado
  - Estado de transacción

- Usos del registro:
  - Conciliación contable
  - Reporting de ingresos y comisiones
  - Análisis de rentabilidad por tipo de transacción y moneda