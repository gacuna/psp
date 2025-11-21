# Refunds y Chargebacks

## 1. Definición y propósito
Los **Refunds** son devoluciones de fondos al cliente tras una solicitud válida o acuerdo con el proveedor. Los **Chargebacks** son disputas iniciadas por el cliente ante el emisor de su tarjeta o banco, generalmente por fraude, pago no reconocido o error en la transacción.

**Objetivos:**
- Gestionar devoluciones de manera eficiente.
- Minimizar riesgos y costos para la plataforma.
- Mantener registros completos para conciliación y reporting.
- Cumplir regulaciones de pago y políticas de tarjeta.

## 2. Actores involucrados
- **Cliente / Usuario final:** solicita el refund o inicia el chargeback.
- **Proveedor / Merchant:** recibe notificación y valida la solicitud.
- **Plataforma:** valida elegibilidad, procesa fondos y registra transacción.
- **Procesador de pagos / Banco:** ejecuta la reversión de fondos o gestiona la disputa.

## 3. Flujos de dinero paso a paso

### 3.1 Refund estándar
1. Cliente solicita refund.
2. Plataforma verifica:
   - Pago original
   - Elegibilidad según política
   - Saldo disponible del proveedor
3. Fondos se devuelven al cliente desde la wallet del proveedor o de la plataforma.
4. Se calculan ajustes de fees e impuestos según política.
5. Se registra la transacción y se notifica al cliente y proveedor.

### 3.2 Chargeback
1. Cliente inicia disputa ante su banco.
2. Procesador de pagos notifica a la plataforma.
3. Plataforma analiza evidencia (comprobante de pago, entrega, comunicación con cliente).
4. Plataforma responde a la disputa:
   - Acepta el chargeback y devuelve fondos.
   - Rechaza el chargeback, proporcionando evidencia al procesador/banco.
5. Registro completo para conciliación y reporting.

## 4. Ejemplo numérico: Refund
- Pago original: **150 USD**
- Comisión procesador: **3 USD**
- Comisión plataforma: **2 USD fijo + 10% IVA = 2.20 USD**

### 4.1 Refund completo
- Fondos devueltos al cliente: 150 USD (sin incluir fees ya cobradas) 
- Ajuste interno: la plataforma puede decidir absorber o trasladar fees según política.

### 4.2 Refund parcial
- Cliente solicita **50 USD** de refund
- Proporcional cálculo de fees e impuestos sobre monto refund:
  - Fee procesador: 50 × 2% = 1 USD
  - Comisión plataforma: 2 USD × (50/150) = 0.67 USD + 10% IVA = 0.74 USD
- Fondos netos devueltos: 50 USD - ajustes según política = 49.26 USD (si se aplica fee compartido)

## 5. Casos especiales
1. **Refund fallido:**
   - Saldo insuficiente en la wallet del proveedor.
   - Se notifica y se programa reintento según política.

2. **Chargeback parcial o disputa compleja:**
   - Transacción parcialmente válida.
   - Se calcula monto proporcional a devolver y fees a ajustar.

3. **Fraude detectado:**
   - Refund o chargeback puede ser bloqueado si existe evidencia de fraude.
   - Registro de alerta y reporte interno para prevención.

## 6. Registro y reporting
- Cada refund o chargeback genera un registro completo:
  - Monto original y devuelto
  - Fees aplicadas y ajustes
  - Impuestos devueltos o retenidos
  - FX aplicado (si corresponde)
  - Estado de la transacción (exitoso, fallido, pendiente, en disputa)

- Usos del registro:
  - Conciliación contable diaria
  - Dashboard de clientes y proveedores
  - Reporting fiscal y regulatorio
