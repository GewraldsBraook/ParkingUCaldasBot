# ParkingUCaldasBot

## 🤖 Manual Funcional del Prompt para Chatbot de WhatsApp (n8n)

Este manual describe el propósito, los parámetros, el comportamiento esperado y la integración del **prompt principal** que define la personalidad y las reglas del chatbot.

---

### 1. 🎯 Objetivo del Prompt

El objetivo principal de este prompt es establecer el **rol, tono y las restricciones** del chatbot para asegurar una interacción consistente, útil y dentro de los límites de la integración con n8n y WhatsApp.

### 2. 📝 Estructura del Prompt (Componentes Clave)

El prompt está compuesto por varias secciones que deben ser claras y jerárquicas:

| Sección | Descripción | Contenido Típico |
| :--- | :--- | :--- |
| **A. Identidad y Rol** | Define quién es el chatbot y cuál es su trabajo principal. | "Eres un Asistente Virtual amable y profesional para [Nombre de la Empresa]. Tu tarea es responder preguntas frecuentes sobre [Producto/Servicio] y guiar al usuario a la sección de [Acción Principal, e.g., Compras]." |
| **B. Reglas y Restricciones** | Las directrices más importantes que el modelo **no debe** romper (límites de contexto, longitud, etc.). | *No* inventes información. Si no tienes la respuesta, pide disculpas y sugiere una alternativa. *Nunca* uses emojis que no sean [Lista de emojis permitidos]. Mantén las respuestas por debajo de [Número] palabras. |
| **C. Contexto o Información Fuente** | La base de conocimiento con la que debe trabajar el chatbot (puede ser insertada dinámicamente). | "La empresa ofrece tres planes: Básico (\$10), Premium (\$25), y Empresarial (\$50). El soporte es de Lunes a Viernes." |
| **D. Formato de Salida (Output Format)** | La estructura que el chatbot debe seguir para que n8n pueda procesar la respuesta o para mejorar la legibilidad en WhatsApp. | Responde siempre con un título en **negritas** y luego la respuesta en párrafos cortos. |

### 3. ⚙️ Integración con n8n (Flujo Funcional)

El prompt interactúa con el flujo de n8n en las siguientes fases:

#### 3.1. Nodo de Entrada (Webhook/WhatsApp Trigger)

* **Función:** Recibe el mensaje (`user_message`) del usuario.
* **n8n Acción:** Este mensaje se almacena en una variable, por ejemplo, `${{$json.body.message.text}}`.

#### 3.2. Nodo de Pre-Procesamiento (Code/Function)

* **Función:** Se utiliza para **construir el prompt final** concatenando las secciones estáticas (A, B, D) con el contexto dinámico (C) y el mensaje del usuario.
* **Variable de Salida Clave:** `final_prompt` (Contiene la instrucción completa para el LLM).

$$\text{final\_prompt} = \text{Identidad} + \text{Reglas} + \text{Contexto Dinámico} + \text{"Usuario pregunta: "} + \text{user\_message}$$

#### 3.3. Nodo de LLM (AI/OpenAI/Custom Model)

* **Función:** Envía el `final_prompt` al modelo de lenguaje (LLM).
* **Parámetros Clave:**
    * **Input:** `final_prompt`
    * **Temperatura:** (Suele ser baja, $T \approx 0.2$ para un comportamiento más predecible y fáctico).
    * **Modelo:** (e.g., `gpt-3.5-turbo`, `gpt-4`).

#### 3.4. Nodo de Post-Procesamiento (Code/Function - Opcional)

* **Función:** Validar y limpiar la respuesta del LLM antes de enviarla.
    * *Ejemplo:* Recortar la respuesta si supera un límite de caracteres específico de WhatsApp, o añadir un `[Enviado por bot]` para trazabilidad.

#### 3.5. Nodo de Salida (WhatsApp Send Message)

* **Función:** Envía la respuesta final al usuario.
* **Input:** La respuesta generada por el LLM.

### 4. ⚠️ Casos Límite y Manejo de Errores

Para que el prompt sea robusto, debe anticipar fallas:

| Evento | Instrucción en el Prompt (Sección B) | n8n Manejo de Errores |
| :--- | :--- | :--- |
| **Consulta Irrelevante** | "Si la pregunta no tiene relación con nuestros productos/servicios, responde cortésmente que solo puedes ayudar con temas de [Dominio]." | Uso de un nodo **IF** que detecte palabras clave (`gracias`, `adiós`) para terminar el flujo sin llamar al LLM, ahorrando coste. |
| **Contexto Agotado** | "Si la respuesta requiere información no proporcionada en el contexto, *debes* responder: 'Lo siento, no tengo esa información. ¿Deseas hablar con un agente?'" | El LLM debe generar este texto, que luego puede disparar un nuevo flujo en n8n para notificar a un humano. |
| **Fallo del LLM** | (No aplicable al prompt, es un error técnico) | **Error Handling Branch** en n8n: Si el nodo de LLM falla, el flujo debe saltar a un nodo que envíe un mensaje genérico: "Estamos teniendo problemas técnicos. Por favor, inténtalo más tarde." |

---

### 5. ✅ Recomendaciones de Optimización

* **Prompt Cero (System Role):** Utiliza la función de "System Role" si tu LLM lo permite para las secciones A y B. Esto asegura que el modelo **siempre** mantiene la personalidad.
* **Longitud de Contexto:** Monitorea el uso de tokens. Si el contexto (Sección C) crece mucho, considera usar una base de datos vectorial (como Pinecone o Weaviate) integrada con n8n para hacer una búsqueda semántica *antes* de construir el `final_prompt` (RAG - Retrieval-Augmented Generation).

---

¡Excelente ajuste! Al especificar el modelo (**Gemini 2.5 Flash**), podemos refinar la terminología del manual para que refleje mejor la configuración de Google AI, especialmente en lo que respecta al "System Instruction" y la gestión de la conversación.

Aquí está el manual funcional actualizado en formato Markdown para GitHub, sustituyendo las referencias genéricas de LLM por la terminología de Gemini:

---

## 4. 🚀 Optimización Específica para Gemini

### 4.1. Eficiencia con 2.5 Flash
El modelo `gemini-2.5-flash` está optimizado para velocidad y coste, lo que lo hace ideal para flujos de chat en tiempo real como WhatsApp.

* **Aprovecha la Ventana de Contexto:** 2.5 Flash tiene una ventana de contexto grande. Si el chat es conversacional, puedes inyectar los **últimos 3 pares de mensajes** como historial en el *Content* de la API para mantener la coherencia sin necesidad de una lógica de estado compleja.

### 4.2. Inyección de Contexto (RAG)
Para bases de conocimiento extensas, se sigue recomendando el patrón RAG (Retrieval-Augmented Generation) para mantener el costo y la latencia bajos:

1.  **Búsqueda:** El `user_message` se usa para buscar los fragmentos de contexto más relevantes.
2.  **Inyección:** Estos fragmentos se añaden a la **Sección C** (Contexto) de la llamada, justo antes de la pregunta del usuario.

> **Formato de Contenido Sugerido:**
> `[CONTEXTO RELEVANTE]: [Fragmento 1]; [Fragmento 2].`
> `USUARIO PREGUNTA: [user_message]`

---
