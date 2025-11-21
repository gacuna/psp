# Gestión de FX y Monedas

## 1. Definición y propósito
La **Gestión de FX (Foreign Exchange)** permite que la plataforma soporte pagos y payouts en múltiples monedas, asegurando que los proveedores y clientes reciban o envíen fondos en su moneda preferida con conversión transparente y registro contable.

**Objetivos:**
- Facilitar pagos internacionales.
- Aplicar márgenes y fees sobre conversión de divisas.
- Minimizar pérdidas por fluctuaciones de mercado.
- Mantener registro detallado de FX para reporting y conciliación.

## 2. Actores involucrados
- **Cliente / Usuario final:** realiza pagos en su moneda.
- **Proveedor / Merchant:** recibe fondos en su moneda local.
- **Plataforma:** aplica conversión, fees y registra transacción.
- **Procesadores de pagos / Bancos / Servicios FX:** proporcionan tipos de cambio y ejecutan conversiones.

## 3. Flujos de operación paso a paso
1. Cliente realiza un pago en moneda A.
2. Plataforma verifica si el proveedor acepta moneda A o necesita conversión a moneda B.
3. Se determina el tipo de cambio aplicable y se calcula monto convertido.
4. Se calculan fees y márgenes de FX.
5. Fondos netos en moneda del proveedor se acreditan en la wallet interna.
6. Se registra FX aplicado y notificación a las partes.

## 4. Ejemplo numérico
- Cliente paga: **100 EUR**
- Proveedor recibe: **USD**
- Tipo de cambio aplicado: 1 EUR = 1.1 USD
- Monto convertido: 100 × 1.1 = 110 USD

### 4.1 Fees y margen FX
- Fee procesador: 2% sobre monto convertido = 110 × 2% = **2.2 USD**
- Margen FX de la plataforma: 1% = 110 × 1% = **1.10 USD**
- Fondos netos al proveedor: 110 − 2.2 − 1.10 = **106.7 USD**

### 4.2 Registro contable
- Monto bruto: 100 EUR
- Monto convertido: 110 USD
- Fees total: 3.3 USD
- Fondos netos: 106.7 USD
- Tipo de cambio aplicado: 1.1

## 5. Casos especiales
1. **Pago en moneda no soportada por el proveedor:**
   - Plataforma convierte automáticamente a moneda aceptada.

2. **Fluctuaciones significativas de FX entre autorización y liquidación:**
   - Se aplica FX al momento de liquidación.
   - Se puede aplicar política de absorción de pérdidas por parte de la plataforma.

3. **Pagos parciales con FX:**
   - Cada pago parcial se convierte según tipo de cambio vigente.
   - Se calcula fees y margen FX proporcionalmente.

## 6. Registro y reporting
- Cada operación FX genera un registro completo:
  - Monto original y moneda
  - Monto convertido y moneda destino
  - Tipo de cambio aplicado
  - Fees y margen FX
  - Estado de transacción

- Usos del registro:
  - Conciliación diaria y mensual
  - Reporting contable y fiscal
  - Análisis de impacto FX y margen