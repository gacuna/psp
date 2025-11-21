# Payouts (Envío de Pagos a Proveedores / Usuarios)

## 1. Definición y propósito
Los **Payouts** son los pagos que la plataforma realiza hacia proveedores, merchants o usuarios finales. Su objetivo es distribuir los fondos netos después de deducir fees, impuestos y FX si corresponde.

**Objetivos:**
- Transferir fondos de manera segura y eficiente.
- Aplicar fees e impuestos correctamente.
- Registrar transacciones para conciliación y reporting.
- Gestionar flujos internacionales con FX cuando sea necesario.

## 2. Actores involucrados
- **Proveedor / Merchant / Usuario final:** recibe el pago.
- **Plataforma:** valida la transacción, calcula fees y genera la transferencia.
- **Procesador de pagos / Banco:** ejecuta la transferencia hacia la cuenta externa.

## 3. Flujos de dinero paso a paso

### 3.1 Flujo estándar
1. Plataforma recibe solicitud de payout.
2. Se verifica saldo disponible y cumplimiento KYC/AML del beneficiario.
3. Se calcula monto neto considerando fees e impuestos.
4. Transferencia de fondos a la cuenta bancaria o wallet externa del beneficiario.
5. Confirmación de la transacción y registro en el sistema.
6. Notificación al proveedor/usuario sobre el estado del payout.

## 4. Ejemplo numérico
Supongamos:
- Fondos a pagar: **200 USD**
- Fee del procesador: **1.5%**
- Comisión de la plataforma: **2 USD fijo**
- Impuesto aplicable: **IVA 10% sobre comisión de la plataforma**

### 4.1 Cálculo paso a paso
1. Fee procesador: 200 × 1.5% = **3 USD**
2. Comisión plataforma: 2 USD + IVA 10% = 2 + 0.20 = **2.20 USD**
3. Fondos netos al proveedor: 200 − 3 − 2.20 = **194.80 USD**

### 4.2 Flujo con FX
- Fondos en USD, proveedor recibe en EUR
- Tipo de cambio: 1 USD = 0.9 EUR
- Monto convertido: 194.80 × 0.9 = **175.32 EUR**
- El registro de fees y FX se mantiene para conciliación y reporting.

## 5. Casos especiales

1. **Payout fallido:**
   - Fondos no se transfieren debido a error bancario.
   - Se reintenta automáticamente según política de la plataforma.
   - Se notifica al proveedor y se registra el error.

2. **Payout parcial:**
   - Solo se puede transferir una parte del monto disponible.
   - Se calcula proporcionalmente fees e impuestos.

3. **Límites de payout:**
   - Restricciones diarias, semanales o mensuales por usuario o cuenta.
   - Payouts que excedan el límite se dividen en múltiples transacciones o se retienen hasta el siguiente periodo.

## 6. Registro y reporting
- Cada payout genera un **registro completo** con:
  - Monto bruto a transferir
  - Fees del procesador
  - Comisión de la plataforma
  - Impuestos aplicables
  - FX aplicado (si corresponde)
  - Estado de la transacción (exitoso, fallido, pendiente)

- Usos del registro:
  - Conciliación contable diaria
  - Dashboard de pagos para proveedores
  - Reporting fiscal (IVA, retenciones, etc.)
