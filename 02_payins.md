# Pay-ins (Recepción de Pagos)

## 1. Definición y propósito
Los **Pay-ins** son los pagos que los clientes realizan hacia la plataforma o directamente hacia los proveedores/merchants a través de la plataforma. Permiten capturar fondos de distintas fuentes y registrar correctamente los balances internos y externos.

**Objetivos:**
- Recibir pagos de manera segura.
- Aplicar fees e impuestos según normativa y política de la plataforma.
- Registrar transacciones para conciliación y reporting.
- Manejar FX en caso de pagos internacionales.

## 2. Actores involucrados
- **Cliente / Usuario final:** inicia el pago.
- **Plataforma:** recibe, valida y registra la transacción.
- **Proveedor / Merchant:** recibe los fondos netos, descontando fees e impuestos.
- **Procesador de pagos / Banco:** ejecuta la transferencia de fondos y reporta el estado.

## 3. Flujos de dinero paso a paso

### 3.1 Flujo estándar
1. Cliente inicia el pago (tarjeta, transferencia, wallet).
2. La plataforma valida: saldo, límites, KYC/AML.
3. Procesador de pagos autoriza la transacción.
4. Fondos ingresan a la **wallet interna de la plataforma**.
5. Se calcula y descuenta:
   - Fee del método de pago
   - Fee de la plataforma
   - Impuestos aplicables
6. Fondos netos se acreditan en **wallet del proveedor**.
7. Notificación a todas las partes sobre el estado de la transacción.

## 4. Ejemplo numérico
Supongamos:
- Pago de un cliente: **100 USD**
- Comisión del procesador: **2.5%**
- Comisión de la plataforma: **1 USD fijo**
- Impuesto aplicable: **IVA 10% sobre comisión de la plataforma**

### 4.1 Cálculo paso a paso
1. Fee procesador: 100 USD × 2.5% = **2.5 USD**
2. Comisión plataforma: 1 USD + IVA 10% = 1.10 USD
3. Fondos netos al proveedor: 100 USD − 2.5 USD − 1.10 USD = **96.4 USD**

> Este registro se utiliza para conciliación contable y reporting.

### 4.2 Flujo con FX
- Cliente paga en **EUR 100**, proveedor recibe en **USD**
- Tipo de cambio aplicado: 1 EUR = 1.1 USD
- Monto convertido: 100 × 1.1 = 110 USD
- Aplicar fees e impuestos:
  - Fee procesador 2.5%: 110 × 2.5% = 2.75 USD
  - Comisión plataforma 1 USD + IVA 10% = 1.10 USD
  - Fondos netos al proveedor: 110 − 2.75 − 1.10 = **106.15 USD**

## 5. Casos especiales

1. **Pago fallido:**
   - Fondos no ingresan al wallet interno.
   - Se notifica al cliente y se registra el error.
   - Reintentos automáticos según política.

2. **Pago parcial:**
   - Cliente envía menos de lo solicitado (ej. 60 USD de 100 USD).
   - Plataforma registra saldo pendiente.
   - Cálculo proporcional de fees e impuestos.

3. **Reverso / Refund antes de acreditación:**
   - Fondos aún no llegan al proveedor.
   - Transacción cancelada y monto devuelto al cliente sin fees adicionales.

## 6. Registro y reporting
- Cada transacción genera un **registro completo**:
  - Monto bruto
  - Fees del procesador
  - Comisión de plataforma
  - Impuestos aplicables
  - FX aplicado (si corresponde)
  - Estado de la transacción (autorizado, fallido, pendiente)

- Usos del registro:
  - Conciliación contable diaria
  - Dashboard de pagos para proveedores
  - Reporting fiscal (IVA, retenciones, etc.)