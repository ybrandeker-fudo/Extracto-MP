# Procesador de Extracto · Mercado Pago Argentina

Herramienta de una sola página para armar el extracto de Mercado Pago listo para conciliar y facturar.
Cruza el reporte de **movimientos (balance)** con el de **todas las transacciones**, identifica cada
movimiento y reparte el resultado en las hojas que usa el circuito.

**Abrir la herramienta:** https://USUARIO.github.io/REPO/

Todo corre en el navegador. Los archivos no se suben a ningún servidor.

---

## Qué necesita

| Archivo | De dónde sale |
|---|---|
| Movimientos (balance) | Mercado Pago → Balance → Liberado |
| Todas las transacciones | Mercado Pago → Informes → Transacciones |
| Cuentas de Dash | Reporte *accounts* (acepta el .zip tal cual se baja) |
| Contactos de Odoo | Export de `res.partner` con Referencia, ID, NIF y responsabilidad AFIP |

## Qué devuelve

Un Excel con cinco hojas:

- **Extracto MP** — lo que se sube a Odoo
- **Saas** — los cobros identificados que no son venta SmartPos ni de hardware
- **Terminales** — las ventas SmartPos con el detalle de líneas, descuentos y cantidad
- **Hardware** — las ventas de hardware, que se facturan aparte
- **Control** — el cuadre del período y los avisos

---

## Las reglas

### Qué entra y qué va a otro diario

Tu delivery, deuda por comisiones y comisión de terceros salen del extracto: se facturan por el
circuito de tienda online. Se van con todo lo suyo, incluidos sus contracargos y anulaciones.

Los impuestos, retenciones y el costo de Mercado Pago quedan **siempre** en este extracto, también los
que corresponden a esos cobros. A tienda online se manda únicamente el total de lo que pagó el cliente.

### Un ID de pago, una fila

Si un cobro tuvo devolución parcial o anulación, va el neto en una sola fila. Los que netean cero
contra su devolución se eliminan. No puede quedar ninguna operación repetida.

### Las cinco líneas del pie

Se suman por concepto y quedan al final del extracto, con la fecha del último día del período:

- Ret. SIRTAC
- Imp. Ley Créditos y Débitos
- Ret. IIBB Otras Jurisdicciones
- Ret. IIBB CABA
- Costo de Mercado Pago (la etiqueta del mes se edita en la pantalla)

### Descripciones

Sale el nombre de la cuenta, que viene del reporte *accounts* por ID de Dash. Si la cuenta no está
ahí, cae a la razón social de Odoo.

- **Venta SmartPos** — `Venta SmartPos - ` y el nombre de la cuenta
- **Venta de hardware** — sólo `Venta de Hardware`, sin cuenta ni contacto: se factura aparte
- **Devoluciones y contracargos** — van sin tipo de operación

### El cuadro de terminales

El identificador `TFP:` trae pares de `cantidad;descuento` contra el precio de lista de la terminal.
De ahí salen las columnas LINEA, DESCUENTO, Q TERMINALES y CONTROL, que tiene que dar cero.

### Los tres controles

1. **El cuadre** — extracto + lo que va a otro diario + comisión de terceros = total de movimientos
2. **IDs de pago repetidos** — no puede quedar ninguno duplicado
3. **Pendientes de identificar** — las cuentas que no se resuelven con los archivos

---

## Lo que no se resuelve solo

Hay casos donde los archivos no alcanzan y hace falta cargar el dato a mano:

- **Cuentas renombradas en Dash** — el identificador trae el usuario viejo
- **Terminales con identificador raro** — el segundo tramo a veces no es el ID de Dash
- **Movimientos sin descripción** — compras, retiros y ajustes que Mercado Pago manda vacíos

Se cargan en los paneles de la propia herramienta y quedan recordados para las próximas corridas.
Los pagos son la excepción: cambian en cada movimiento, así que se usan en esa corrida y no se guardan.

> El diccionario se guarda solo cuando la herramienta se abre desde una URL. Si se abre haciendo doble
> clic sobre el archivo, el navegador bloquea el guardado: ahí hay que usar **Exportar diccionario** e
> **Importar diccionario**.
