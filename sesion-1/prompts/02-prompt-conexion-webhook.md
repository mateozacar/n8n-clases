# Prompt 2 — Conexión del Webhook de N8N

> **Antes de usar este prompt:**
> 1. Asegúrate de haber copiado la URL del webhook desde N8N (Paso 5.2 del README).
> 2. Reemplaza `{URL_DEL_WEBHOOK}` con tu URL real.
> 3. Pega el prompt completo en el chat de Lovable.

---

```
En la Calculadora de Metas de Ahorro, al momento de dar clic en "Calcular", debes enviar el mensaje al siguiente webhook:

{URL_DEL_WEBHOOK}

Método: GET
Parámetro: "prompt" (conteniendo el texto exacto ingresado por el usuario)

---

IMPORTANTE — Formato real de respuesta del webhook:

El webhook NO devuelve un JSON estructurado directamente. Devuelve el siguiente objeto:

{
  "output": "texto en formato markdown con el plan de ahorro completo"
}

El campo "output" es un string de texto en markdown que contiene toda la información mezclada: el monto mensual de ahorro, la fecha proyectada y las recomendaciones de reducción de gastos.

---

Lógica de parseo que debes implementar:

1. Recibe la respuesta del webhook y lee el campo "output" (string de texto).
2. Del texto de "output", extrae usando expresiones regulares o búsqueda de patrones:
   - **Monto mensual:** busca un número precedido de "$" o seguido de palabras como "mensual", "mes", "ahorro mensual". Puede estar en formato $1.500.000 o 1500000 o 1,500,000.
   - **Fecha proyectada:** busca patrones de fecha como "dd/mm/yyyy", "enero 2026", "marzo de 2026", o cualquier formato de fecha en español.
   - **Recomendaciones:** extrae líneas que empiecen con "- ", "* ", números seguidos de punto (1. 2. 3.) o cualquier estructura de lista del texto.
3. Si no logras extraer algún campo con el parseo, muestra el texto completo de "output" como fallback en lugar de mostrar vacío o un error.

---

Visualización de resultados:

Muestra los resultados en 3 secciones coherentes con el diseño del dashboard existente (mismos colores, tipografía y estilo de cards):

**Tarjeta 1 — Ahorro mensual requerido**
- Ícono de dinero o piggy bank
- Monto en COP con formato $X.XXX.XXX
- Color de acento verde (#2e7d32)

**Tarjeta 2 — Fecha proyectada de finalización**
- Ícono de calendario
- Fecha en español
- Color de acento azul (#283593)

**Tarjeta 3 — Recomendaciones de reducción de gastos**
- Ícono de lista o lightbulb
- Lista vertical de recomendaciones, cada una en su propia línea
- Si hay texto que no encaja en las tarjetas anteriores, muéstralo aquí también

**Estado de carga:**
- Mientras espera la respuesta, muestra un spinner con el texto "Analizando tu meta de ahorro..."

**Estado de error:**
- Si el webhook falla (timeout, error de red, etc.), muestra un mensaje amigable: "No pudimos conectar con el asesor. Intenta de nuevo en un momento."
- Incluye un botón para reintentar

**Estado vacío:**
- Antes de hacer cualquier consulta, muestra un mensaje motivador invitando al usuario a escribir su meta.

---

No cambies nada del diseño de las otras pestañas (Dashboard y Actividad). Solo modifica la pestaña Calculadora de Ahorro.
```

---

## Notas sobre el formato de respuesta

Durante el desarrollo de esta clase, encontramos que N8N devuelve la respuesta del agente IA en el campo `output` como texto plano/markdown, **no** como un JSON estructurado. Por eso el prompt instruye a Lovable a parsear el texto libre.

Ejemplo de respuesta real del webhook:
```json
{
  "output": "Para alcanzar tu meta de ahorro de $5.000.000 en 6 meses, necesitas ahorrar aproximadamente **$833.333 mensuales**.\n\nLa fecha proyectada de finalización sería el **1 de septiembre de 2026**.\n\nRecomendaciones de reducción de gastos basadas en tus transacciones:\n1. Reducir gastos de entretenimiento ($200.000/mes)\n2. Optimizar gastos de alimentación ($150.000/mes)\n3. Revisar suscripciones mensuales ($80.000/mes)"
}
```

Si en el futuro quieres que N8N devuelva JSON estructurado, agrega un nodo **"Code"** o **"Set"** antes del "Respond to Webhook" que transforme la salida del agente al formato:

```json
{
  "Cantidad mensual de ahorro requerida": 833333,
  "Fecha proyectada de finalización": "2026-09-01",
  "Recomendaciones de reducción de gastos": {
    "Entretenimiento": "Reducir $200.000 mensuales",
    "Alimentación": "Optimizar $150.000 mensuales",
    "Suscripciones": "Revisar $80.000 mensuales"
  }
}
```
