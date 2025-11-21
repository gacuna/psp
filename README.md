# Payment Service Provider

## 1. Introducción a la Lógica de Negocio
- Propósito de la sección: mostrar cómo funciona la plataforma desde el punto de vista financiero y operativo.
- Conceptos clave: pay-ins, payouts, settlements, refunds, FX, fees, impuestos.
- Alcance: todos los flujos de dinero, actores involucrados y reglas de negocio.

## 2. Actores y Cuentas
- **Clientes / Usuarios finales**: quienes realizan pagos (pay-ins) o reciben reembolsos.
- **Proveedores / Merchants**: reciben pagos y pueden solicitar payouts.
- **Plataforma / Wallet interna**: controla los balances internos, fees y conciliaciones.
- **Bancos / Procesadores de pagos**: facilitan la liquidación de fondos y FX.

**Tipos de cuentas dentro de la plataforma**:
- Wallets de clientes
- Wallets de proveedores
- Wallet de la plataforma (fees y reservas)
- Cuentas externas vinculadas (bancos y procesadores)

## 3. Secciones detalladas
Para explorar cada módulo con máximo detalle, puedes acceder a los archivos correspondientes:

- [01 Introducción](01_introduccion.md)
- [02 Pay-ins](02_payins.md)
- [03 Payouts](03_payouts.md)
- [04 Refunds y Chargebacks](04_refunds_chargebacks.md)
- [05 Settlements y Conciliación](05_settlements.md)
- [06 Gestión de FX y Monedas](06_fx_monedas.md)
- [07 Cálculo de Fees y Comisiones](07_fees_comisiones.md)
- [08 Impuestos y Compliance](08_impuestos_compliance.md)
- [09 Reglas de Negocio y Casos Especiales](09_reglas_negocio.md)

> Nota: Cada archivo contiene ejemplos, flujos de dinero detallados, cálculo de fees, impuestos, FX y casos especiales, sirviendo como base para el desarrollo completo de la plataforma.
