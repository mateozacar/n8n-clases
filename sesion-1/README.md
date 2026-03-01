# Sesión 1 — Asesor Financiero con Lovable + N8N + Supabase

Construiremos una página web de un asesor financiero que:
- Visualiza gastos e ingresos en un dashboard interactivo
- Permite registrar transacciones conectadas a Supabase
- Incluye una **Calculadora de Metas de Ahorro** que usa un agente de IA en N8N

---

## Resultado Final

```
Usuario escribe su meta en Lovable
        ↓
Lovable envía GET al Webhook de N8N
        ↓
N8N activa el Agente IA (LLM + System Prompt)
        ↓
El Agente lee transacciones reales desde Supabase
        ↓
Genera plan de ahorro personalizado
        ↓
N8N responde → Lovable muestra los resultados
```

---

## Recursos

- [Prompt inicial para Lovable](./prompts/01-prompt-dashboard-inicial.md)
- [Prompt para conectar el Webhook](./prompts/02-prompt-conexion-webhook.md)

---

## Paso 1 — Crear cuentas

### 1.1 N8N
Crea tu cuenta en [n8n.io](https://n8n.io) (tienen plan gratuito en la nube).

### 1.2 Lovable
Crea tu cuenta en [lovable.dev](https://lovable.dev).

### 1.3 Supabase
Crea tu cuenta en [supabase.com](https://supabase.com).

---

## Paso 2 — Crear la página en Lovable

1. Abre [lovable.dev](https://lovable.dev) y crea un nuevo proyecto.
2. Copia el contenido de [`prompts/01-prompt-dashboard-inicial.md`](./prompts/01-prompt-dashboard-inicial.md) y pégalo en el chat de Lovable.
3. Espera a que Lovable genere la aplicación completa.

El resultado debe tener tres pestañas:
- **Dashboard** — gráficos y resumen financiero
- **Actividad** — formulario para registrar transacciones
- **Calculadora de Ahorro** — campo de texto para consultas al agente de IA

---

## Paso 3 — Conectar Supabase a Lovable

### 3.1 Autorizar Lovable en Supabase

1. En Lovable, haz clic en el botón de **Supabase** (arriba a la derecha).
2. Elige la opción **"Add another organization"**.
3. Se abrirá un pop-up — autoriza a Lovable para conectarse a tu cuenta de Supabase.

### 3.2 Crear el proyecto en Supabase

1. Haz clic nuevamente en el botón de **Supabase** en Lovable.
2. Pon el mouse sobre tu organización y da clic en **"Create new project"**.
3. Escribe un nombre y una contraseña para el proyecto. Deja el resto de la configuración sin cambios.
4. Vuelve a Lovable, da clic en el botón de Supabase, elige tu organización y selecciona el proyecto recién creado.
   > A veces el proyecto tarda unos minutos en aparecer — espera y recarga si no lo ves.
5. En el chat de Lovable debería aparecer la confirmación de conexión.

---

## Paso 4 — Crear la tabla en Supabase

Escribe el siguiente prompt en el chat de Lovable:

```
Ya conectamos el proyecto de clases de supabase.

Ahora necesito crear una tabla de la cual saldrá toda la información de la página.
Crea una tabla con estos campos:

* Tipo de transacción (gasto o ingreso)
* Descripción de la transacción (texto)
* Categoría (menú desplegable con opciones comunes + capacidad de crear personalizadas)
* Cuenta (menú desplegable de las cuentas del usuario)
* Fecha (selector de fecha con valor predeterminado a la fecha actual)
* Monto (campo numérico con símbolo de peso colombiano $)
* Método de pago (efectivo, crédito, débito, etc.)
* Notas (opcional - área de texto)
* Carga de recibo (opcional - carga de imagen)

y conecta la información de esa tabla a todas las visualizaciones del dashboard y a la opción de crear una transacción. Cuando se crea una transacción se debe adicionar una fila a la tabla.

No crees ningún tipo de autenticación.
```

> **Verificación:** Lovable te pedirá permiso para modificar la base de datos — acéptalo. Después, intenta crear un gasto desde la pestaña de Actividad y revisa que aparezca en el Dashboard.

---

## Paso 5 — Crear el flujo en N8N

### 5.1 Estructura del workflow

1. Abre tu cuenta de N8N y crea un nuevo workflow.
2. Agrega los siguientes nodos en este orden:

```
[Webhook] → [AI Agent] → [Respond to Webhook]
```

| Nodo | Tipo en N8N |
|------|-------------|
| Entrada | Webhook |
| Agente IA | AI Agent |
| Salida | Respond to Webhook |

### 5.2 Configurar el nodo Webhook

1. Agrega un nodo de tipo **Webhook**.
2. En el campo **"Respond"**, selecciona **"Using 'Respond to Webhook' Node"**.
3. Da clic en **"Listen for test event"** — esto activa el webhook temporalmente para pruebas.
4. **Copia la URL de prueba** que aparece (la necesitarás en el Paso 7).

---

## Paso 6 — Configurar el Agente de IA

### 6.1 Modelo de lenguaje

1. En el nodo **AI Agent**, conecta el modelo de tu preferencia (OpenAI GPT, Anthropic Claude, etc.).
2. Crea las credenciales del modelo cuando N8N te lo solicite (necesitarás una API Key del proveedor).

### 6.2 Memoria

1. En el panel del agente, busca la opción de memoria.
2. Selecciona **Window Buffer Memory**.
3. Deja la configuración predeterminada.

### 6.3 System Prompt

1. En el panel del agente, haz clic en **Options** → **System Message**.
2. Ingresa el siguiente system prompt:

```
Eres un asistente financiero encargado de crear un plan de ahorro para el usuario.
El usuario te va a decir cuáles son sus objetivos, tiempos, ingresos y otra información relevante referente a una meta de ahorro.

Tu debes usar la información que te entregó el usuario para definir un plan de ahorro, definiendo:

- Cantidad mensual de ahorro requerida
- Fecha proyectada de finalización
- Recomendaciones de reducción de gastos

Para la reducción de gastos, debes dirigirte a la herramienta "supabase" y leer la tabla. En esta tabla encontrarás toda la información de los gastos e ingresos del usuario. Tu tarea es entender los gastos del usuario y proponer recortes que podría hacer.

IMPORTANTE:

La fecha de hoy es {{ DateTime.local().toFormat('cccc d LLLL yyyy') }}. Siempre toma esta fecha para el inicio de la meta de ahorro.
```

### 6.4 Configurar la entrada del prompt

1. En el agente, cambia el campo **"Prompt Source (User Message)"** a **"Define Below"**.
2. En el panel izquierdo (INPUT), busca el campo **"prompt"** (que viene del webhook).
3. Arrástralo al campo **"Text"** del agente.

---

## Paso 7 — Conectar la herramienta Supabase al Agente

1. En el nodo AI Agent, agrega una herramienta de tipo **Supabase**.
2. Crea las credenciales de Supabase:

   | Campo en N8N | Dónde encontrarlo en Supabase |
   |---|---|
   | **Host (URL)** | Supabase → Tu proyecto → Settings → API → Project URL |
   | **Service Role Secret** | Supabase → Tu proyecto → Settings → API → `service_role` key (revélala) |

3. En la herramienta Supabase, configura la operación como **"Get Many"**.
4. Selecciona la tabla **`transactions`**.
5. Deja el resto de campos con sus valores predeterminados.

---

## Paso 8 — Conectar el Webhook con Lovable

1. Asegúrate de tener la **URL del webhook** de prueba (copiada en el Paso 5.2).
2. Copia el contenido de [`prompts/02-prompt-conexion-webhook.md`](./prompts/02-prompt-conexion-webhook.md).
3. **Reemplaza `{URL_DEL_WEBHOOK}` con la URL real** que copiaste de N8N.
4. Pega el prompt en el chat de Lovable y espera que haga los cambios.

---

## Paso 9 — Prueba final

1. En Lovable, ve a la pestaña **"Calculadora de Ahorro"**.
2. Escribe una consulta como:

   > *"Quiero ahorrar 5 millones de pesos en 6 meses para un viaje. Mi ingreso mensual es de 3 millones."*

3. Da clic en **"Calcular"**.
4. En N8N debería activarse el flujo — revisa que el agente consulte Supabase y genere la respuesta.
5. En Lovable deberían aparecer las tarjetas con:
   - Ahorro mensual requerido
   - Fecha proyectada de finalización
   - Recomendaciones de reducción de gastos

---

## Solución de Problemas Comunes

### El webhook no recibe datos
- Verifica que en el nodo Webhook, el campo **"Respond"** esté en **"Using 'Respond to Webhook' Node"**.
- Asegúrate de haber dado clic en **"Listen for test event"** antes de hacer la prueba.

### La calculadora no muestra resultados
- Abre la consola del navegador (F12 → Console) y verifica si hay errores de red.
- Revisa que la URL del webhook en el código de Lovable coincida exactamente con la que copiaste de N8N.
- El webhook devuelve `{ "output": "texto markdown" }` — el prompt del Paso 8 ya contempla este formato.

### El agente no lee datos de Supabase
- Verifica que las credenciales de Supabase (URL y Service Role Key) estén correctas.
- Confirma que la tabla se llame `transactions` (en minúsculas).
- Asegúrate de que la operación en la herramienta Supabase sea **"Get Many"**.

### Error de CORS al llamar el webhook
- Esto puede ocurrir si el webhook está en modo **"Test"** y se llama desde un dominio diferente.
- Para producción, activa el workflow en N8N (cambia el Webhook de "Test" a "Production") y actualiza la URL en Lovable.
