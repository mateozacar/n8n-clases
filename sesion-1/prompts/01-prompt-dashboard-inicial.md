# Prompt 1 — Dashboard Financiero Inicial

> Copia y pega este prompt completo en el chat de Lovable al crear un nuevo proyecto.

---

```
Crea una aplicación web de panel financiero personal en español con pesos colombianos (COP).

La app debe tener exactamente 3 pestañas:
1. Dashboard
2. Actividad
3. Calculadora de Ahorro

---

## PESTAÑA 1: DASHBOARD

Crea exactamente estas visualizaciones (no añadir ni quitar elementos):

### Gráficos
- **Gráfico circular (pie chart):** Gastos por categoría. Muestra el porcentaje y el monto en COP de cada categoría.
- **Gráfico de barras:** Ingresos vs Gastos. Comparación mensual de los últimos 6 meses.
- **Gráfico de líneas:** Saldo acumulado en el tiempo. Un punto por mes.

### Tarjetas de resumen (arriba del dashboard)
- Total de ingresos del mes actual
- Total de gastos del mes actual
- Saldo neto del mes actual
- Número de transacciones del mes actual

### Tabla de desglose
- Tabla con columnas: Fecha, Descripción, Categoría, Cuenta, Monto, Tipo (ingreso/gasto)
- Muestra las últimas 10 transacciones
- Los gastos en rojo, los ingresos en verde

---

## PESTAÑA 2: ACTIVIDAD

Formulario para registrar una nueva transacción con estos campos exactos:

- **Tipo de transacción:** Radio buttons o select — opciones: "Gasto" / "Ingreso"
- **Descripción:** Campo de texto libre
- **Categoría:** Menú desplegable con opciones comunes (Alimentación, Transporte, Vivienda, Entretenimiento, Salud, Educación, Ropa, Servicios, Otros) + opción para crear una categoría personalizada escribiéndola
- **Cuenta:** Menú desplegable (Efectivo, Bancolombia, Nequi, Daviplata, Davivienda, Otro)
- **Fecha:** Selector de fecha — valor predeterminado: fecha actual
- **Monto:** Campo numérico con símbolo $ (COP). Sin decimales.
- **Método de pago:** Select — opciones: Efectivo, Tarjeta crédito, Tarjeta débito, Transferencia, PSE
- **Notas:** Área de texto opcional
- **Comprobante:** Carga de imagen opcional

Botón: "Registrar transacción"

Al dar clic en el botón, la transacción debe guardarse y reflejarse de inmediato en el Dashboard.

---

## PESTAÑA 3: CALCULADORA DE AHORRO

- Un campo de texto grande donde el usuario escribe libremente su meta de ahorro (ejemplo: "Quiero ahorrar 5 millones en 6 meses para un viaje")
- Un botón "Calcular"
- Al dar clic, la app hace una llamada GET al webhook de n8n (la URL se configurará después)
- Mientras espera la respuesta, muestra un spinner o mensaje de carga
- Al recibir la respuesta, muestra los resultados en tarjetas:
  - **Ahorro mensual requerido** (monto en COP)
  - **Fecha proyectada de finalización** (fecha)
  - **Recomendaciones de reducción de gastos** (lista de sugerencias)

Por ahora deja el campo del webhook vacío o con un placeholder. Se configurará en un paso posterior.

---

## DISEÑO Y ESTILOS

- Tipografía: Montserrat para títulos, Roboto para texto
- Colores principales: Azul oscuro (#1a237e), Azul medio (#283593), Verde (#2e7d32), Rojo (#c62828)
- Fondo: Gris muy claro (#f5f5f5)
- Cards con bordes redondeados y sombra suave
- Completamente responsivo

---

## RESTRICCIONES

- Todo el contenido en español
- Todos los montos en pesos colombianos con formato: $1.500.000
- No crear ningún tipo de autenticación ni login
- No usar datos mock en los gráficos — si no hay datos, mostrar estado vacío con mensaje amigable
- No añadir funcionalidades extra más allá de las especificadas
```
